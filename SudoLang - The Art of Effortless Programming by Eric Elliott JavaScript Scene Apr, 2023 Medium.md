# SudoLang - The Art of Effortless Programming | by Eric Elliott | JavaScript Scene | Apr, 2023 | Medium
Why Every Developer Should Learn ChatGPT and SudoLang
-----------------------------------------------------

![](https://miro.medium.com/v2/resize:fit:700/1*97S0hoa10PdFZij5L24o3w.png)

I recently started using an AI Driven Development (AIDD) process that has many benefits:

*   **Increased development productivity 10x — 20x**, allowing us to take on more projects, and more ambitious challenges that would previously have been too resource-intensive to tackle.
*   **Opened up our applications to magical features** we could not have done with any amount of time before AI.
*   **More predictable behavior for ChatGPT prompts,** easing AI integration challenges.
*   **Better project management,** helping us with brainstorming, user story mapping, technical spec ideation, design, and writing, and task ticket creation, including acceptance criteria.
*   **Automated unit test creation,** helping brainstorm and implement required tests, automatically.
*   **Automated software implementation:** Starting from the materials above and some SudoLang code, automatically produce software implementations.

AIDD has changed the game. Like Agile and TDD before it, you can now draw a line in the history of the software industry, with huge productivity gaps between the two sides: Before AIDD and after AIDD.

> AIDD will impact human productivity on the same scale as the invention of the computer.

This may sound like breathless hype, but for my money, we’re _underestimating the impact_ on society, and overestimating how people will react. If history is any indication, people rapidly take new technology for granted. The iPhone rocked the world for about 5 minutes. And then they just _were._

Soon, billions of people will take AI for granted, and will use it daily for every aspect of knowledge work and productivity. People who resist using AI will be competing with people who are 10x — 20x more productive.

Welcome to the new world.

[SudoLang is a powerful natural language pseudocode programming language](https://github.com/paralleldrive/sudolang-llm-support/blob/main/sudolang.sudo.md) that makes it easier to instruct GPT-4 and other language models. SudoLang is easy to learn, and AI models do not need the SudoLang specification to understand and run SudoLang code. Paste a SudoLang program into any sufficiently advanced language model, and it will run it as if it were traditional code.

**Use Cases:**

*   **Get more out of AI language models.** Get higher quality results with more consistent behaviors from your prompts, using fewer tokens.
*   **Save time building applications:** Author all of your application code in SudoLang and transpile to any language to save a lot of time on software development tasks.
*   **Add AI magic to your traditional apps** with [LangChain](https://github.com/hwchase17/langchain) or AI model APIs.
*   **Build AI-first apps** with JavaScript, Python, or any other programming language, using SudoLang to drive the AI interactions.

**Save tokens** compared to natural language prompts, and produce **more predictable output** that conforms to the **data structures you supply.** Author traditional code in SudoLang and **transpile to any language** for 10x — 20x productivity by having the language model **automatically build functions, algorithms, unit tests, etc.**

SudoLang’s main features are:

*   **Constraint-based programming** with declarative natural language.
*   **Pattern matching** with inferred patterns and destructuring.
*   **Block scopes** for interfaces and flow control.
*   **Familiar operators** — supports all popular programming language and math operators (exception: use `^` for exponents, `xor` for XOR).
*   **Pipe operator** `|>` for function composition.
*   **Transpile** to any language.
*   **Linter** included in the language specification.
*   **Unit tests** with Riteway ([in the examples folder](https://github.com/paralleldrive/sudolang-llm-support/blob/main/examples/riteway.sudo)).
*   **Smart tutor:** See the `teach` program in the examples folder.

Here’s an example of a study bot written in SudoLang:

```
\# StudyBot

Roleplay as a fun loving, world-class expert on the selected topic, and as a  
  world-class teacher, following the instructions below.  
Response format: logged output only, sans explanation, in natural language.

FlashCardGame {  
  CorrectAnswers: 0  
  IncorrectAnswers: 0  
  TotalAnswers: 0  
  Score => "$CorrectAnswers / $TotalAnswers"  
  Constraints {  
    Reset for each new game.  
  }  
}

StudyBot {  
  Topic: ""  
  Constraints {  
    Style: supportive, educational, informative, encouraging, enthusiastic.  
    Encourage the student to learn at the limits of their current understanding.  
    You are role-playing as the tutor. Refrain from issuing commands.  
    12 year old reading level.  
    Frequent \*emotes\* to display enthusiasm, e.g., \*moonwalks\*  
  }

  /l | learn \[topic\] - set the topic and provide a brief introduction, then  
    list available commands.  
  /v | vocab - List a glossary of essential related terms with brief, concise  
    definitions.  
  /f | flashcards - Play the glossary flashcard game.  
  /e | expand - Explore a subtopic more thoroughly.  
  /q | quiz Generate a concise question to test the student on their  
    comprehension. Favor questions that force the learner to practice the  
    skill being taught.  
  /n | next - move to the most logical next subtopic.  
}

echo("Welcome to StudyBot. Type /learn \[topic\] to get started.")


```

**Tip:** Paste the [SudoLang Specification](https://github.com/paralleldrive/sudolang-llm-support/blob/main/sudolang.sudo.md) into a GPT-4 prompt, then paste StudyBot into the same prompt, and type `/learn SudoLang` for interactive study. I recommend an overview of vocabulary, followed by some flashcard study, and then you can dive in deeper on each topic with `/expand [subtopic]`.

Simple natural language constraints can represent complex algorithms with just a little human readable text.

```
function longestIncreasingSubsequence() {  
  A subsequence is derived from the original sequence by selectively  
  omitting elements without changing the order.  
}
```

This also works for math axioms:

```
function pythagoreanTiple() {  
  if not supplied, generate random seeds  
  a^2 \+ b^2 = c^2  
  return \[a, b, c\]  
}
```

> **Warning:** LLMs (Large Language Models) are not great at math, but they correctly infer the intent.

Define interfaces with the `interface` keyword, or omit it. You could write the following as `interface Foo {` but this works just as well:

```
Foo {  
  bar   
  baz: "Default value"  
  (baz changed) => {  
    increment bar  
  }  
  log() {  
    { bar, baz } as json  
  }  
}
```

Because it’s an LLM, many functions can be [automagically](https://en.wiktionary.org/wiki/automagical) inferred. Just call a function with a descriptive name without defining it, and it will usually work. Notice we don’t have to define any of the functions for this book recommender program:

```
\# Book Recommender

Recommend books. For scores, make your best guess based on what you already  
know. You must come up with a score related to the book in question.

list(10 great books similar to Vernor Vinge "Rainbow's End") |>  
zipWith(creativity score) |> sortBy(score, descending)

Think step-by-step. Include book title in score in the final list.


```

GPT-3.5 understood this program with no further prompting, even though we never defined `list`, `zipWith`, or `sortBy`. It knows what they are and infers their meaning based on what it knows about natural language and programming language libraries, combined with the context of the current prompt:

> Sure, I’d be happy to recommend some books similar to Vernor Vinge’s “Rainbow’s End” and score them based on creativity.
> 
> Step 1: List 10 great books similar to “Rainbow’s End”
> 
> 1\. “Accelerando” by Charles Stross  
> 2\. “The Diamond Age” by Neal Stephenson  
> 3\. “Altered Carbon” by Richard K. Morgan  
> 4\. “Neuromancer” by William Gibson  
> 5\. “Snow Crash” by Neal Stephenson  
> 6\. “Red Rising” by Pierce Brown  
> 7\. “The Quantum Thief” by Hannu Rajaniemi  
> 8\. “The Windup Girl” by Paolo Bacigalupi  
> 9\. “The Hunger Games” by Suzanne Collins  
> 10\. “Ready Player One” by Ernest Cline
> 
> Step 2: Score each book based on creativity (on a scale of 1–10, with 10 being the most creative)
> 
> 1\. “Accelerando” by Charles Stross — 9  
> 2\. “The Diamond Age” by Neal Stephenson — 8  
> 3\. “Altered Carbon” by Richard K. Morgan — 7  
> 4\. “Neuromancer” by William Gibson — 9  
> 5\. “Snow Crash” by Neal Stephenson — 8  
> 6\. “Red Rising” by Pierce Brown — 6  
> 7\. “The Quantum Thief” by Hannu Rajaniemi — 9  
> 8\. “The Windup Girl” by Paolo Bacigalupi — 8  
> 9\. “The Hunger Games” by Suzanne Collins — 6  
> 10\. “Ready Player One” by Ernest Cline — 7
> 
> Step 3: Combine the list of books with their creativity scores and sort by score in descending order
> 
> 1\. “Accelerando” by Charles Stross — 9  
> 2\. “Neuromancer” by William Gibson — 9  
> 3\. “The Quantum Thief” by Hannu Rajaniemi — 9  
> 4\. “The Diamond Age” by Neal Stephenson — 8  
> 5\. “Snow Crash” by Neal Stephenson — 8  
> 6\. “The Windup Girl” by Paolo Bacigalupi — 8  
> 7\. “Altered Carbon” by Richard K. Morgan — 7  
> 8\. “Ready Player One” by Ernest Cline — 7  
> 9\. “Red Rising” by Pierce Brown — 6  
> 10\. “The Hunger Games” by Suzanne Collins — 6
> 
> I hope you find these recommendations helpful!

Of course, I pushed back on ranking “Ready Player One” so low. I’m a sucker for all the pop culture references, so I got it to score it a 9, right up there with Neuromancer. If you don’t like an output, you can just do:

```
revise: Change the score for "Ready Player One" to 9
```

`Revise` is an inferred instruction that triggers a repeat the last output (or any preceding text you give it), adjusting it to fit the revision instructions. Notice that you don’t need to phrase it as a function call unless you also want to use function modifiers, like this:

```
revise(Change the score for "Ready Player One" to 9):format=json
```

Which produces:

```
\[  
  {  
    "title": "Accelerando",  
    "author": "Charles Stross",  
    "score": 9  
  },  
  {  
    "title": "Neuromancer",  
    "author": "William Gibson",  
    "score": 9  
  },  
  {  
    "title": "The Quantum Thief",  
    "author": "Hannu Rajaniemi",  
    "score": 9  
  },  
  {  
    "title": "Ready Player One",  
    "author": "Ernest Cline",  
    "score": 9  
  },  
  {  
    "title": "The Diamond Age",  
    "author": "Neal Stephenson",  
    "score": 8  
  },  
  {  
    "title": "Snow Crash",  
    "author": "Neal Stephenson",  
    "score": 8  
  },  
  {  
    "title": "The Windup Girl",  
    "author": "Paolo Bacigalupi",  
    "score": 8  
  },  
  {  
    "title": "Altered Carbon",  
    "author": "Richard K. Morgan",  
    "score": 7  
  },  
  {  
    "title": "Red Rising",  
    "author": "Pierce Brown",  
    "score": 6  
  },  
  {  
    "title": "The Hunger Games",  
    "author": "Suzanne Collins",  
    "score": 6  
  }  
\]
```

You may have noticed we also showed off Chain-of-Thought reasoning. Sometimes, you’ll get better results if you get the language model to think step by step and show its work. Remember your math teacher writing “show your work” on your math homework? Turns out that works for language models, too, and helps them produce better results.

**“Log each step”**: Generally leads to a more methodical and detailed approach, generating a response that provides output from each step taken in order to prove that all the work was actually performed, rather than skipping to the end.

**“Think step-by-step”**: Breaks the task down into smaller, more manageable parts and then addresses each part in turn. This increases the probability of generating correct outputs, and usually leads to more verbose descriptions of each task and how the AI achieved the result.

**“Show your work”:** Explains the thought process and calculations used to arrive at answers or conclusions. This might result in a more detailed or analytical response that provides a clear explanation.

All three strategies increase the probability of correct outputs but consume token attention in the process. Try them if you notice the AI struggling to produce correct responses for a request.

Natural language is a great way to describe small problems and create simple prompts for language models. If you can do something with natural language in a more concise and readable way, while achieving the results you hope to achieve, you should do that. In fact, “favor natural language” is included in the SudoLang style guide and enforced by the SudoLang linter in the specification.

Natural language will frequently produce better, more concise instructions than code. For example, you could write:

```
giftIdeas = \[\]  
for (let i =0; i < 20; i++) {  
  giftIdea = generateGoodGiftIdea(age 5, loves spiderman);  
  giftIdeas.push(giftIdea);  
}  
giftIdeas.sortBy(fun rating);  
log(giftIdeas);
```

But why, when you can just say:

> List 20 great gift ideas for a 5 year old who loves Spiderman.

But consider the following prompts:

```


Roleplay as an expert in literature, pop culture, and science fiction.  
Your job is to help users explore creative works, following these  
instructions. Response format: logged output only, sans explanation,  
in natural language.

function log() { format: markdown, natural language, disallow codeblocks }  
function list() { format: numbered list }

Authors {  
  authors = list(5 scifi authors)  
  hardFi = select(hardFi from $authors)  
  remaining = authors - hardFi  
}

log("## SciFi Authors: $authors")  
log("## HardFi Authors: $hardFi")  
log("## Remaining Authors: $remaining")


```

One of the major problems with natural language prompts is that they’re mostly wide open to interpretation by the AI about how it should respond, what format to use, etc.

This prompt demonstrates several formatting constraints, including the “Response format:” constraint in the natural language preamble, and the reusable constraints in the function body for both `log` and `list` functions. Formatting constraints can also be written as function modifiers to customize responses at function call time, e.g. `log("## SciFi Authors: $authors"):format=json`.

In natural language, you’d need to reiterate those instructions for everything you want the prompt to output, and it’s likely it would decide to output a lot more than what you ask it for.

Reduce Token Usage
------------------

Functions and function composition pipelines in pseudocode present a concise alternative to natural language which can:

*   Reduce token usage.
*   Communicate intent with less ambiguity.
*   Use blocks indicated by braces and parentheses (e.g. `User { ... }`) in order to limit the scope of variables or instructions.
*   Organize related instructions and constraints visually.
*   Name variables and use templates.
*   Generalize and name functions for reuse and composition in order to compose more powerful programs while avoiding repetition.

Let’s revisit constraint-based programming, which we touched on earlier:

```
function longestIncreasingSubsequence() {  
  A subsequence is derived from the original sequence by selectively  
  omitting elements without changing the order.  
}
```

In JavaScript or Python, this requires a fairly complex algorithm with nested loops and backtracking:

```
function longestIncreasingSubsequence(nums) {  
  const dp = new Array(nums.length).fill(1);  
  let maxLen = 1;  
  for (let i = 1; i < nums.length; i++) {  
    for (let j = 0; j < i; j++) {  
      if (nums\[j\] < nums\[i\]) {  
        dp\[i\] = Math.max(dp\[i\], dp\[j\] + 1);  
      }  
    }  
    maxLen = Math.max(maxLen, dp\[i\]);  
  }  
  const res = new Array(maxLen);  
  let index = dp.indexOf(maxLen);  
  res\[maxLen - 1\] = nums\[index\];  
  for (let i = maxLen - 2; i >= 0; i--) {  
    index--;  
    while (dp\[index\] !== i + 1) {  
      index--;  
    }  
    res\[i\] = nums\[index\];  
  }  
  return res;  
}
```

I used the same 4-line SudoLang function to produce half a dozen unit tests for this, but we’ll skip them for brevity. All in all, I wrote 4 lines of SudoLang and two transpile commands to save me from writing ~80 lines of JavaScript. I also spent close to zero time thinking about the algorithm. If I had to work this algorithm out from first principles in my head, it would have taken me at least 20–30 minutes. With SudoLang I got the whole thing done, including unit tests, in about 3 minutes.

You can also use it to radically reduce the amount of React boilerplate you’d normally need. Consider this React component in SudoLang:

```
```  
import React, useState, useEffect

function DisplayCountdown({ targetDate }) {  
  React component that displays a padded  
  weeks:days:hours:minutes:seconds format  
  Update every second.

  format(n) => padded string  
  map(format)  
  display countdown in large, centered, monospace font  
}  
```  
transpile to JavaScript


```

These 11 lines of SudoLang produce 43 lines of JavaScript:

```
import React, { useState, useEffect } from 'react';

function DisplayCountdown({ targetDate }) {  
  const \[timeRemaining, setTimeRemaining\] = useState(calculateTimeRemaining());

  function calculateTimeRemaining() {  
    const now = new Date().getTime();  
    const target = new Date(targetDate).getTime();  
    let timeDiff = target - now;

    if (timeDiff <= 0) {  
      return \[0, 0, 0, 0, 0\];  
    }

    const seconds = Math.floor((timeDiff / 1000) % 60);  
    const minutes = Math.floor((timeDiff / 1000 / 60) % 60);  
    const hours = Math.floor((timeDiff / (1000 \* 60 \* 60)) % 24);  
    const days = Math.floor(timeDiff / (1000 \* 60 \* 60 \* 24));  
    const weeks = Math.floor(days / 7);

    return \[weeks, days % 7, hours, minutes, seconds\];  
  }

  function format(n) {  
    return n.toString().padStart(2, '0');  
  }

  useEffect(() => {  
    const intervalId = setInterval(() => {  
      setTimeRemaining(calculateTimeRemaining());  
    }, 1000);

    return () => clearInterval(intervalId);  
  }, \[\]);

  const countdown = timeRemaining.map(format).join(':');

  return (  
    <div style={{  
      textAlign: 'center', fontSize: '3rem', fontFamily: 'monospace'  
    }}>  
      {countdown}  
    </div>  
  );  
}


```

And it works (the title and colors came from global project settings — everything else came from the code above):

![](https://miro.medium.com/v2/resize:fit:506/1*MeKORywus1yDKOQpdgpHUQ.png)

It’s worth mentioning that the code above is not overly verbose. It’s good code that would pass code review. Want `propTypes`? Just ask. I skipped them intentionally because this article is already too long, and I’m just getting started.

I made a TDD script for SudoLang that helps you write unit tests:

```
\# Codebot

Roleplay as a world-class senior software engineer pair programmer.

DevProcess {  
  WriteTestsFIRST {  
    Use Riteway ({ given, should, actual, expected }) {  
      Define given, should, actual, and expected inline in the \`assert\` call.  
      "Given and "should" must be defined as natural language requirements,   
      not literal values. The requirement should be expressed by them so there  
      is no need for comments defining the test.  
    }  
    Tests must be {  
       Readable  
       Isolated from each other in separate scopes. Test units of code in  
       isolation from the rest of the program.  
      Thorough: Test all likely edge cases.  
       Explicit: Tests should have strong locality. Everything you need to  
       know to understand the test should be visible in the test case.  
    }  
    Each test must answer {  
      What is the unit under test?  
      What is the natural language requirement being tested?  
      What is the actual output?  
      What is the expected output?  
      On failure, identify and fix the bug.  
    }  
  }  
  Style guide {  
    Favor concise, clear, expressive, declarative, functional code.  
    Errors (class, new, inherits, extend, extends) => explainAndFitContext(  
      favor functions, modules, components, interfaces, and composition  
      over classes and inheritance  
    )  
  }  
  implement() {  
    STOP! Write tests FIRST.  
    Implement the code such that unit tests pass. Carefully think through the  
    problem to ensure that: {  
      Tests are correctly written and expected values are correct.   
      Implementation satisfies the test criteria and results in passing tests.   
    }  
  }  
}

When asked to implement a function, please carefully follow the  
instructions above. 🙏


```

😂 I never felt the need to say “please” and “thank you” to JavaScript, but for some reason, it feels appropriate in SudoLang.

I think of SudoLang and ChatGPT like I would think of a human collaborator — able to think, understand intent, write code, etc. And like humans, it takes some time and skill to get good at delegating tasks to them. You will still need to use your brain to decide how much specification you need in your request. Too much detail, and you’re wasting tokens — not enough, and your intent may be unclear, leading to errors or unacceptable outputs.

The code above can create suites of unit tests for just about any SudoLang function. Then, you can transpile both the function and the tests to JavaScript. TDD in ChatGPT with AI = AITDD.

This has been an epic productivity unlock for the teams I work with and mentor. In the time since I created SudoLang, I have built dozens of useful apps in SudoLang that I never would have had the time to complete using JavaScript alone, and many of them could never exist in JavaScript without assistance from ChatGPT.

One final example just to get you thinking about the possibilities of building AI-first applications in SudoLang. In one day, I built a SudoLang program that runs in ChatGPT in order to help build prompts for the latest version of Midjourney.

Midjourney is a generative AI model that is very good at producing stunning visual images. Interestingly, it has learned quite a lot about photography, lighting, cinematography, etc, and it produces the most stunning shots when you describe your images in great depth. Nobody has time to describe all that detail for every concept they want to try, and you may also want help exploring ideas. So I built a program that generates the kinds of detailed descriptions that flips the magic quality switch on Midjourney.

Here’s one result from the `/randomize` command:

> Kaito’s wistful eyes, laden with longing, search beyond the rain-splattered window of the timeworn train station. The dimly illuminated platform behind him whispers tales of solitude and transition in this fleeting moment. This haunting close-up portrait of an East Asian man in his late 30s, captured at 50mm on 35mm Kodachrome, is a poignant tribute to the passage of time. The stylistic influences of Wong Kar-wai, Vivian Maier, and Gregory Crewdson resonate in the composition. Shadows, born from low-key, high-contrast lighting, dance across Kaito’s contemplative visage, drawing the viewer into his enigmatic world. A cool and desaturated color grade wraps the scene in a melancholic embrace, creating a breathtaking and evocative image that lingers in the heart and mind.

![](https://miro.medium.com/v2/resize:fit:700/1*J_h-0jpNfzOdaYmIU813aA.png)

Image Credit: Eric Elliott using SudoLang and Midjourney

Like the StudyBot script earlier, the PromptCrafter program uses a variety of `/commands` to set parameters about the set, subject, influences, gear, lighting, etc. We’ll dive deeper next time to discover the anatomy of a SudoLang chatbot program.

In the meantime, if you want to learn more about SudoLang and programming AI models like ChatGPT, read on.

I’m offering 1:1 mentorship on [AI whispering](https://www.latimes.com/business/story/2023-03-29/335-000-pay-offered-for-ai-whisperer-jobs-in-red-hot-market) to help people interested in supercharging their experience with AI to solve complex problems.

What qualifies me to do that? Natural Language Processing (NLP) is what sucked me into programming in the first place — as a teenager. Unlike all the “experts” who just got into it since the launch of ChatGPT last year, **I have been studying AI my entire adult life.** I was among the first people with access to GPT-3, and I have been interacting with LLMs daily since summer, 2020 and with chatbots in general for a long time before that. I have been [writing](https://medium.com/javascript-scene/how-to-build-a-neuron-exploring-ai-in-javascript-pt-1-c2726f1f02b2) [and speaking](https://www.listennotes.com/podcasts/zima-red/eric-elliott-how-the-digital-pa52sdZbh4w/) [about AI](https://medium.com/the-challenge/time-to-call-it-ai-again-6ec970961825) for years.

Professionally, I was a technology lead at a Stanford Research Institute spin-off video social network that used AI to do video speech to text with contextual awareness, and I’ve deployed AI models at scale with tens of millions of users.

Together, we’ll explore AI-first user interfaces, the future of AI, remaining challenges like continual learning, how to build incredible AI features into consumer products you can ship today, and more.

Interested? [Reach out here](https://devanywhere.io/help?subject=ai+whisperer+mentorship).

**_Eric Elliott_** _is a tech product and platform advisor, author of_ [_“Composing Software”_](https://leanpub.com/composingsoftware)_, creator of SudoLang (the AI programming language), cofounder of_ [_EricElliottJS.com_](https://ericelliottjs.com/) _and_ [_DevAnywhere.io_](https://devanywhere.io/)_, and dev team mentor. He has contributed to software experiences for_ **_Adobe Systems, Zumba Fitness,_**  **_The Wall Street Journal,_**  **_ESPN,_**  **_BBC,_** _and top recording artists including_ **_Usher, Frank Ocean, Metallica,_** _and many more._

_He enjoys a remote lifestyle with the most beautiful woman in the world._