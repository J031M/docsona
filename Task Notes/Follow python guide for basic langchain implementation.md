
## Document loader

### Text
langchain.document_loaders.TextLoader is used for text files.

### PDF
langchain.document_loaders.PyPDFLoader is used for pdfs. You need to import pypdf. Advantage is document can be retrived with page numbers. They used OpenAI's embeddings for some reason so API key is needed.

There's Unstructured which showed a way to fetch remote pdfs (like arxiv).

PyMupdf is the fastest option, and it also returns a lot of metadata including one document per page.

### Integrations
Other than popular filetypes there are a shit-ton of integrations to load documents from github, youtube, twitter, obsidian, wikipedia, whatsapp, reddit, jupyter, hackernews, email,  discord, arxiv you get the idea. This is insane!

## Document Transformers

Langchain comes with textsplitters that split the document in a way that keeps the splits semantically similar and preserves context between splits. Here's the code
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(    
# Set a really small chunk size, just to show.
chunk_size = 100,
chunk_overlap  = 20,
length_function = len,
add_start_index = True,
)

texts = text_splitter.create_documents([state_of_the_union])
print(texts[0])
print(texts[1])
```

```
page_content='Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and' metadata={'start_index': 0}
page_content='of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.' metadata={'start_index': 82}
```

There are also specialised splitters for md, html and programming languages.

## Text Embedding Models

Langchain has an embeddings class that serves as a common interface to all the embedding providers. There's a seperate function to embed documents and embed queries coz some have seperate embeders.

## Vector Stores

Does literally just that. Stores embedings of your docs and at query time, embeds the query and retrieves similar vectors. This takes care of storing and doing the vector search. 

FAISS vector database uses Facebook AI similarity search. There's also pinecone, chroma and Redis, you'll need to pick which you want. 

## Retrievers

It returns documents given an unstructured query. Vector stores may be used as their backbone but there are alternatives. The documentation was a tutorial of our project instead.

This part of the docs is kinda hands-on, I'll do my best to distill it

### Context compressors

The context compressor takes input from the retreiver and further distills it to what is relevant to the query. 

**LLMChainExtractor**: Iterates over the results and extracts relevant context.
**LLMChainFilter**: Iterates over the results and filters out irrelevant documents without changing their content
**EmbeddingsFilter**: An LLM call may be too expensive in which case filter through documents which have similar embeddings (which is the whole point of vector search no? Maybe this is for the other retreival methods)

### Self query

These retrievers are capable of extracting filters from the query and apply those filters on the metadata of the document. I don't think it's relevant to our project.

```python
# This example specifies a query and a filter
retriever.get_relevant_documents("Has Greta Gerwig directed any movies about women")
```
```
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig')


