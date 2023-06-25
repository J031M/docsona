[Presentation link](https://docs.google.com/presentation/d/1XOAV_Au52le07uXYJZQxmwT_GwvhFZjh/edit?usp=sharing&ouid=113458396771149391534&rtpof=true&sd=true)

1. List of all papers reviewed under this (if no notes just add paper n
2. ame):
   - [[Attention is all you need]]
   - [[LLM.int8()]]
3. List of Methods used in existing systems :
   - privateGPT
   - chatPDF
   - geniusPDF
   
 >Above 2 are entire projects that do the summarisation of documents.
   The methods to achieve this? Except privateGPT everyone else sends API calls to gpt-3.5. PrivateGPT uses [this](./Find%20out%20the%20current%20state%20of%20LLMs#[Llama.cpp](https://github.com/ggerganov/llama.cpp)) and [this](./Find%20out%20the%20current%20state%20of%20LLMs#[LangChain](https://github.com/hwchase17/langchain)) for running inferences locally. I'll go more into langchain in the methodology.
   
3. If any notes on feasibility of project add :
Our project can be as difficult as we want it to be.

Ideally we could train a multi-billion parameter model on a supercomputer cluster costing millions fine tuned on hoards of summarisation data.

We instead chose to use an [existing model](Find%20out%20the%20current%20state%20of%20LLMs#^modelchoice). The selection was based on the amount of RAM it would take and its performance on the hugging face open LLM rankings. (ellaborate on this ranking and how the benchmarking is done)

Our research suggests that it's feasible to finetune this model for our usecase but there isn't a major benefit since summarisation is a rather generic task. But if we've got time we would check to see if it's any better.

We're also using a [vector datastore](./Follow%20python%20guide%20for%20basic%20langchain%20implementation#Vector%20Stores) and [contextual compression](./Follow%20python%20guide%20for%20basic%20langchain%20implementation#Context%20compressors) to counteract the fact that we've got a limited context window with the 13b model.

5. Detailed discussion of problem and methodology (step/stage wise approach to solve the problem). Requires flow chart toos

Check this out: [[Follow python guide for basic langchain implementation]] for details.

Our model has 2 phases: ingestion and generation.

In ingestion we give it all our documents. It starts by splitting the documents in a way that preserves semantics and context. Then we vectorise the chunks of text into embeddings and store it in a vector database. 

In generation, the query is taken from the user and embedded in a similar way. We then do a similarity vector search to fetch the relevant chunks of the document. We can then use a context compressor as mentioned above to filter out any other irrelevant parts of the text. 

Finally this distilled information is fed into our LLM along with the query and it generates the answer. 

All this circus we do is because we don't want to pass the entire document to the LLM and call it a day. This method leads to lower latency, better accuracy and also lets us use smaller models effectively.

For flowchart, ran the methodology on GPT and found this as the most suiting answer for us :
Here is a suggested flowchart outlining the tasks and subtasks of your project based on the provided methodology:

1. Ingestion Phase:
   - Splitting Documents:
     - Identify and implement a semantic and context-preserving document splitting algorithm.
   - Text Embedding:
     - Select an appropriate text embedding technique.
     - Develop a pipeline to convert the text chunks into embeddings.
   - Vector Database:
     - Set up a vector database to store the generated embeddings.
     - Implement the necessary functions for efficient storage and retrieval.

2. Generation Phase:
   - Query Embedding:
     - Choose a suitable embedding method for queries.
     - Develop a process to embed user queries using the selected technique.
   - Similarity Vector Search:
     - Design and implement a similarity vector search algorithm.
     - Retrieve relevant text chunks based on the similarity search.
   - Context Compression:
     - Develop a context compression mechanism to filter out irrelevant information from the retrieved text chunks.
     - Implement the necessary algorithms to perform context compression effectively.

3. Answer Generation:
   - Language Model Integration:
     - Select an appropriate Language Model (LLM) for answer generation.
     - Integrate the LLM with the context-compressed input and user query.
   - Answer Generation:
     - Develop a pipeline to generate accurate and contextually appropriate answers using the LLM.
   - Post-processing:
     - Implement any necessary post-processing steps to refine the generated answers.

4. Overall Integration:
   - Workflow Integration:
     - Create a comprehensive workflow that connects the ingestion and generation phases.
   - User Interface:
     - Design and develop a user interface for interacting with the system.
   - System Testing and Evaluation:
     - Conduct rigorous testing to ensure the functionality and performance of the system.
     - Evaluate the system's accuracy, latency, and usability.