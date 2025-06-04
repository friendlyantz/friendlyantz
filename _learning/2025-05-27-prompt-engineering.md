---
excerpt: Short Summary how to efficiently use LLMs
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - ml
  - ai
  - llm
---
- LLM configs
- Prompting techniques

---

Temperature, Top-K, top-P
<img src="temp topp topK.png" alt="drawing" width="500"/>

---

## Temperate (0…1)
- `0` - predictable, ‘dry’, sober
- `1` - most creative

 > Start with 0.2

---

## Top K(1…) 
max count of words with the highest probabilities. Start with 30 
```js
	- I.e. `topK = 5`
["wine" (30%), "vino" (25%), "VB" (20%), "vermouth" (15%), "vine" (10%)]
```

---

## Top P(0..1) 
The model sorts all words by probability. 
It keeps adding words until their cumulative probability exceeds P. good start with 0.95 
```js
		(i.e. 0.9) -> 

["Wine" (20%), "Vino" (20%), "Vinho" (20%), "Wein" (20%), "GrapeJuice" (10%), ...]
```


---

> Note: Extreme values of above cancel each other out

---

As a general starting point:
- **Balanced / coherent results with a touch of creativity** 
	- -> temperature of .2, top-P of .95, and top-K of 30 
- **especially creative results**,  
	- temperature of .9, top-P of .99, and top-K of 40. 
- **less creative results**, 
	- temperature of .1, top-P of .9, and top-K of 20.

---
# Prompting techniques

---
## General prompting / zero shot
- I.e. Classify movie reviews as POSITIVE, NEUTRAL or NEGATIVE.

---
## One-shot & few-shot 
> (provides a single example / multiple examples to the model.)
    
 ```json
 EXAMPLE: I want a small pizza with cheese, tomato sauce, and pepperoni.
    
- JSON Response:
{ 
	"size": "small",
	"type": "normal",
	"ingredients": [["cheese"....
}
    
```

> you should use at least three to five examples for few-shot prompting

---
## System, contextual and role prompting:

---
**System prompting** 
sets the overall context and purpose for the language model. It defines the ‘big picture’ of what the model should be doing, like translating a language, classifying a review etc.

---

**Contextual prompting** 
```
Context: You are writing for a blog about retro 80's arcade
video games.

Suggest 3 topics to write an article about with a few lines of
description of what this article should contain
```
provides specific details or background information relevant to the current conversation or task. It helps the model to understand the nuances of what’s being asked and tailor the response accordingly.

---
**Role prompting** 
assigns a specific character or identity for the language model to adopt. This helps the model generate responses that are consistent with the assigned role and its associated knowledge and behavior.

---
## Step-back prompting


imagine a task(non step back):
```
write a one paragraph storyline for a new level of a
first-person shooter video game 
that is challenging and engaging.
```

---

```
Based on popular first-person shooter action games, 
what are 5 fictional key settings that contribute to a
challenging and engaging level storyline 
in a first-person shooter video game?
```

```ruby
result from LLM with 5 themes...
```

```
Take one of the themes and write a one paragraph storyline
for a new level of a first-person shooter video game that is
challenging and engaging.
```

---
## Self-consistency

- a zero-shot chain of thought prompt will be sent to the LLM multiple times, to see if the responses differ after each submit.
- Self-consistency gives a pseudo-probability likelihood of an answer being correct, but obviously has high costs.
---
## Chain of Thought (CoT)
 ```
When I was 3 years old, my partner was 3 times my age.
Now, I am 20 years old. How old is my partner?
Let's think step by step. 
```
> -> 26years

Vs non-CoT: 
```
When I was 3 years old, my partner was 3 times my age.
Now, I am 20 years old. How old is my partner?
```
> -> 63year

---
## Tree of Thoughts (ToT) 
[https://arxiv.org/abs/2305.08291](https://arxiv.org/abs/2305.08291)
 It generalizes the concept of CoT prompting because it allows LLMs to explore multiple different reasoning paths simultaneously, rather than just following a single linear chain of thought

---
## ReAct (reason & act)
current chatGPT: reason -> lookup web -> summarize

---
REFERENCES:
[Prompt engineering paper](https://drive.google.com/file/d/1AbaBYbEa_EbPelsT40-vj64L-2IwUJHy/view) by google