[Document(page_content='A bunch of normal-sized women are supremely wholesome and some men pine after them', metadata={'director': 'Greta Gerwig', 'rating': 8.3, 'year': 2019.0})]
```

### Time decay

Reduce relevance of less recently accessed documents. Good if we've got a huge database but for accuracy reasons I think it's best to limit the ingested amount per chat.

---
# Code

I'm going to add some MLOps stuff here which needs to be merged with the main  file. 

## Project structure
I'll be using cookiecutter to set up the structure. Data science is iterative and trial and error based, so its structure is different from typical projects but without structure it would devolve into a mess of notebooks.

I'm using the [cookiecutter datascience template](https://drivendata.github.io/cookiecutter-data-science/) since its minimal and we don't know the entire scope of our project yet. I think this lacks version controlling (as folders) and testing structure, we'll see about that later. 

Execute this to create the structure

```nohighlight
cookiecutter https://github.com/drivendata/cookiecutter-data-science
```

### Rules:

1. Data here is to be treated as immutable. Anyone should be able to reproduce the final products with only the code in `src` and the data in `data/raw`. If treated as immutable, you don't need to version control it hence its in the `.gitignore` file by default.

2. Notebooks are for exploration and communication. Don't collaborate on notebooks as their jsons aren't git friendly. Refactor useful parts from notebooks as part of `src`. There are 2 kinds of notebooks, exploratory (messy) ones and reports (which can be exported as beautiful htmls). The naming format for notebooks are `<step>-<ghuser>-<description>.ipynb` (e.g., `0.3-bull-visualize-distributions.ipynb`).

3. Store secrets (keys, credentials etc) in a `.env` file at root and **BE SURE TO ADD IT TO `.gitignore`**. Use something like [python-dotenv](https://github.com/theskumar/python-dotenv) to access the secrets in your code. Things like aws credentials can be stored in a .aws folder.

Here's the directory structure
		
```nohighlight
├── LICENSE
├── Makefile           <- Makefile with commands like `make data` or `make train`
├── README.md          <- The top-level README for developers using this project.
├── data
│   ├── external       <- Data from third party sources.
│   ├── interim        <- Intermediate data that has been transformed.
│   ├── processed      <- The final, canonical data sets for modeling.
│   └── raw            <- The original, immutable data dump.
│
├── docs               <- A default Sphinx project; see sphinx-doc.org for details
│
├── models             <- Trained and serialized models, model predictions, or model summaries
│
├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
│                         the creator's initials, and a short `-` delimited description, e.g.
│                         `1.0-jqp-initial-data-exploration`.
│
├── references         <- Data dictionaries, manuals, and all other explanatory materials.
│
├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures        <- Generated graphics and figures to be used in reporting
│
├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
│                         generated with `pip freeze > requirements.txt`
│
├── setup.py           <- Make this project pip installable with `pip install -e`
├── src                <- Source code for use in this project.
│   ├── __init__.py    <- Makes src a Python module
│   │
│   ├── data           <- Scripts to download or generate data
│   │   └── make_dataset.py
│   │
│   ├── features       <- Scripts to turn raw data into features for modeling
│   │   └── build_features.py
│   │
│   ├── models         <- Scripts to train models and then use trained models to make
│   │   │                 predictions
│   │   ├── predict_model.py
│   │   └── train_model.py
│   │
│   └── visualization  <- Scripts to create exploratory and results oriented visualizations
│       └── visualize.py
│
└── tox.ini            <- tox file with settings for running tox; see tox.readthedocs.io
```


## Llama-cpp-python
My God this note is a mess. Anyway I'm using it to document my work on the first implementation of the project. Llama-cpp-python is used to allow our vicuna llm to communicate with langchain. 

There were things mentioned about optimising the build process for my laptop but I think a simple `pip install llama-cpp-python` should do the trick.

[Here's](https://abetlen.github.io/llama-cpp-python/) the documentation for this. 

This also has a web server so it's compatible with OpenAI compatible clients.

You can adjust context-window and stuff here.

## Choosing the model
I'm looking at the [table](https://huggingface.co/TheBloke/Wizard-Vicuna-13B-Uncensored-GGML) of ggmls models we chose. I think 5 bit quantisation is the most reasonable (10-12 gb). For now, I'm choosing the 2 bit quantised model with least memory footprint for trials: `Wizard-Vicuna-13B-Uncensored.ggmlv3.q2_K.bin`. Afterwards I think we should go for the one with the highest accuracy and ram usage: `Wizard-Vicuna-13B-Uncensored.ggmlv3.q5_1.bin`

MY FUCKING GOD!!! IT'S FUCKING HEREEEEEEEEEEEE!!!!!!!!!!!!!!!!!!!!!!!!! It works!!! What's more, my RAM usage BARELY increased by 1 gb! Like, WTF?! I think llama-cpp-python is doing something under the hood for this. And it took 14 fucking seconds bro!

## Llama-cpp-python with Langchain
[Here](https://python.langchain.com/docs/modules/model_io/models/llms/integrations/llamacpp) the guide.

## Some stuff
- [ ] what's map reduce [here](https://python.langchain.com/docs/use_cases/summarization)
- [ ] haven't gone into the use of make files [here](https://drivendata.github.io/cookiecutter-data-science/)
- [ ] what's these 12 factor principles I'm hearing about??

## Warnings
- [ ] You're not using the same model as in the ranking but instead a 4/5 bit ggml model. Performance is expected to suffer.