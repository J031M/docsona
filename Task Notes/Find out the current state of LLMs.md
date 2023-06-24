# Introduction

I'm finding the right LLM.

You'll probably need to go through each line because some of them are todos.

I'm trying to query some of the models hosted on hugging face but it's taking forever. I've decided to go with  [wizard-vicuna-13b-uncensored](https://huggingface.co/TheBloke/Wizard-Vicuna-13B-Uncensored-GGML) since it has the best score in hugging face's open LLM rankings. ^modelchoice

# Compressions techniques

- **Quantisation**: reduce precision of parameters
- **Distillation**: training a smaller model to mimic a larger one
- **Pruning**: Removing unecessary parameters

## Quantisation

There are two ways to go about this

- **Weight quantisation**
If only weights are quantised then they'd need to be upscaled before passing it through the activations. Even if it takes less space on disk, upscaling means it's effect on ram doesn't change. Furthermore it might increase latency. Large models seem to have no loss in accuracy but smaller ones suffer.

- **Weight and activation quantisation**
Running the entire model in a lower precision means reduces model size, memory usage and inference times. WAQ affects larger LLMs more, sometimes to the point where the decrease in accuracy is more than the benefits of using a large one.

# Tools

## [QLoRA](https://github.com/artidoro/qlora)
An efficient way to finetune LLMs using single GPUs, uses bitsandbytes and is integrated with HF.

### [LangChain](https://github.com/hwchase17/langchain) 
Boy I'll be damned! This is a standard in an of itself. Beautiful.

### [Oobabooga](https://github.com/oobabooga/text-generation-webui)
The Automatic 1111 of LLMs

### [GGML](https://github.com/ggerganov/ggml)
A C library for LLMs using quantisation to allow them to run on CPU! [C Transformers](https://github.com/ggerganov/ggmlL) is its python wrapper that's compatible with langchain. My guess right now is that models are finetunes with QLoRA and then converted to GGML format. Damn I'm a genius. [This](https://github.com/ggerganov/llama.cpp/discussions/1595) was the discussion about fine-tuning GGML, I don't think there'

### [Llama.cpp](https://github.com/ggerganov/llama.cpp)

Allows for inferencing LLMs. Oobabooga front end supported. [Python bindings](https://github.com/abetlen/llama-cpp-python) exist too, which is compatible with langchain.

# LangChain for ChatPDF

**Retrieval augmented generation**: Your prompt is used to gather relevant parts of the document and then those parts with the prompt is fed to the LLM.

**Ingestion**: This is the stage where documents are indexed (so that it's easier to find relevant parts) typically using a vectorstore. Load -> Split -> Embed -> Store

**Generation**: Prompt -> lookup index -> modded prompt -> response

# Plan
Take wizard-vicuna-13b-uncensored and (finetune?) it using QLoRA and convert it into GGML to inference using llama.cpp with oogabooga front end. 

But... finetuning is unecassary so... Anyway, let's see what langchain has to say about this.

# Rambles

I would say since we're running it locally, it's better to use a cpu only model as setting up gpu and cuda is a pain for the average joe.

- [ ] You are using a quantised 13b model. Would it be better to use a 7b model that's finetuned to your use case instead?

It is suggested to quantise a medium sized (<10B params) LLM and finetune it for your use case.

I'm worried about the context length of this model. Being small, I doubt it can keep track of much, which isn't ideal for document summary. Maybe... have another way of serving up the relevant parts of a document for the LLM to summarise? But that can't handle complex queries.

There are different quantisations available for wizard-13b in the page. Since the wrapper below isn't compatible with the recent quants I'll stick to some of the older ones.

Fine tuning isn't necessary for a generic task like summarising documents, hmm. But what about converstions in the context of a document?

Still have no clue what instruct method is when training models.


# Resources

[Exceptional summaries focusing on MLops, LLMs and tinyMLops](https://tinyml.substack.com/) 
[LLM.int8()](https://arxiv.org/pdf/2208.07339.pdf) discusses the issue with large models and offers WQ soln
[GPTQ paper](https://arxiv.org/pdf/2210.17323.pdf) was able to 3bit quantise it 
[Effects of quantisation on LLMs, WQ vs WAQ](https://arxiv.org/pdf/2303.08302.pdf)
[Notebook on openai finetuning](https://colab.research.google.com/drive/1_sGhwQ5BrbNIt0NpCb3JSrjyc7zKLKDe?usp=sharingscrollTo=BRcpq-fbHOeA)
