# Integrating ChatGPT with internal knowledge base and question-answer platform | by Quy Tang | Government Digital Services, Singapore | Mar, 2023 | Medium
[Integrating ChatGPT with internal knowledge base and question-answer platform | by Quy Tang | Government Digital Services, Singapore | Mar, 2023 | Medium](https://medium.com/singapore-gds/integrating-chatgpt-with-internal-knowledge-base-and-question-answer-platform-36a3283d6334) 

 
Bring the power of ChatGPT to internal knowledge management
-----------------------------------------------------------

![](https://miro.medium.com/v2/resize:fit:700/1*4W89CkaDECn5fZvqeq8hig.png)

**ChatGPT** is highly adept at providing general information, albeit with some limitations. Meanwhile, internal knowledge management is becoming more and more critical in a post-pandemic world with hybrid working and higher employee turnover, [according to Gartner](https://www.cio.com/article/302797/why-we-think-internal-knowledge-management-will-be-critical-in-2022-as-per-gartner.html). How can we bring the power of ChatGPT to internal knowledge management?

**In this post, I will outline a simple method to achieve this. In our case, we have an internal knowledge base and an internal Q&A platform similar to StackOverflow.**

Before we dive in, let me clarify some terms. ChatGPT is a chatbot powered by a series of GPTs, first GPT-3.5 and now GPT-4. GPTs (Generative pre-trained transformers) are language models under the big umbrella of Large Language Models (LLMs), which also include Google’s LaMDA and PaLM models (which power Bard chatbot) and open models like [BLOOM](https://bigscience.huggingface.co/blog/bloom) or [GPT-Neo-X](https://huggingface.co/EleutherAI/gpt-neox-20b). These are AI models purpose-built for various natural language processing tasks, such as text generation, classification, question-answering, summarisation, and translation.

> “Large” refers to the model’s size, which is measured by the number of parameters it contains. GPT-3 has 175 billion parameters, while GPT-4 is expected to have an order of magnitude more. Google’s LaMDA, which powers Bard, has 137 billion while the new model, PaLM, has [540 billion](https://ai.googleblog.com/2022/04/pathways-language-model-palm-scaling-to.html).

Let’s test ChatGPT with a simple query related to a product from our [Singapore Government Tech Stack (SGTS)](https://www.developer.tech.gov.sg/singapore-government-tech-stack/index.html).

Below is the response from ChatGPT (GPT-3.5):

![](https://miro.medium.com/v2/resize:fit:700/1*4YU7ZtiubGflR3U_giIdKQ.png)

At least it admits that it doesn’t know. Let’s try to provide more context:

![](https://miro.medium.com/v2/resize:fit:700/1*EFQSuwPWAPn-Ess4xAOmwA.png)

ChatGPT’s answer is horrendously wrong. SHIP stands for Secure Hybrid Integration Pipeline, while HATS Hive Agile Testing Solutions. They comprise [a suite of CICD tools](https://docs.developer.tech.gov.sg/docs/ship-hats-tools/tools-overview).

This situation with ChatGPT is known as bot hallucination, where the bot gives a grammatically correct but factually inaccurate answer.

To enhance an existing Large Language Model with custom knowledge, there are 2 main methods:

*   **Fine-tuning**: further training the model using the custom dataset.
*   **In context learning**: supplying the necessary information from the custom dataset related to user query while querying.

**Fine-tuning** provides a high degree of accuracy and completeness but it requires significant time and resources to train and host the custom model. **In-context learning**, on the other hand, offers greater flexibility and costs much less, but it’s limited by the model’s token limit.

> _As of writing, assuming a relatively-small custom dataset of 1M tokens (~200,000 words or ~400 Wikipedia articles) and an average query using 1000 tokens,_ [_fine-tuned Davinci_](https://openai.com/pricing) _costs_ **_$30,000 for training_** _and_ **_12 cents per query_**_. In contrast, in-context learning costs_ **_<$1 for embedding_**_, and at most_ **_1 cent per query_** _(assuming 4000 tokens per query due to additional context)._
> 
> _According to some sources, ChatGPT has been trained on ~500 billion words. That is 2.5 million times of the sample size above._

In-context learning is clearly a simpler solution.

Prompt Engineering and **Retrieval Augmented Generation**
---------------------------------------------------------

We can already enhance the quality of ChatGPT’s answer simply by providing it additional context. There’s a whole new discipline dedicated to this called [**prompt engineering**](https://en.wikipedia.org/wiki/Prompt_engineering), aimed at developing and optimising prompts to use language models efficiently.

Prompt engineering applied to the case of an internal knowledge base means feeding relevant data from the knowledge base to ChatGPT every time we interact with it. You can imagine how troublesome this can quickly become. The process needs to be automated.

This is where [**Retrieval Augmented Generation**](https://huggingface.co/docs/transformers/model_doc/rag) workflow comes in.

The idea is simple. Instead of asking a question directly, the process first uses the user question to perform a search to retrieve relevant documents from the internal dataset and then provides these documents together with the question to ChatGPT. With the additional context, ChatGPT can answer **as though it has been trained with the internal dataset**.

So roughly speaking, instead of simply `<query>` it’d be:

`answer following question given <relevant texts>, <query>`

Here’s a simple diagram for this process:

![](https://miro.medium.com/v2/resize:fit:700/1*MSCEwcGU-1eRXzusS_iTpg.png)

Following is an implementation of RAG Q&A in Python. The Jupiter notebook is hosted [on Google Colab](https://colab.research.google.com/drive/129VMF1HG24I59taVcn7d2ZWcLbHqKhtB#scrollTo=NANhNz315A6w).

The dataset
-----------

For demo purpose, I’m using public data provided by GovTech SGTS team. These are documents for `[https://www.developer.tech.gov.sg](https://www.developer.tech.gov.sg/)` hosted in the GitHub repo `[https://github.com/GovTechSG/developer.gov.sg](https://github.com/GovTechSG/developer.gov.sg).`

The workflow (aka chain)
------------------------

I use the [**Langchain**](https://github.com/hwchase17/langchain) library, which enables chaining together multiple capabilities including integration with LLMs (e.g. ChatGPT). In this case, it allows me to implement the Retrieval-Augmented Generation Q&A process as described above. Alternatives to Langchain include [LlamaIndex](https://github.com/jerryjliu/llama_index/blob/main/examples/knowledge_graph/KnowledgeGraphDemo.ipynb) (built on top of Langchain) and [Haystack](https://docs.haystack.deepset.ai/docs/answer_generator).

The document database
---------------------

The document database we need to build is going to be a vector database instead of a traditional database.

> _A vector database indexes and stores vector embeddings for fast retrieval and similarity search, with capabilities like CRUD operations, metadata filtering, and horizontal scaling. Vector databases excel at_ [_similarity search_](https://www.pinecone.io/learn/what-is-similarity-search/)_, or “vector search.” Vector search enables users to describe what they want to find without having to know which keywords or metadata classifications are ascribed to the stored objects. Vector search can also return results that are similar or near-neighbor matches, providing a more comprehensive list of results that otherwise may have remained hidden. —_ [_https://www.pinecone.io/learn/vector-database/_](https://www.pinecone.io/learn/vector-database/)

To store documents as vectors, a vector database requires a process called embedding to convert each word into a vector of hundreds or thousands of different dimensions. For example, OpenAI Ada embedding results in over 1500 dimensions.

This also means that building a database incurs some costs, depending on the size of the database. But this is negligent compared to the training cost of a fine-tuned model. With 1,000,000 tokens, it costs only 40 cents (contrast that with $30,000 for training).

I use the [FAISS](https://github.com/facebookresearch/faiss) library as the database, which stands for Facebook AI Similarity Search. I can save the FAISS database to local files and load it later on for querying. This cuts down the cost of building the database. Other than FAISS, Langchain supports [Chroma](https://www.trychroma.com/), [Pinecone](https://www.pinecone.io/), [Weaviate](https://weaviate.io/), [OpenSearch](https://opensearch.org/docs/latest/search-plugins/knn/index/), and [many others](https://python.langchain.com/en/latest/modules/indexes/vectorstores/getting_started.html).

Building the database
---------------------

**Install required packages**

Install `langchain` `openai` `faiss-cpu` packages.

**Set up OPEN\_API\_KEY and necessary variables**

```
import os  
from getpass import getpass

os.environ\["OPENAI\_API\_KEY"\] = getpass("Paste your OpenAI API key here and hit enter:") 

REPO_URL = "https://github.com/GovTechSG/developer.gov.sg"    
DOCS_FOLDER = "docs"    
REPO\_DOCUMENTS\_PATH = "collections/_products/categories/devops/ship-hats"    
DOCUMENT\_BASE\_URL = "https://www.developer.tech.gov.sg/products/categories/devops/ship-hats"    
DATA\_STORE\_DIR = "data_store" 
```

I specify a small subset with the path to keep the database small for testing. In actual application, I include everything under `collections/_products`.

**Clone the GitHub repo**

`git clone $REPO_URL $DOCS_FOLDER`

**Load documents and split them into chunks for conversion to embeddings**

```
repo\_path = pathlib.Path(os.path.join(DOCS\_FOLDER, REPO\_DOCUMENTS\_PATH))  
document_files = list(repo\_path.glob(name\_filter))

def convert\_path\_to\_doc\_url(doc_path):  
    
  return re.sub(f"{DOCS_FOLDER}/{REPO\_DOCUMENTS\_PATH}/(.*)\\.\[\\w\\d\]+", f"{DOCUMENT\_BASE\_URL}/\\\1", str(doc_path))

documents = \[  
    Document(  
        page_content=open(file, "r").read(),  
        metadata={"source": convert\_path\_to\_doc\_url(file)}  
    )  
    for file in document_files  
\]

text\_splitter = CharacterTextSplitter(separator=separator, chunk\_size=chunk\_size\_limit, chunk\_overlap=max\_chunk_overlap)  
split\_docs = text\_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()  
vector\_store = FAISS.from\_documents(split_docs, embeddings)


```

That’s it, we have `vector_store` as our database. The documents are split into chunks of roughly 1000 tokens each and then sent to OpenAI for embedding before being stored in the FAISS database.

> _An optional step is to save this to local files for future reuse with:_ `_vector_store.save_local(DATA_STORE_DIR)_`
> 
> _And reload it using:_ `_vector_store = FAISS.load_local(DATA_STORE_DIR, OpenAIEmbeddings())_`

Now that we have built the database, this is the fun part where we can query our custom data.

**Set up the chat model and specific prompt**

```
from langchain.prompts.chat import (  
    ChatPromptTemplate,  
    SystemMessagePromptTemplate,  
    HumanMessagePromptTemplate,  
)

system_template="""Use the following pieces of context to answer the users question.  
Take note of the sources and include them in the answer in the format: "SOURCES: source1 source2", use "SOURCES" in capital letters regardless of the number of sources.  
If you don't know the answer, just say that "I don't know", don't try to make up an answer.  
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-  
{summaries}"""  
messages = \[  
    SystemMessagePromptTemplate.from\_template(system\_template),  
    HumanMessagePromptTemplate.from_template("{question}")  
\]  
prompt = ChatPromptTemplate.from_messages(messages)

from langchain.chat_models import ChatOpenAI  
from langchain.chains import RetrievalQAWithSourcesChain

chain\_type\_kwargs = {"prompt": prompt}  
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0, max_tokens=256)    
chain = RetrievalQAWithSourcesChain.from\_chain\_type(  
    llm=llm,  
    chain_type="stuff",  
    retriever=vector\_store.as\_retriever(),  
    return\_source\_documents=True,  
    chain\_type\_kwargs=chain\_type\_kwargs  
)

def print_result(result):  
  output_text = f"""### Question:   
  {query}  
  ### Answer:   
  {result\['answer'\]}  
  ### Sources:   
  {result\['sources'\]}  
  ### All relevant sources:  
  {' '.join(list(set(\[doc.metadata\['source'\] for doc in result\['source_documents'\]\])))}  
  """


```

The main part of the above code is the setup of the `RetrievalQAWithSourcesChain` object with OpenAI’s `gpt-3.5-turbo` model as LLM and our `vector_store` database as the retriever. The prompt can be further customised for different use cases. Note also that we set the model’s temperature to 0 to make it stick to the context.

**Use the chain to query**

```
query = "What is SHIP-HATS?"  
result = chain(query)  
print_result(result)
```

**Result**

![](https://miro.medium.com/v2/resize:fit:700/1*QQENQGVGXs4jnzicrkgz2w.png)

There we have it, the answer is correct.

In addition, it also provides the source data, which is this page: [https://www.developer.tech.gov.sg/products/categories/devops/ship-hats/overview](https://www.developer.tech.gov.sg/products/categories/devops/ship-hats/overview)

This is extremely useful for the user to verify the answer when in doubt or to read up and find out more about the search topic. **This is an important advantage over plain vanilla ChatGPT**.

**What happened behind the scene? — OpenAI API call**

To understand what happened, it’s useful to look at the payload for the OpenAI API call here:

![](https://miro.medium.com/v2/resize:fit:700/1*kiHoHz0chbcESyNo6bDf3A.png)

Relevant chunks of texts and their source have been added to the `system` message. This is the secret to the context-aware answer from ChatGPT.

> _The complete Jupiter notebook is hosted_ [_on Google Colab_](https://colab.research.google.com/drive/129VMF1HG24I59taVcn7d2ZWcLbHqKhtB)_._

We have had a glimpse at bringing ChatGPT to internal knowledge management. As of writing, I am working on many additional enhancements.

**Integration as a Telegram or Slack Bot**
------------------------------------------

Now that we have the main workflow done, we can easily create a bot for it on platforms like Telegram or Slack. The bot is easily accessible by our engineering teams who use either Telegram or Slack as main communication tools.

Here’s an example on Telegram:

![](https://miro.medium.com/v2/resize:fit:700/1*Id8QRfo4LXof419uuq2ItQ.png)

**Integration with Q&A Platform**
---------------------------------

In my department, we have an internal Q&A platform codenamed Hivemind, which is similar to StackOverflow. The bot can be trained with additional data from Hivemind.

Furthermore, the system can allow users to select question+answer pairs to be posted to Hivemind and to post questions that the bot fails to answer on Hivemind. Over time, the bot will have a sizeable set of answers and become better as our central knowledge guru.

This diagram describes the integration between the bot and Hivemind:

![](https://miro.medium.com/v2/resize:fit:679/1*mkkVtXWje1BaFGCFbiHv7g.png)

Enhancing Data Security
-----------------------

A majority of our internal knowledge base is sensitive. As a result, using an LLM service like ChatGPT doesn’t satisfy data security requirements since a subset of the data is sent to OpenAI. We are also considering different ways to host an LLM model in our Government cloud.

We have only touched the tip of the iceberg in the process of bringing LLM power to internal knowledge management. There’s much more to do with this, including using different LLMs or locally hosted models for data security as mentioned above, using human-assisted answers to improve accuracy and adding multimodal capabilities to support images, videos, speech.

As demonstrated, LLMs provide the potential to build powerful applications very quickly. We are only at the very beginning of a Cambrian explosion of LLM-powered applications. At the same time, there’re numerous [ethical and social risks](https://www.deepmind.com/publications/ethical-and-social-risks-of-harm-from-language-models) with language models. We can either stay out of the race or join it to better comprehend the powers and implications so that we can contribute meaningfully to the safe advancement of AI.

_Thank you for reading. Do comment below to share your thoughts._

_The complete Jupiter notebook is hosted_ [_on Google Colab_](https://colab.research.google.com/drive/129VMF1HG24I59taVcn7d2ZWcLbHqKhtB)_._