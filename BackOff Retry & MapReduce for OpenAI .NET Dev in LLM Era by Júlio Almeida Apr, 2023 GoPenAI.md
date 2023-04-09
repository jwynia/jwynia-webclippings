# BackOff Retry & MapReduce for OpenAI: .NET Dev in LLM Era | by Júlio Almeida | Apr, 2023 | GoPenAI
[BackOff Retry & MapReduce for OpenAI: .NET Dev in LLM Era | by Júlio Almeida | Apr, 2023 | GoPenAI](https://blog.gopenai.com/backoff-retry-mapreduce-for-openai-net-dev-in-llm-era-e25b22dc7a59) 

 ![](https://miro.medium.com/v2/resize:fit:1400/1*RZMcvLHQSLwP4rxft8A20A.png)

I have been working with C# development and OpenAI for some time now, and this article summarizes the key conclusions I have drawn from my experiences. Since I started using Python and other tools like LangChain (which is more oriented towards data science), some of my solutions have improved significantly.

Additionally, there are many factors to consider when implementing solutions with OpenAI to ensure they are production-ready.

Motivations and Limitations
---------------------------

![](https://miro.medium.com/v2/resize:fit:1400/1*jZl14NqiYm6qvoaNaPYVJA.png)

This is the basic outlook for the several models. When making API requests we should focus on three points:

![](https://miro.medium.com/v2/resize:fit:1366/1*31YZMemhjwt0jrcDo_JI1A.png)

basic outlook for each model

Every model has its strengths and weaknesses, reflecting the classic engineering trade-off. Nowadays, using **Text-Davinci is not as advantageous in most cases**, since Chat essentially offers the same model but with **half the payload limit and only a tenth of the cost**. Unless you have other requirements like [Edit/Insert](https://openai.com/blog/gpt-3-edit-insert), there is little need to use Davinci. Admittedly, it is faster.

Moving forward, some aspects you should focus on, from an “API request basis,” include the **quality of the prompt**. Most of the time, a 3.5 rating will do the job. If not, you can use embeddings to assist in the search by providing a pattern of examples. If this still falls short, you can fine-tune the model. And if that is not enough, you can apply the same logic but with GPT-4, though this will increase the cost.

In conclusion, the majority of solutions tend to involve **GPT-3.5-Turbo combined with MapReduce**. This offers a balance of low cost, quality, and scalability. However, there are some loose ends to address, such as managing Requests per Minute.

BackOff, Retry, and Rate Limiters
---------------------------------

When your app scales, you may encounter the [429 status code problem](https://platform.openai.com/docs/guides/rate-limits/overview). This occurs when your account has reached the limit of Requests or Tokens per minute within a given time period — typically one minute. As mentioned earlier, different models have different limits.

While you could use model-agnostic techniques like [Middleware rate limiters](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-7.0), and set the requests to a 90% threshold, for example, to avoid overloading the system, it is advisable to **control this at the service level**. You need to consider not only the **Token count** but also the **various types of models** that can be selected for different use cases.

To solve this problem, you can use the solution below:

![](https://miro.medium.com/v2/resize:fit:1192/1*fo5tRq9yYzcJg81BjK0oqQ.png)

BackOffRetry component diagram

We can use Polly, with a policy to control all these limitations:

```
using Polly;

public class RetryWithBackOff  
{  
 private readonly IAsyncPolicy _retryPolicy;

 public RetryWithBackOff(int maxRetryAttempts, double exponentialBackOffFactor, TimeSpan initialDelay)  
 {  
  _retryPolicy = Policy  
   .Handle<Exception>()  
   .WaitAndRetryAsync(  
    retryCount: maxRetryAttempts,  
    sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(initialDelay.TotalMilliseconds * Math.Pow(exponentialBackOffFactor, attempt - 1)),  
    onRetryAsync: (outcome, timespan, retryAttempt, context) =>  
    {  
     ExceptionDigest(outcome, timespan, retryAttempt, context);  
     return Task.CompletedTask;  
    }  
   );  
 }

 private void ExceptionDigest(Exception outcome, TimeSpan timespan, int retryAttempt, Context context)  
 {  
    
    
 }

 public async Task<T\> ExecuteAsync<T>(Func<Task<T>> operation)  
 {  
  return await _retryPolicy.ExecuteAsync(operation);  
 }  
}


```

Polly tends to be super verbose, so let me make this clear:

*   Gets a number of retries as a limit
*   Gets the **backoff factor**, which is a fancy way to say “fails after waiting for a second? wait 2 or 3!”
*   Gets initial time to delay

You can go to the **catch** for several reasons, but here we want to take care of the 429 (Too many requests) status code. **Log the exception, and see if makes sense to ask to increase the quota**.

**To have several Retry managers, use this approach:**
------------------------------------------------------

Create a class to be used for the Dependency Injection:

![](https://miro.medium.com/v2/resize:fit:1400/1*mQYrU6tlw40FtIR2UJJJLQ.png)

Use this on the startup to register the services:

![](https://miro.medium.com/v2/resize:fit:1400/1*DggMLPBJ2DayFH_hb6B5MQ.png)

Now you can use several policies for different models,

![](https://miro.medium.com/v2/resize:fit:1242/1*ugI8IXMEIuLEvOcqbE7c_g.png)

External Libraries
------------------

API support has indeed come a long way, and there are now excellent third-party solutions available, such as [OpenAI-API-dotnet](https://github.com/OkGoDoIt/OpenAI-API-dotnet). These can be effectively combined with the aforementioned patterns.

A special shout-out goes to [aiqinxuancai](https://github.com/aiqinxuancai) for creating a replica of Tiktoken. It works exceptionally well and fulfills its intended purpose. This development is particularly beneficial for those who prefer to have some infrastructure in C# and not rely solely on external Python services.

```
using TiktokenSharp;

private static bool IsOverThreshold(string paragraphs, int offset)  
{  
  TikToken tikToken = TikToken.EncodingForModel(Model);  
  var i = tikToken.Encode(paragraphs);  
  var count = i.Count;  
  return count > MaxToken - offset;  
}


```

Does it work? The values are similar to those you see in the prompt on the playground or on the [tokenizer site](https://platform.openai.com/tokenizer).

MapReduce & Shuffle. Adapted from Langchain
-------------------------------------------

I have been genuinely impressed with how LangChain’s features have been designed. In terms of analogy, its structure is quite similar to what **Entity Framework has accomplished** — offering the ability to replace different components. One such component is MapReduce, which is more commonly found in other data science libraries. However, in the context of LangChain, it makes perfect sense to use MapReduce when addressing the token limit problem.

![](https://miro.medium.com/v2/resize:fit:1326/1*GqeQ5FnJ0q-s1P6CmppR5A.png)

Map reduce for this example

Splitting
---------

The first step is obviously to split the data. This is no big deal, of course, just make sure you split by a pattern (e.g “\\n”) and keep the size between the **TokenThreshold**. Here is a basic example:

![](https://miro.medium.com/v2/resize:fit:1400/1*T9ifRBXzyCFGGrSgyNE_Jg.png)

Mapping
-------

At this point, we should have something that resembles this:

![](https://miro.medium.com/v2/resize:fit:1290/1*TzeehhqUgNqLmomT424ggg.png)

Using the **RetryWithBackOff** and OpenAI request API, we can get the result of several batches and then take care the most tricky part, Reduce or Shuffle.

Reduce or Shuffle
-----------------

In a use case where we have a group of text that needs to be summarized, we can refer to this process as a “reduce”. This is yet another great feature from [LangChain](https://python.langchain.com/en/latest/reference/modules/chains.html?highlight=mapreduce#langchain.chains.MapReduceChain). Performing this task is simple:

![](https://miro.medium.com/v2/resize:fit:1400/1*HbQyXjEG9CQJLKLZMxSZ_g.png)

Reduce diagram with Summarization

If the text group is too large, you have other options such as using embeddings to accommodate a much larger payload. If that is also not suitable for the use case, you may need to perform multiple iterations.

However, if the data is not suitable for summarization and requires some form of aggregation instead, you can employ the following technique:

```
using Newtonsoft.Json;  
using Newtonsoft.Json.Linq;
```

```
private string JoinResults(List<string\> results)  
{  
    JObject mergedResult = new JObject();

    foreach (string result in results)  
    {  
        JObject parsedResult = JObject.Parse(result);

        foreach (var property in parsedResult.Properties())  
        {  
            if (!mergedResult.ContainsKey(property.Name))  
            {  
                mergedResult\[property.Name\] = property.Value;  
            }  
            else  
            {  
                JToken currentValue = mergedResult\[property.Name\];  
                JToken newValue = property.Value;

                if (currentValue.Type == JTokenType.Array && newValue.Type == JTokenType.Array)  
                {  
                    JArray currentArray = (JArray)currentValue;  
                    JArray newArray = (JArray)newValue;

                    foreach (JToken item in newArray)  
                    {  
                        currentArray.Add(item);  
                    }  
                }  
                else  
                {  
                    mergedResult\[property.Name\] = newValue;  
                }  
            }  
        }  
    }

    return mergedResult.ToString(Formatting.None);  
}


```

Visually, we end up with something like this:

![](https://miro.medium.com/v2/resize:fit:1400/1*rntwkv_Nq3yBwfhoToJCsw.png)

Gathering the data from the several requests

Shuffle, is not really the best name for this, but I like to keep the analogy going.

Conclusion
----------

The tools in C# are indeed somewhat basic compared to the extensive Python offerings, which is expected given Python’s popularity in the AI and data science domains. However, the current C# tools are sufficient to support a SaaS product as long as you have the Python stack to handle the remaining pieces. In the future, we can anticipate official adaptations and improvements, potentially stemming from the Microsoft Graph ecosystem, leveraging GPT-4 and other advanced AI technologies.