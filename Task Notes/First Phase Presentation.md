[Presentation link](https://docs.google.com/presentation/d/1XOAV_Au52le07uXYJZQxmwT_GwvhFZjh/edit?usp=sharing&ouid=113458396771149391534&rtpof=true&sd=true)

1. List of all papers reviewed under this (if no notes just add paper name):
   - [[Attention is all you need]]
   - LLM.int8() (Arindam's git isn't working, I'll add the summary when he sends)
2. List of Methods used in existing systems :
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