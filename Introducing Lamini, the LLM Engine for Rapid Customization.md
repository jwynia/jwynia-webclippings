# Introducing Lamini, the LLM Engine for Rapid Customization
#### Training LLMs should be as easy as prompt-tuning 🦾

Why is writing a prompt so easy, but training an LLM from a base model still so hard? Iteration cycles for fine-tuning on modest datasets are measured in months because it takes significant time to figure out why fine-tuned models fail. Conversely, prompt-tuning iterations are on the order of seconds, but performance plateaus in a matter of hours. Only a limited amount of data can be crammed into the prompt, not the terabytes of data in a warehouse. 

It took OpenAI months with an incredible ML team to fine-tune and run RLHF on their base GPT-3 model that was available for years — creating what became ChatGPT. This training process is only accessible to large ML teams, often with PhDs in AI. 

Technical leaders at Fortune 500 companies have told us:

*   “Our team of 10 machine learning engineers hit the OpenAI fine-tuning API, but our model got worse — help!” 
*   “I don’t know how to make the best use of my data — I’ve exhausted all the prompt magic we can summon from tutorials online.”

That’s why we’re building [Lamini](https://lamini.ai/): to give **every developer the superpowers that took the world from GPT-3 to ChatGPT.**

#### Rapidly train LLMs to be as good as ChatGPT from any base model 🚀

Lamini is an **LLM engine** that allows any developer, not just machine learning experts, to train high-performing LLMs, as good as ChatGPT, on large datasets with just a few lines of code from the [Lamini library](https://lamini-ai.github.io/) (check out an [example here](https://lamini-ai.github.io/example/)!).  
‍  
The optimizations in this library reach far beyond what’s available to developers now, from more challenging optimizations like RLHF to simpler ones like reducing hallucinations.

![](https://global-uploads.webflow.com/63ebd7a58848e8a8f651aad0/644c1da1ff728aa0a3402f5a_pull%20figure.png)

Lamini makes it easy to run multiple base model comparisons in just a single line of code, from OpenAI’s models to open-source ones on HuggingFace.  
‍  
Now that you know a bit about where we’re going: today, we’re excited to release our first major community resource!

### Available now: a hosted data generator for LLM training 🎉

We are excited to release several important steps to training your own LLM:

*   The [Lamini library](https://lamini-ai.github.io/) for optimized prompt-tuning and typed outputs ([try our playground](https://app.lamini.ai/) to see it now).
*   The advanced Lamini library for fine-tuning and RLHF, in just a few lines of code ([sign up for early access](https://lamini.ai/contact)).
*   The [first ever hosted data generator](https://github.com/lamini-ai/lamini/) for creating data needed to train instruction-following LLMs, licensed for commercial use (all yours, you own it!).
*   Open-source instruction-following LLM, using the above tools with only a few lines of code ([play with it](https://huggingface.co/spaces/lamini/instruct-playground)).

#### Steps to a ChatGPT-like LLM for your use case 1️⃣2️⃣3️⃣

Base models have a good understanding of English for consumer use cases. But when you need them to learn your vertical-specific language and guidelines, prompt-tuning is often not enough and you will need to build your own LLM.

Here are the steps to get an LLM that follows instructions to handle your use case like ChatGPT:

![](https://global-uploads.webflow.com/63ebd7a58848e8a8f651aad0/64485f29c398644819c2ff50_meta%20steps%20diagram%20(1).png)

0.  **Try prompt-tuning ChatGPT or another model**. You can use [Lamini library](https://lamini-ai.github.io/)’s APIs to quickly prompt-tune across different models, swapping between OpenAI and open-source models in just one line of code. We optimize the right prompt for you, so you can take advantage of different models without worrying about how to format the prompt for each model.
1.  **Build a large dataset of input-output pairs.** These will show your model how it should respond to its inputs, whether that's following instructions given in English, or responding in JSON. Today, we’re [releasing a repo](https://github.com/lamini-ai/lamini/) with just a few lines of code using the Lamini library to generate 50k data points from as few as 100 data points, using the Lamini library to hit the Lamini engine, so you don’t have to spin up any GPUs. We include an open-source 50k dataset in the repo. (More details below on how you can do this!)
2.  **Finetune a base model on your large dataset**. Alongside the data generator, we’re also releasing an LLM that is fine-tuned on the generated data using Lamini. We’ll soon be releasing the ability to do this programmatically ([early access](https://lamini.ai/contact)). You can also hit OpenAI’s fine-tuning API as a great starting point.
3.  **Run RLHF on your fine-tuned model**. With Lamini, you no longer need a large ML and human labeling team to run RLHF.
4.  **Deploy to your cloud.** Simply hit the API endpoint in your product or feature.

Lamini delivers the ease of prompt-tuning, with the performance of RLHF and fine-tuning. It will soon handle this entire process ([sign up](https://lamini.ai/contact) for early access!).

#### Deeper dive into step #1: a ChatGPT-like data generator

For your application, you might want similar "instruction-following" data, but you could also want something completely different, like responding only in JSON.  
‍  
‍ChatGPT took the world by storm because it could follow instructions from the user, while the base model that it was trained from (GPT-3) couldn’t do that consistently. For example, if you asked the base model a question, it might generate another question instead of answering it.  
‍

You'll need a dataset of ~50k instruction-following examples to start. Don't panic. You can now use [Lamini’s hosted data generator](https://github.com/lamini-ai/lamini) to turn just 100 examples into over 50k in just a few lines of code. 

You don’t need to spin up any GPUs, because Lamini hosts it for you. All the data that is used is commercial-use-friendly, meaning you own all the data that comes out of it.

  
You can customize the initial 100+ instructions so that the LLM follows instructions in your own vertical. Once you have those, submit them to the Lamini data generator, and voilà: you get a large instruction-following dataset on your use case as a result!

#### How the data generator works

![](https://global-uploads.webflow.com/63ebd7a58848e8a8f651aad0/644a0f9c37053f7e9a9ba899_final%20-%20Steps%20diagram.png)

The Lamini data generator is a pipeline of LLMs that takes your original small set of 100+ instructions, paired with the expected responses, to generate 50k+ new pairs, inspired by [Stanford Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html).

This generation pipeline uses the Lamini library to define and call LLMs to generate different, yet similar, pairs of instructions and responses. Trained on this data, your LLM will improve to follow these instructions.   
‍  
We provide a good default for the generation pipeline that uses open-source LLMs, which we call Lamini Open and Lamini Instruct. With new LLMs being released each day, we update the defaults to the best-performing models.

As of this release, we are using EleutherAI’s Pythia for Lamini Open and Databricks’ Dolly for Lamini Instruct. Lamini Open generates more instructions, and Lamini Instruct generates paired responses to those instructions.  
‍

The final generated dataset is available for your free commercial use (CC-BY license). 

‍  
The Lamini library allows you to swap our defaults for other open-source or OpenAI models in just one line of code. **Note that while we find OpenAI models to perform better on average, their** [**license**](https://openai.com/policies/terms-of-use) **restricts commercial use of generated data for training models similar to ChatGPT.   
‍**If you’re interested in more details on how our data generator works, read more or run it [here](https://github.com/lamini-ai/lamini).

### Releasing step #2: an open-source LLM, fine-tuned on generated data from step #1 using Lamini

Some of the generated data is good, some not. Before fine-tuning, the next step is to filter the generated data to mostly high-quality data ([just run this simple script](https://github.com/lamini-ai/lamini/blob/main/remove_duplicates.py) in the same repo). Lamini then creates a custom LLM by training a base model on this filtered, generated dataset.  
‍

We have released an [open-source instruction-following LLM](https://huggingface.co/lamini/instruct-tuned-2.8b) (CC-BY license) using Lamini to train the Pythia base model with 37k generated instructions, filtered from 70k. Play with [this custom LLM in the playground](https://huggingface.co/spaces/lamini/instruct-playground) now.

#### Pushing the boundaries of fast & usable generative AI

We’re excited to dramatically improve the performance of training LLMs and make it easy for engineering teams to train them. These two frontiers are intertwined: with faster, more effective iteration cycles, more people will be able to build these models, beyond just fiddling with prompts. We exist to help any company unlock the power of generative AI by making it easy to put their own data to work.  
‍  

**Team++:** We are growing our team with people who are passionate about making it possible to build LLMs 10x faster and making them widely accessible to empower new, extraordinary use cases. If that’s you, please send your resume and a note to [\[email protected\]](https://lamini.ai/cdn-cgi/l/email-protection#9ffcfeedfafaedecdff3fef2f6f1f6b1fef6a0eceafdf5fafceba2deefeff3e6f6f1f8baadaff9f0edbaadaffef1baadaffaf1f8f6f1fafaedf6f1f8baadafedf0f3fabaadaffeebbaadafd3fef2f6f1f6be). 🤝