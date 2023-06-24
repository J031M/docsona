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

