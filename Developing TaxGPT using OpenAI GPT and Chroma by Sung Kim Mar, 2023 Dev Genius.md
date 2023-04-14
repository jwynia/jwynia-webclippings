# Developing TaxGPT using OpenAI GPT and Chroma | by Sung Kim | Mar, 2023 | Dev Genius
[Developing TaxGPT using OpenAI GPT and Chroma | by Sung Kim | Mar, 2023 | Dev Genius](https://blog.devgenius.io/developing-taxgpt-using-openai-gpt-and-chroma-548c23ae7657) 

 
Developing TaxGPT application that can answer complex tax questions for tax professionals
-----------------------------------------------------------------------------------------

![](https://miro.medium.com/v2/resize:fit:1400/0*x0uh0sm6mv_RsvPm)

Microsoft Bing Image Creator (Tax Accountant Using AI)

On March 14, 2023, Greg Brockman from OpenAI introduced an example of “TaxGPT,” in which he used GPT-4 to ask questions about taxes. As a tax accountant in my past life, I decided to create a better version of TaxGPT. Instead of copying and pasting tax codes into GPT-4, I have decided to create an embedding database of both the Internal Revenue Codes and Internal Revenue Regulations. This database will allow me to ask complex tax questions, such as income taxes for U.S. citizens and resident aliens abroad.

I became curious about the feasibility of this endeavor because I had always believed that the Internal Revenue Code was written using some sort of cryptography that I couldn’t decipher. Additionally, my past experiences had shown me that the Internal Revenue Regulations were not very helpful. If GPT could decipher these texts successfully, it should be capable of decoding any text.

TaxGPT is a tax application for tax professionals, as they need to know what questions to ask. It’s similar to non-developers asking ChatGPT to develop an application for them; it can be done, but the results may be poor.

To answer a specific tax-related inquiry, several steps were taken.

*   Firstly, an embedding database of the Internal Revenue Codes was created, which was scraped from Bloomberg Tax.
*   Secondly, an embedding database of the Internal Revenue Regulations was created, which was scraped from Internal Revenue Service.
*   Subsequently, the embedding database of Internal Revenue Codes was queried using the tax question, which will yield a list of applicable Internal Revenue Codes (I.R.C.). This was done to assist in querying Internal Revenue Regulations since that database is big.
*   This list of I.R.C . was then appended to the tax question, and the embedding database of Internal Revenue Regulations was queried.
*   Finally, using GPT on search results, an answer that includes relevant citations can be generated.

By following these steps, I hope that a comprehensive and accurate response to any tax query can be provided.

This code was developed using Google Colab to test its feasibility. If you believe this tool could be useful, please leave a comment so I can commence developing a web-based application that can be used by anyone.

Prerequisites
-------------

The following are prerequisites for this tutorial:

*   Python Package: `openai`
*   Python Package: `tiktoken`
*   Python Package: `chromadb`
*   Python Package: `langchain`
*   Python Package: `unstructured`

1.  Install Python Packages

```
%%writefile requirements.txt  
openai  
chromadb  
tiktoken  
langchain
```

```
%pip install -r requirements.txt
```

2\. Import Python Packages

```
import os  
import platform

import requests  
from bs4 import BeautifulSoup  
from urllib.parse import urljoin  
import time

import openai  
import tiktoken  
import langchain  
import chromadb  
chroma_client = chromadb.Client()

from langchain.embeddings.openai import OpenAIEmbeddings  
from langchain.vectorstores import Chroma  
from langchain.text_splitter import TokenTextSplitter  
from langchain.llms import OpenAI  
from langchain.document_loaders import UnstructuredURLLoader

import urllib  
from urllib import request

print('Python: ', platform.python_version())


```

3\. Crawl and scrape Internal Revenue Codes

```
def get\_all\_links(url, prefix):  
    try:  
        response = requests.get(url)  
        if response.status_code != 200:  
            print(f"Failed to load the page. Status code: {response.status_code}")  
            return \[\]

        soup = BeautifulSoup(response.text, "html.parser")  
        links = \[\]

        for link in soup.find_all("a"):  
            href = link.get("href")  
            if href:  
                absolute_url = urljoin(url, href)  
                if absolute_url.startswith(prefix):  
                    links.append(absolute_url)

        return links

    except Exception as e:  
        print(f"Error: {e}")  
        return \[\]

def crawl(url, prefix, depth, visited=None):  
    if visited is None:  
        visited = set()

    if depth == 0:  
        return visited

    links = get\_all\_links(url, prefix)  
    visited.add(url)

    for link in links:  
        if link not in visited:  
            time.sleep(1)    
            visited = crawl(link, prefix, depth - 1, visited)

    return visited


```

```
url = "https://irc.bloombergtax.com/"    
url_prefix = "https://irc.bloombergtax.com/public/uscode/toc/irc/"  
max_depth = 5    
visited\_urls = crawl(url, url\_prefix, max_depth)  
print("List of URLs:")  
for visited_url in visited_urls:  
    print(visited_url)
```

3\. Crawl and scrape Internal Revenue Regulations

```
url_prefix = "https://www.irs.gov/irb/"  
max_depth = 2  

visited_urls = set()  
for page_number in range(21):  
    url = f"https://www.irs.gov/irb?page={page_number}"    
    visited\_urls = visited\_urls.union(crawl(url, url\_prefix, max\_depth))

print("List of URLs:")  
for visited_url in visited_urls:  
    print(visited_url)


```

4\. Set Configurations

4.1. Mount Storage — Google Drive

```
from google.colab import drive  
drive.mount('/content/drive')
```

4.2. Set OpenAI API Key

```
os.environ\["OPENAI\_API\_KEY"\] = "openai api key"  
openai.api_key = os.getenv("OPENAI\_API\_KEY")
```

5\. Create Internal Revenue Codes Embedding Database

5.1. List Bloomberg Tax Internal Revenue Codes URLs

```
urls = \[  
   "https://irc.bloombergtax.com/public/uscode/toc/irc/subtitle-f/chapter-65/subchapter-b",  
    "https://irc.bloombergtax.com/public/uscode/toc/irc/subtitle-d/chapter-50",  
    "https://irc.bloombergtax.com/public/uscode/doc/irc/section_4907",  
    "https://irc.bloombergtax.com/public/uscode/doc/irc/section_1411",  
    "https://irc.bloombergtax.com/public/uscode/toc/irc/subtitle-f/chapter-61/subchapter-b"  
\]
```

_Notes: Please note that I have only listed 5 URLs above. There are about a hundred URLs for Bloomberg Tax Internal Revenue Codes. It should cost about $2 to create this embedding database, payable to OpenAI._

5.2. Load Bloomberg Tax Internal Revenue Codes

```
loader = UnstructuredURLLoader(urls=urls)  
irc_data = loader.load()
```

5.3. Create Internal Revenue Codes embeddings database

```
collection_name="irc"  
persist_directory="/content/drive/My Drive/Colab Notebooks/chromadb/tax"

text\_splitter = TokenTextSplitter(chunk\_size=1000, chunk_overlap=0)  
irc\_doc = text\_splitter.split\_documents(irc\_data)

embeddings = OpenAIEmbeddings()  
irc\_db = Chroma.from\_documents(irc\_doc, embeddings, collection\_name=collection\_name, persist\_directory=persist_directory)  
irc_db.persist()


```

![](https://miro.medium.com/v2/resize:fit:1400/1*1LVexCuipcqtDmIWg-F4zA.png)

6\. Create Internal Revenue Regulations Embedding Database

5.1. List Internal Revenue Service Internal Revenue Regulations URLs

```
urls = \[  
    "https://www.irs.gov/irb/2005-38_IRB",  
    "https://www.irs.gov/irb/2009-30_IRB",  
    "https://www.irs.gov/irb/2017-23_IRB",  
    "https://www.irs.gov/irb/2013-07_IRB",  
    "https://www.irs.gov/irb/2006-52_IRB"\]
```

_Notes: Please note that I have only listed 5 URLs above. There are hundreds more URLs for Internal Revenue Regulations. It should cost about $20 to create this embedding database, payable to OpenAI._

5.2. Load Internal Revenue Service Internal Revenue Regulations

```
loader = UnstructuredURLLoader(urls=urls)  
irc_data = loader.load()
```

5.3. Create Internal Revenue Service Internal Revenue Regulations embeddings database

```
collection_name="irb"  
persist_directory="/content/drive/My Drive/Colab Notebooks/chromadb/tax"
```

```
text\_splitter = TokenTextSplitter(chunk\_size=1000, chunk_overlap=0)  
irc\_doc = text\_splitter.split\_documents(irb\_data)embeddings = OpenAIEmbeddings()  
irc\_db = Chroma.from\_documents(irb\_doc, embeddings, collection\_name=collection\_name, persist\_directory=persist_directory)  
irc_db.persist()
```

![](https://miro.medium.com/v2/resize:fit:1400/1*agUGpzrWiiX7TB2bNolGdA.png)

Chroma uses both DuckDB and Apache Parquet for its backend where it creates two .parquet files. Refer to this article [Modularize SQL in Jupyter Notebooks Using DuckDB](https://medium.com/geekculture/sql-notebooks-for-data-analytics-a051f3693742) for details on how to perform the following analysis.

*   chroma-collections.parquet

![](https://miro.medium.com/v2/resize:fit:1400/1*WRKyNwlvwBKDkugFq6IlYQ.png)

*   chroma-embeddings.parquet

![](https://miro.medium.com/v2/resize:fit:1400/1*p4nDPr_DJf6OBUjZ1BNChA.png)

There are only two records in “chroma-collections.parquet”, which corresponds to two collections we have created.

![](https://miro.medium.com/v2/resize:fit:1400/1*uZ0pWHRsI0LQGb5C9zUEuQ.png)

Embeddings are stored in “chroma-embeddings.parquet” with a foreign key back to “chroma-collections.parquet”.

![](https://miro.medium.com/v2/resize:fit:1400/1*Rd7agmotLi7rLlR1SOvMEA.png)

Finally, here is a sample view of “ “chroma-embeddings.parquet”.

![](https://miro.medium.com/v2/resize:fit:1400/1*6lMAMN1eLylbqxc9Ydxh4Q.png)

7\. Load Chroma Database

7.1. Load Database

```
from chromadb.config import Settings  
client = chromadb.Client(Settings(  
    chroma\_db\_impl="duckdb+parquet",  
    persist_directory="/content/drive/My Drive/Colab Notebooks/chromadb/tax/"   
))
```

7.2. Load Collections

```
embeddings = openai.Embedding()  
irc\_collection = client.get\_collection(name="irc", embedding_function=embeddings)  
irb\_collection = client.get\_collection(name="irb", embedding_function=embeddings)
```

8\. Define Helper Functions

8.1. Transform Text to Embeddings

```
def get_embedding(text, model="text-embedding-ada-002"):  
   text = text.replace("\\n", " ")  
   return openai.Embedding.create(input = \[text\], model=model)\['data'\]\[0\]\['embedding'\]
```

_Please note that a helper function is required to query the embedding database. To do so, all text must be transformed into embeddings using OpenAI’s embedding models, after which the embeddings can be used to query the embedding database. This process is essential for obtaining accurate and reliable results._

8.2 Breakup Text to Chunks

```
def break\_up\_text\_to\_chunks(text, chunk_size=2000, overlap_size=100):  
    encoding = tiktoken.get_encoding("gpt2")

    tokens = encoding.encode(text)  
    num_tokens = len(tokens)

    chunks = \[\]  
    for i in range(0, num\_tokens, chunk\_size - overlap_size):  
        chunk = tokens\[i:i + chunk_size\]  
        chunks.append(chunk)

        return chunks


```

_It should be noted that a helper function is necessary because the query result from the embedding database may be quite large, exceeding the 4,096 token limit of OpenAI’s models. This function is crucial for ensuring that all relevant information is captured and that the results are accurate and comprehensive._

9\. Ask TaxGPT

This Python function consists of several steps that are necessary for processing and analyzing tax-related inquiries. Each step serves a specific purpose, and they all work together to provide accurate and reliable results. I guess I could have modularized the code, but I just wanted to do something quick to see the results.

*   Firstly, the function converts the question into embeddings.
*   Secondly, the Internal Revenue Code (IRC) database is queried using these embeddings, returning up to ten relevant results. These results are then combined into a single text.
*   Thirdly, GPT is used to return a list of only the most relevant IRCs that pertain to the given question. This list is then appended to the original question, which is once again converted into embeddings.
*   Fourthly, the Internal Revenue Regulations (IRB) database is queried using these embeddings, returning up to twenty relevant results, which are also combined into a single text.
*   Finally, GPT is used to generate answers that reference the IRCs and cover the given question comprehensively and accurately.

_Notes: I could have used LangChain for the following set of codes, but I did not want to deal with the additional complexity that comes with using LangChain._

```
def askTaxGPT(question, debug = False):

      
    irc\_question\_ids = get_embedding(question)

      
    irc\_query\_results = irc_collection.query(  
        query\_embeddings=irc\_question_ids,  
        n_results=10,  
        include=\["documents"\]  
    )

      
    irc\_documents = irc\_query_results\["documents"\]\[0\]  
    irc\_query\_results_doc = "".join(irc_documents)

    if debug == True:  
        print(irc\_query\_results_doc)

      
    prompt_response = \[\]  
    encoding = tiktoken.get_encoding("gpt2")  
    chunks = break\_up\_text\_to\_chunks(irc\_query\_results_doc)

    for i, chunk in enumerate(chunks):  
        prompt_request = question + " Only return a list relevant Internal Revenue Codes that covers this topic.: " \+ encoding.decode(chunks\[i\])  
          
        response = openai.Completion.create(  
                model="text-davinci-003",  
                prompt=prompt_request,  
                temperature=0,  
                max_tokens=1000,  
                top_p=1,  
                frequency_penalty=0,  
                presence_penalty=0  
        )          
        prompt_response.append(response\["choices"\]\[0\]\["text"\].strip())

      
    prompt_request = "Consoloidate these a list of Internal Revenue Codes: " \+ str(prompt_response)

    if debug == True:  
        print(prompt_request)

    response = openai.Completion.create(  
            model="text-davinci-003",  
            prompt=prompt_request,  
            temperature=0,  
            max_tokens=1000,  
            top_p=1,  
            frequency_penalty=0,  
            presence_penalty=0  
        )

        irc_codes = response\["choices"\]\[0\]\["text"\].strip()

    if debug == True:  
        print(prompt_request)

      
    irb\_question\_ids = get\_embedding(question + irc\_codes)

      
    irb\_query\_results = irb_collection.query(  
        query\_embeddings=irb\_question_ids,  
        n_results=20,  
        include=\["documents"\]  
    )

      
    irb\_documents = irb\_query_results\["documents"\]\[0\]  
    irb\_query\_results_doc = "".join(irb_documents)

    if debug == True:  
        print(irb\_query\_results_doc)

      
    prompt_response = \[\]  
    encoding = tiktoken.get_encoding("gpt2")  
    chunks = break\_up\_text\_to\_chunks(irb\_query\_results_doc)

    for i, chunk in enumerate(chunks):  
        prompt_request = question + " Cite I.R.C. as references." \+ encoding.decode(chunks\[i\])  
          
        response = openai.Completion.create(  
                model="text-davinci-003",  
                prompt=prompt_request,  
                temperature=0,  
                max_tokens=1000,  
                top_p=1,  
                frequency_penalty=0,  
                presence_penalty=0  
        )          
        prompt_response.append(response\["choices"\]\[0\]\["text"\].strip())

        if debug == True:  
        print(prompt_request)

      
    prompt_response = \[\]  
    encoding = tiktoken.get_encoding("gpt2")  
    chunks = break\_up\_text\_to\_chunks(str(prompt_response))

    for i, chunk in enumerate(chunks):  
        prompt_request = question + " Cite I.R.C. as references." \+ encoding.decode(chunks\[i\])  
        response = openai.Completion.create(  
                model="text-davinci-003",  
                prompt=prompt_request,  
                temperature=0,  
                max_tokens=1000,  
                top_p=1,  
                frequency_penalty=0,  
                presence_penalty=0              
            )    

                return response\["choices"\]\[0\]\["text"\].strip()


```

10\. Questions and Answers with TaxGPT

10.1. The first question is about digital nomads.

```
answer = askTaxGPT("I am USA citizen who is working for a USA-based company, but lived outside of USA for the calendar year. Do I still need to pay income tax?")
```

![](https://miro.medium.com/v2/resize:fit:1400/1*-wFvVFgaYYkRbjTjeCPhXQ.png)

```
answer
```

```
'In summary, a USA citizen who is working for a USA-based company and living outside of the USA for the calendar year is still required to pay income tax. The Internal Revenue Code (I.R.C.) does not exclude income derived in the U.S. from taxable income. The I.R.C. defines a taxpayer as any person subject to any internal revenue tax and further defines a person as an individual, trust, estate, partnership, association, company or corporation. Taxpayers are required to disclose foreign financial accounts to the Treasury Department and to report the income earned thereon. Taxpayers must report income earned as an individual on a Form 1040 and may not attribute the income to a trust created solely for the purpose of tax-avoidance, or claim deductions related to any expenses purportedly incurred by such a trust. Business expenses, including expenses related to a home-based business, are not deductible unless the expenses relate to a legitimate profit-seeking trade or business. Taxpayers are required to file returns and pay any tax owed. Additionally, taxpayers may be subject to additional taxes or penalties if they fail to comply with the I.R.C. and related regulations.'
```

10.2. The second question is about digital nomads who live in Colombia and pay Colombian taxes.

```
answer = askTaxGPT("I am USA citizen who Lives in Colombia and is working for a USA-based company for the calendar year. I am required to pay income tax to Colombian government as part of my visa. Do I still need to pay income tax to USA?")
```

![](https://miro.medium.com/v2/resize:fit:1400/1*VVOoHzdI5_v5tml0DcEwYA.png)

```
answer
```

```
'In conclusion, a USA citizen who lives in Colombia and is working for a USA-based company for the calendar year is required to pay income tax to the Colombian government as part of their visa. However, they are still required to pay income tax to the USA, as there is no exclusion from taxable income under the Internal Revenue Code (IRC). The IRC provides that a dual resident taxpayer who is treated as a nonresident alien for purposes of computing their US tax liability is not considered a US person for the taxable year. Furthermore, the IRC defines withholding as the deduction and withholding of tax at the applicable rate from a payment. Finally, the IRC provides that a civilian spouse who claims tax residence in a US territory under the Military Spouses Residency Relief Act (MSRRA) and is a bona fide resident of the US territory is not required to file a US federal income tax return or pay US federal income tax on income derived from sources within the US territory. For more information, please refer to IRS Publication 570.'
```

10.3. The third question is about selling restricted stock units.

```
answer = askTaxGPT("What are the tax implications of exercising stock options or selling restricted stock units (RSUs)?")
```

![](https://miro.medium.com/v2/resize:fit:1400/1*dDpx4PvySCi1gBfanDBESQ.png)

```
answer
```

```
'Exercising stock options and selling restricted stock units (RSUs) both have tax implications. \\n\\nWhen exercising stock options, the difference between the exercise price and the fair market value of the stock at the time of exercise is considered ordinary income and is subject to income tax. This is known as the “bargain element” and is reported on Form 1099-MISC. The bargain element is also subject to employment taxes such as Social Security and Medicare taxes.\\n\\nWhen selling RSUs, the fair market value of the stock at the time of sale is considered ordinary income and is subject to income tax. This is reported on Form 1099-MISC. The sale of RSUs is also subject to employment taxes such as Social Security and Medicare taxes.\\n\\nThe Internal Revenue Code (IRC) sections that apply to the taxation of stock options and RSUs are sections 83 and 451. Section 83 of the IRC states that the difference between the exercise price and the fair market value of the stock at the time of exercise is considered ordinary income and is subject to income tax. Section 451 of the IRC states that the fair market value of the stock at the time of sale is considered ordinary income and is subject to income tax.'
```

I hope you have enjoyed this article. If you have any questions or comments, please provide them here.

**Resources**

*   [Bloomberg Tax](https://pro.bloombergtax.com/)
*   [IRS Online Bulletins](https://www.irs.gov/irb)