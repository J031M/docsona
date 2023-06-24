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
I think I've spent long enough just reading the wiki. I'm going to move on to the guide.

