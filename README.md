# How Chat Assistants Generate Text: A Reality Show Analogy

When you ask a question to your chat assistant,  
it produces an answer one word at a time.  
You may have noticed the words appearing one after the other.

**How do such systems work?**

They expect a text as input,  
sometimes called *the prompt*.

Let's say there are four words initially.  
The system _processes_ these words to generate a fifth word.  
The fifth word is then added to the input.  
The new text of five words is _processed_ to produce the sixth, and so on,  
until the system decides to stop.

---
---

## My Objective

I would like to **share an intuition**, using analogies,  
about **how the text is _processed_ to choose the next word**.

Here are some of my constraints and decisions:

- I have renounced using formulae and code.
- I try to keep things simple and use analogies to explain concepts such as 
  - Token embeddings, transformers, attention, causal masking, KV-cache, and MoE.
  - Don't worry if these words mean nothing yet - the analogies will take care of that!
- I could have asked my favourite chat assistant:
    - _"Explain to a 10-year-old child how large language models generate the next words"_.
    - But I did not.
    - Instead, I would like to present an analogy I developed recently, while reading about the topic.
    - Disclaimer: some analogies using the word `"apple"` are inspired by [videos](https://www.youtube.com/watch?v=OxCpWwDCDFQ) of Luis Serrano.
- I also could have generated some figures to illustrate the text.
    - But I did not.
    - I would prefer the reader or listener to imagine the scenes themselves, and create their own representations.

As with all analogies, it is not perfect.  
But hopefully it will help you gain an understanding,  
and avoid misconceptions about these abstract and increasingly complex systems.

At the end of this page, I provide [a technical mapping](#technical-mapping),  
showing how my analogies map to actual technical concepts.

---
---

## The Dictionary Book

The task of the system is simple in words:  
**Decide the most relevant word to continue a sentence.**

> _What are the possible words to choose from?_

Let's assume an English dictionary book contains 99,999 words.  
For instance:

- The word `"apple"` may be the 100th word referenced in the dictionary.
- While the word `"zoo"` may be the 99,900th entry.

Let's also allow for one "magic word" - more on that at the end.

In total, the **selection can be done among 100,000 possible words**.

---

## Guessing What Comes Next

Let's consider the following sequence of words:

> **`"I eat an apple and a"`**

It already contains 6 words.  
The system must decide which of the 100,000 words comes next.

One option is the following:  
The system should assign each of the 100,000 candidates a probability.

For instance:

- The word `"banana"` could be 10% likely to follow.
- The word `"zoo"` 0%, because it makes no sense here.

And so on: **100,000 probabilities are computed to decide which word comes next**.  
Finally, the word with the highest probability is selected.

> _How do we create these 100,000 probabilities from the 6 initial words?_

Let's answer with a long story, borrowing the analogy of a TV reality show.

---

## The Reality Show

The show begun on January 1st.  
There was one single participant, representing the first word in the sequence: `"I"`.

- One month later, a 2nd participant joined the show, representing `"eat"`.
- One month later, a 3rd participant joined the show, representing `"an"`.
- One month later, a 4th participant joined the show, representing `"apple"`.
- One month later, a 5th participant joined the show, representing `"and"`.

Now comes the 6th participant, after 6 months, representing `"a"`.

All together, they need to decide the next word.  
And they have exactly one month to do so.

The month start with the same protocol as in the previous months:  
**Each participant receives 4,000 numbers**.  
Let's call it their "description".

These numbers are important:  
The participants should take care of them, every day, until the end of the month.

---

## 4,000 Numbers

> _How are the 4,000 numbers decided and assigned for each participant?_

Let's remember the positions of entries in the dictionary.  
They are like identifiers:

- `"apple"` is identified by the ID #100.
- `"zoo"` by ID #99,900.

There exists a **mapping from these identifiers to the 4,000-long descriptions**.  
Imagine a table with 100,000 rows and 4,000 columns.  
The 100th row is the one for `"apple"`.  
It has 4,000 decimal numbers associated with it:  
For instance, 0.123, -4.567, and so on.

Important: No matter whether `"apple"` refers to the fruit or the iPhone manufacturer,  
it is assigned at the start of the month the same 4,000 numbers as its description.

---

## Profiling Words Like People

> _What do these 4,000 numbers mean?_

In the context of humans, one could come up  
with a list of **4,000 numerical pieces of information to describe a person**:

- The age could be the first number.
- Then the height.
- Then the weight.
- Perhaps then eye sight quality.
- The number of apples the person has already eaten in their life.
- The time the person needs to swim 100 metres.
- And so on.

> _What could be similar metrics for words?_

---

## Scoring Words

Similar to the numbers describing a human above,  
let's **introduce some characteristics**.

For instance: "fruitiness" and "technology-ness".  
**Each word can now be scored against each characteristic**.

- Words like `"blueberry"` and `"strawberry"` will have:
    - High scores for "fruitiness"
    - But low scores for "technology-ness".
- Conversely, words like `"smartphone"` and `"laptop"` will have:
    - High scores for "technology-ness".
    - But low scores for "fruitiness".

---

## On The Table

Imagine placing words on a table, following these rules:
- The more "fruity" the word, the further to the right.
- The more "techy" the word, the further to the back.

Following this rule:

- Labels like `"blueberry"` and `"strawberry"` will lie in one corner (front-right).
- Labels like `"smartphone"` and `"laptop"` will lie in the diagonally opposite corner (back-left).

One could thus say that **the words `"blueberry"` and `"smartphone"` are  
far away from one another on the table.

Note that the table plane has only 2 dimensions here, representing:
- Fruitiness,
- Technology-ness.

---

## The Apple That Won't Sit Still

> _What would be the 2 scores for the word `"apple"`?_

Put another way:

> _Where would the word `"apple"` be placed on the table?_

Close to the group of fruits?  
Or close to the one of high-tech devices?  
The answer is: in the middle.  
Because **it could be both, depending on the context**.

The label `"apple"` will move on the table towards one of the two groups   
as it becomes aware of the other words in the sentence.

In the example `"I eat an apple and a"`,   
it will be **semantically pulled towards the group of fruits**.  
More on that later.

---

## Far in Spelling, Near in Meaning

The nice property of this scoring approach is that  
**semantically close words have similar scores  
and are also close in the table**.

Other metrics such as spelling distance or dictionary distance  
do not capture semantics well.

Let's consider two words:

- `"blueberry"`, which starts with `"b"`.
- `"strawberry"`, which starts with `"s"`.

> How many pages need to be turned in the dictionary book to go from one to another?

Many!  
These words are far away in the dictionary.  
Yet they are **close semantically**.  
Which is captured by the scoring approach.

---

## More Dimensions

Other characteristics can be introduced to score words.
For instance:

- **Formality-ness**: the word `"therefore"` scores higher than `"yo"`.
- **Abstractness**: the word `"freedom"` scores higher than `"spoon"`.
- **Helpfulness**: the word `"assist"` scores higher than `"abandon"`.
- **Future-orientedness**: the word `"will"` scores higher than `"was"`.
- **Frequency**: the word `"always"` scores higher than `"rarely"`.
- **Contrast**: the word `"however"` scores higher than `"also"`.
- **Positivity**: the word `"joy"` scores higher than `"grimace"`.

And so on.  
Many such dimensions are added to score the words.  
Until reaching 4,000 in size.  
And **forming the descriptions of 4,000 numbers**.

While it is possible to easily visualise a 2D space, like a table,  
it is harder for a space of 4,000 dimensions.  
But the same principle of comparisons holds to determine  
whether words are semantically close or not.

---

## Meanwhile ...

Back to the 6 participants in the TV reality show.  
The newcomer is the 6th participant, representing `"a"`.

As already mentioned, each participant starts the month  
with a description of their word, composed of 4,000 numbers.  
These **descriptions will evolve over the days**,  
but always stay 4,000 in size.

---

## Games of Influence

> _How do their descriptions evolve over the days?_

Each day, multiple one-to-one sessions are organised.  
The first participant meets the second participant.  
Only the first one can talk about its 4,000 numbers.  
**The second listens and adjusts its own 4,000 numbers**.

In another session, roles are inverted:  
the second participant talks to the first one.

In total, there are 6 × 6 = 36 meetings.  
Yes, each participant even meets themselves!

When the participant representing `"apple"` listens to the one representing "eat",  
then they adjust their 4,000 numbers.  
The description moves in the huge 4,000-dimensional space.  
It gets closer to words representing fruits and food.  
And gets pushed away from words about tech.  
The **meaning of `"apple"` gets resolved, considering the context of the sentence**.

---

## Stubbornness

Older participants have mastered the fine art of ignoring the new arrivals.  
**They still attend the meetings  
but do not listen to participants who joined after them in the TV show.**  

That means, when the second participant talks to the first one,  
the first one ignores them and does not modify their 4,000 numbers.

At the end of the day, all participants have updated their descriptions.

- The most recent participant had to consider all discussions with others: Many things to learn about!
- The oldest participant ignored all other participants: Nothing new to learn!

By the way, **meetings are recorded**.  
More precisely, the adjustments made on the 4,000 numbers following discussions are saved.  
They will be used for the next month.  
More on that later.

---

## Alone With Their Numbers

Now comes the night.  
Each participant stays in their single room.  
There is no exchange any more between participants,  
which can sound a bit boring for a TV show.

Each participant looks at their own 4,000 numbers.  
And **based on some playbook provided by the show producer every night,  
the participant adjusts their numbers**.

The day after, 36 meetings happen again.  
Then the night.  
Each time, the descriptions of size 4,000 receive adjustments.

---

## The Backstage Experts

The producer decides to introduce a new twist to the story.  
And modify the format of the "night-time reflections".

**Instead of reflecting alone**,  
each participant **receives help from a panel of experts**, who live backstage.

There are, say, 64 experts behind the scenes.

Each expert has:

- A **particular speciality** (humour, formal language, cooking, programming, legal tone, storytelling, etc.),
- Their own secret notebook of transformation rules. To help transform the 4,000 numbers of the participant.

But consulting all 64 experts every night would be too slow, expensive and inefficient.

---

## The Audience Vote

> _How to choose which experts to consult?_

At the beginning of each night, each participant briefly introduces themselves:

> "Given what happened today, here's who I am right now..."

Based on this short self-introduction, the **show's viewers vote**:

- **Which 2 or 4 experts are most relevant to this participant tonight**.
- How much each expert's opinion should matter.

---

## Expert Adjustments

For example:

The token representing `"apple"` in a food context may primarily get:
- The culinary expert,
- The nutrition expert.

The token representing `"apple"` in a tech context may instead get:
- The high-tech expert,
- The consumer-electronics expert.

**Each selected expert gives their own adjustment**  
to the participant's 4,000 numbers.

---

## Daytime Drama, Night-time Consulting

During the day meetings, things work exactly as before.  
**Experts only affect the night-time reflections**.

---

## The Show's Little Moral

Arrives now the end of the month.  
It is time to decide which participant should join the game.  
I.e. which word among the 100,000 available words  
should complete the sentence `"I eat an apple and a"`.

> _Is there a special meeting organised between all participants for that decision?_

No!  
Instead, **only the lastly arrived participant is required.  
More precisely, its 4,000 numbers.**  
The 5 other participants also have their own 4,000 numbers,  
but these are not used for the final decision.

Funny how the newcomer, ignored all month,  
suddenly becomes the star of the finale!  

---

## The Suspense Reaches Its Peak

The show producer provides a converter during prime time.  
It is a machine that takes 4,000 numbers as input.  
And outputs exactly 100,000 probabilities.  
The circle is complete!

Remember, these 100,000 probabilities say, for each word in the dictionary,  
how likely it is to be selected as the next word.

The lastly arrived participant provides its 4,000 numbers.  
They are passed through the machine.  
The audience is eager to see the results.  
A short advertisement break happens.  
Then the results are revealed.  
From the returned 100,000 probabilities,  
**the word with the highest probability is selected**.

This becomes the 7th word in the sequence: `"banana"`.  
The sentence becomes

> **`"I eat an apple and a banana"`**

---

## A New Participant Joins

A participant representing the word `"banana"` joins the others.

As always at the beginning of a month,  
each participant receives a description of 4,000 numbers.

Participants that were already present in the last month  
receive **exactly the same description as before**.  
But their memories are reset.  
Yes, it sounds a bit inefficient.

As in the previous month:

- Meetings are held during the day
- And isolated-reflection during the night.

The first participant should talk to the second.  
Yet, this unidirectional discussion has already happened in the past.  
The producer therefore cancels the meeting.  
Instead, **the recording from this meeting is provided**.  
The 4,000 numbers of the second participant are adjusted according to the recording.

Indeed, **many of these meetings are skipped**.  
Because they have already happened in the past months.  
**Only meetings with the newcomer are run.**  
This spares a lot of meetings, which are costly!

After one month, the 8th word is decided.
The adventure continues!

---

## The Athletic Track Metaphor

It's time to get some fresh air and exercise.

One could imagine the now 8 participants placed at the starting line of an athletic track.  
Ready to complete the 100 metres on a straight line, each in their lane.

Each participant are given 4,000 numbers.

They all progress at the same pace.  
**Every 10 metres, they make a break to politely chat about their 4,000 numbers**.  
They exchange information and update their numbers.  
The older participants ignore the newcomers, as before.  
Then they resume running, this time **focusing on themselves, and calling external experts, to further adjust their numbers**.

The numbers are carried until the finish line.  
At the finish line, **the descriptions of the first seven participants are discarded**.  
The 4,000 numbers forming the description of the 8th and last participant are used to determine  
which word should be selected as the 9th word in the sentence.

---

## The End of the Show

> _When does the show stop?_

When the **magic word** is selected among the 100,000 possibilities.

The participants can at last relax,  
knowing their month-long deliberations have not been in vain.

Yet they are exhausted,  
murmuring to one another that perhaps words are more demanding  
than any reality show they have ever known.

---

## Conclusion

I hope this story helps you gain a feeling  
about how text is generated by chat assistants  
and other systems powered by similar technology.

In particular, how **words are converted to numbers**,  
because it is easier for computers to process.  
And how **computationally intensive** it is to produce one next word.

---

## Technical Mapping

> _Which part of the LLM does each analogy piece represent?_

| Analogy element                                                                                  | LLM / Transformer equivalent                                                                                                                                                          |
|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Participant (representing a word)                                                                | Token                                                                                                                                                                                 |
| Description (4,000 numbers)                                                                      | Token embedding                                                                                                                                                                       |
| 4,000                                                                                            | Embedding dimension (hidden size) (e.g. 4,096 for LLaMA models)                                                                                                                       |
| Month                                                                                            | Transformer block (layer) (e.g. 32 blocks in small models, up to 120 in large models)                                                                                                 |
| One word generated per month                                                                     | One token generated per forward path using a constant amount of compute each time _(ok, a month can be between 28 and 31 days, the analogy is therefore not perfect here)_            |
| Day meetings                                                                                     | Attention computation                                                                                                                                                                 |
| Meeting adjustments                                                                              | Updates to embeddings via attention mechanism                                                                                                                                         |
| `6 * 6 = 36` meetings                                                                            | Attention score computation by multiplying `Q` and `K` matrices (quadratic in the number of input tokens)                                                                             |
| The participant that listens                                                                     | The token that produces a Query vector (`Q`)                                                                                                                                          |
| The participant that talks                                                                       | The token that provides its Key (`K`) and Value (`V`)                                                                                                                                 |
| One talks and the other listens to make updates                                                  | Attention mechanism: The listener uses (`Q·K`) to decide how much it should pay attention to the speaker, and uses `V` to update its internal representation                          |
| Older participants ignoring newcomers                                                            | Causal masking (cannot attend future tokens)                                                                                                                                          |
| Night reflection                                                                                 | Dense / feedforward layers (MLP)                                                                                                                                                      |
| Producer's playbook for night reflection, identical for all participants and changing each night | Two MLP matrices of the dense layer                                                                                                                                                   |
| The isolation during the night                                                                   | MLP operation is per token. There is no cross-token mixing                                                                                                                            |
| Backstage panel of experts helping participants at night                                         | Mixture of Experts (MoE) MLP layer                                                                                                                                                    |
| Show viewers voting which experts to consult                                                     | Gating network selecting top-k experts                                                                                                                                                |
| 64 experts adjusting the 4,000 numbers in their own style                                        | Expert MLPs producing token-specific transformations                                                                                                                                  |
| Meeting recordings                                                                               | Key-Value (KV) cache                                                                                                                                                                  |
| Producer's machine converting 4,000 numbers at the end of the month                              | Multiplication by output vocabulary matrix                                                                                                                                            |
| Selected word                                                                                    | Next token                                                                                                                                                                            |
| Magic word                                                                                       | `<eos>` token (end of sequence)                                                                                                                                                       |
| 100,000 available words                                                                          | Vocabulary of the tokenizer (`openai 4o`'s tokenizer has a vocabulary size of ~ 200,000)                                                                                              |
| 100,000-row table                                                                                | Input Vocabulary embedding matrix                                                                                                                                                     |
| "Fruitiness", "Technology-ness", …                                                               | Abstract embedding dimensions. Not hard-coded. But shaped during training: Words of the dictionary are spread in the embedding space to cluster similar ones and repel different ones |
| Completion of the `"I eat an apple and a"`                                                       | Next token prediction task. Same task used in chat assistant to answer long and complex questions                                                                                     |

**Missing in the analogy:**

- The details of attention: `Q`, `K`, `V`.
- The multi-head attention.
- The residual connections and normalisation inside transformer blocks.
- The positional encoding.
- The training phase.
- The chat structure with special tokens.
