---
title: My talk at Ruby Oceania in Melbourne - BigO complexity of the job search
excerpt: tip and tricks to help with interviewing or how to become a FAANG CTO in 10 easy steps
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - professional_growth
toc: true
permalink: /big-o-complexity-of-the-job-search/
---
# Action Plan

- stages of job search
- tips and tricks for each part, focus on techies side
- resources and tools

---
# About me

```ruby
class Friendlyantz
  def initialize
    @name = 'Anton'
    @title = 'Engineer, Software Stuntman, Dev`s Advocate'
    @hobbies = [ 'Kitesurfing', 'Camping', 'Art', 'Motorcycling','eSkating' ]
  end
  
  def current_location = 'Melbourne, Australia'
  def work = 'Fresho🍋'
  def click_bait = 'https://friendlyantz.me/''
```

---

# Job Search Stages
1. If made redundant
2. CV update
3. apply
4. screening with HR
5. screening with manager
6. technical challenge + code review
7. system design challenge
8. behavioural interview
9. offer / negotiations

---
# If made redundant
- go to Fair Work - book Unfair dismissal consultation on day 1
- you will get a free 1hr sessions with top pro-bono lawyers (wait is about a week+)
- you have 21 days to apply. 

---
# CV
- KISS - keep it simple
- make it easily accessible and editable 
	- -> [use GitHub pages with Jekyll, avoid DOC](https://github.com/friendlyantz/cv)
- list key deliverables

---
## Key deliverables:
- *ensured team of 6 was fully caffeinated with Antarctican coffee beans ground to 14nm particles*
- *Connected with David Heinemeier Hansson on LinkedIn*
- *Organized team bonding through company potato sack race resulting in increased team cohesion*
- *Spearheaded 5 Microsofters*

---
# Apply

---
## go as wide as you can
	get those conversations going, 
	even if these are not the jobs you want

---
## first apply for non-ideal jobs you are not afraid to miss out on 
	- just practice the interviews / build up the pace
	- apply JavaScript - soooo many jobs, 
	- TypeScript, Python - all doable languages for takehome challenge

---
## don't waste time tailoring every application
	HR's don't care, just get a meeting
	- linkedin
	- indeed

---
# Screening - HR / Manager

---
## have a beautiful story
- why you looking for change
- what are you looking in a new company
- try to avoid money discussion

---
- managers can do basic tech questions sometimes
- test your browsers / OS permissions to work with Chrome, Firefox, MS teams and have screen-sharing permissions

---
# Coding challenge
- take home
- live

---

Understand basic algorithms (contested topic)

---
## understand basic data structures
		- static VS dynamic Arrays / String 
		- HasMap
		- LinkedLists / Nodes
		- Queue / Stack

		- Tree
		- Trie
		- Graph

		- Ring buffer
		- Least Recently Used (LRU)

---
## understand basic search algos
	- Binary Search
	- Linear Search
---
## understand recursion
	- preparetion / check
	- recurse step
	- go back the stack
## understand tree traversal

---
## understand basic sorting
 ```
- bubble
- quicksot
- heap sort
- merge sort
```

---
## understand BigO complexity of TIME and SPACE
- [Algo primer](https://frontendmasters.com/courses/algorithms/) - fantastic free resource by a GigaChad `ThePrimeagen`
- [Big-O complexities for both space and time](https://www.bigocheatsheet.com/), 
- [another bigO resource](https://cooervo.github.io/Algorithms-DataStructures-BigONotation/index.html).

---
BigO basic rules:
- constants are ignored
- every loop is an n-complexity. loop = O(n), loop-withi-a-loo = 0(n^2), etc
- worst case is generally is considered

---
## do basic LeetCode challenges
- focus on hottest challenges. don't just do everything LeetCode gives you
- https://neetcode.io/ - do all EASY challenges, then some MEDIUM, and then try HARD

---
familiarise with HackerRank platform
	some companies use it to do take home test, 
	so you need to be comfy with it's quirks, but practice on LeetCode

---
use chatGPT 
	to explain basic things, don't ask  complicated questions to avoid hallucinations. 

---
Put MAJOR effort with any coding assignments
- provide a test suite, 
- review before submitting.
- have good README and ideally [Makefile](https://gist.github.com/friendlyantz/3b92838e1e0361c78b23b05abdd17086)
-  may be add basic [CI script / makefile](https://gist.github.com/friendlyantz/7b7075525fe67719202648374fcf1325)
- search GitHub for already submitted solutions for some inspiration and ideas

---

## keep up with industry trends and news
- https://rubyweekly.com/
- [The Pragmatic Engineer](https://www.pragmaticengineer.com/)

---
## know basic design patterns
- Structural
- Behavioural
- Creational

[https://refactoring.guru](https://refactoring.guru)

---
Other resources
- [Tech Interview](https://github.com/yangshun/tech-interview-handbook) Handbook
- [Coding Interview Tips](https://www.interviewcake.com/coding-interview-tips)
- [How to Prepare for Tech Interviews](https://www.reddit.com/r/cscareerquestions/comments/1jov24/heres_how_to_prepare_for_tech_interviews)

---
## have questions for them
- how is their engineering culture?
- learning days / budget
- what's their plan for 1, 2, 5 years
- what version of Ruby / RoR they use, if not latest - then why

---
# System Design

---
## Go wide / don't dive into details first
- understanding basic designs of
	- messaging app, shopping app, social app, link shortener etc
- watch mock interviews with big guns 
	- -> heaps on YouTube, i.e. [Exponent channel](https://www.youtube.com/watch?v=Z-0g_aJL5Fw)
- skim [system design primer](https://github.com/donnemartin/system-design-primer)

---
## [Design Data Intensive book](https://dataintensive.net/) -
- quick youTube overview of all chapters -> https://www.youtube.com/watch?v=W5cWSNZY3l8
-  quite rich, but gives a lot of food for thought for modern distributed system design that will put you way ahead of competition. 

---
## Familiarise yourself with [Excalidraw](https://excalidraw.com/) 
- everyone uses it for diagraming
- on the right you can load awesome icons

---
# Behavioural Interview
- how you handle conflict / disagreement (EVERY interview)
- have questions for them

---
# Offer

- Negotiate -> [RubyConf talk "A People Pleasers Guide to Salary Negotiation" - Colleen Lavin'](https://www.youtube.com/watch?v=4EoMYQ6fmss)
- try to align timing for few offers
- time to think - 3-5days. for startups very short
- be proactive with communication, firm and polite - communicate often
- review contract - HRs are not your friends

---
# Other tips

- Maintain pace and routine, have a timetable
- eat healthy and exercise
- don't give up
- don't waste time and mix your activities
- participate in meetups and bookclubs
- learn and keep the dopamine loop

---
```ruby
if you_need_help => reach out!

remember: this is a fantastic time to learn!!!
```
