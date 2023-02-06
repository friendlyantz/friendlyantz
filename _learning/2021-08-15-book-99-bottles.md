---
title: "Book: '99 Bottles' by Sandi Metz"
excerpt: "first book every Ruby dev should read"
collection: learning
---

Repo: [https://github.com/friendlyantz/99bottles](https://github.com/friendlyantz/99bottles)

My takeaways:

- 1. When you were new to programming you wrote simple code. Although you may not have appreciated it at the time, this was a great strength. Since then, you’ve learned new skills, tackled harder problems, and produced increasingly complex solutions. Experience has taught you that most code will someday change, and you’ve begun to craft it in anticipation of that day. Complexity seems both natural and inevitable.

This is the basic promise of Object-Oriented Design (OOD): that if you’re willing to accept increases in the complexity of your code along some dimensions, you’ll be rewarded with decreases in complexity along others. OOD doesn’t claim to be free; it merely asserts that its benefits outweigh its costs.

The price you pay for DRYing out code is that the invoker of the new method no longer knows the result, only the message it should send.

---

Consistency - The style of the conditionals to be consistent. 
Duplication - The code should not duplicate both data and logic. Data duplications are easy to see, while logic is not obvious.
Names -  methods are defined, and messages are sent.

---

1. How difficult was it to write?
2. How hard is it to understand?
3. How expensive will it be to change?

---

Software Engineering is an art in some sense, and you can describe it as easily
as an art object or wine:
“ I like my code to be elegant and efficient. — Bjarne Stroustrup
inventor of C++

“ Clean code is full of crisp abstractions
— Grady Booch
author of Object Oriented Analysis and Design with Applications

“ Clean code was written by someone who cares.
— Michael Feathers

---

Since form follows function, good code can also be defined simply, and somewhat circularly, as that which provides the highest value for the lowest cost.

---

metrics are a crowd-sourced opinions about the quality of code.

---
Chapter 2

The purpose of some of your tests might very well be to prove that they represent bad ideas

---

The distinction between intention and implementation [ ] allows you to understand a computation first in essence and later, if necessary, in detail.
— Kent Beck Implementation Patterns (p. 69)

---

Issue with tests:

Many applications have tests that are difficult to understand, challenging to change, and prohibitively time-consuming to run. Instead of enabling change, these tests actively impede it.


A great deal of this pain originates with tests that are tied too closely to code. When this is true, every improvement to the code breaks the tests, forcing them to change in turn. Therefore, the first step in learning the art of testing is to understand how to write tests that confirm what your code does without any knowledge of how your code does it.

---

Testing, done well, speeds development and lowers costs. Unfortunately it’s also true that flawed tests slow you down and cost you money.

---

Chapterr 3

If the problem is solved, and you choose to refactor now rather than later, you pay the opportunity cost of not being able to work on other problems. Spending time "improving" code based purely on aesthetics may not be the best use of your precious time.

A good way to know that you’re using limited time wisely is to be driven by changes in requirements. The arrival of a new requirement tells you two
things, one very specific, the other more general.

Specifically, a new requirement tells you exactly how the code should change. Waiting for this requirement avoids the need to speculate about the future. The requirement reveals exactly how you should have initially arranged the code.

---

Conditionals are the bane of OO.

---

Flocking Rules
1. Selectthethingsthataremostalike.
2. Findthesmallestdifferencebetweenthem.
3. Makethesimplestchangethatwillremovethatdifference.

Changes to code can be subdivided into four distinct steps:
1. parsethenewcode
2. parseandexecuteit
3. parse,executeanduseitsresult 4. deleteunusedcode

Flocking - The group’s behavior is the result of a continuous series of small decisions being made by each participating individual:
1. Alignment-Steer towards the average heading of neighbors
2. Separation-Don’t get too close to a neighbor
3. Cohesion-Steer towards the average position of the flock
Thus, complex behavior emerges from the repeated application of simple rules.

[Steven Strogatz’s Ted Talk](https://www.youtube.com/watch?v=IiXaZGZqpVI&t=196s)
---

DRYing out sameness has some value, but DRYing out difference has more.

---

Erich Gamma, Richard Helm, Ralph Johnson and John Vlissides are commonly referred to as the "Gang of Four," in reference to their joint authorship of Design Patterns: Elements of Reusable Object-Oriented Software, describe twenty-three patterns or solutions to common OO programming problems:
_The focus here is encapsulating the concept that varies, a theme of many design patterns._

---

It is common to find that hard problems are hard only because the easy ones have not yet been solved.

---

the name of a thing should be one level of abstraction higher than the thing itself.

> First, the new requirement. Recall that the impetus for this refactoring was the need to say "six-pack" instead of "bottle/bottles" when there are 6 bottles. The string "six-pack" is one more concrete example of the underlying abstraction. This suggests that if you name the method "bottle," you will regret this decision in short order.
1
bottle
6
six-pack
n
bottles

---

Making a slew of simultaneous changes is not refactoring—it’s rehacktoring.

---

## Chapter 4

The refactoring rules say to start by choosing the cases that are most alike.

---

The fact that the argument is known to equal 1 does not matter. This
substitution is important, not because it changes the resulting value, but because it increases the level of abstraction.

### 4.5
#### Sections 4.5+ -> Liskov Substitution Principle: "Subtypes must be substitutable for their supertypes."
- 4.5 'These methods are incredibly consistent, and this did not happen by accident--they’re a direct result of the refactoring rules' 
- 4.5 Code is read many more times than it is written
- 4.6 '`No more` conundrum. The lowercase variant of "no more" is required by verse 1, and now verse 0 needs the same two words, except capitalized as the start of a sentence. The underlying concept is the same in both cases ("no more" is to be sung when the number of bottles is 0)'
- 4.6 img4.33: The above change follows the strategy of gradually making things more alike in hopes that it will then become clear how to make them identical
- 4.6 FIXNUM Error: 'You may be itching to fix this error by making a change in the quantity method, but it’s instructive to try attacking it here in verse .'
- 4.6 Inconsistency of return types by `quantity` method forces the sender of the message to know more than it should.
- 4.6 'all Ruby objects understand `to_s` , so it was programmatically convenient to blithely convert every return into a string'
- 4.7 'You may have noticed that the method you create during this refactoring contains code that exactly mirrors the shape of the original case statement. Once this becomes apparent, it makes sense to begin extracting the method in a single step,'
- 4.7 'The previous chapter showed an example where the entire container method was created at once. That was held up as an example of what not to do. The action method above looks a lot like that original `container` method, and it may seem as if you are now being given permission to act in a way that was previously prohibited.
- 4.8 'It may help to consider these questions. When the value of number is 5 , what does quantity return? How about when number is 95 ? And finally, what would quantity return if you passed in 99 ?
If you just realized that you can make these lines a little bit more alike by passing the 99 into quantity , you’ve got it.'
```ruby
when 0
  "#{quantity(99)} bottles of beer on the wall.\n"
else
  "#{quantity(number-1)} #{container(number-1)} of beer on the wall.\n"
```
- 4.10 You don’t have to understand the entire problem in order to find and express the correct abstractions—you merely apply these rules, repeatedly, and abstractions will naturally appear.

---

## Chapter 5

Human agreement about the necessity and rightness of change is reflected in the choice of the word variable for use within computer programming languages. What purpose a variable other than to vary? Most object-oriented programmers write code that both expects and relies upon object mutation. Objects are constructed, used, mutated, and then used again.

Regardless of how intuitive and natural it may seem, mutation is not an absolute requirement. It is perfectly possible (as programmers of functional languages will happily inform you) to construct applications from immutable objects, i.e. objects that do not change. For those unused to this idea, it can be disorienting to imagine reality as constructed by the functional programmer. Instead of refilling your existing cup, you discard it in favor of a new one that looks identical but is full of coffee. Rather than changing yourself to be more fit, you swap yourself for the new, fitter, you. As the Himalayas rise, you replace your existing copy with a brand new mountain range that’s a tiny bit taller.

One of the best things about immutable objects is that they are easy to understand and reason about. These objects never start out one way and then secretly morph into something else. You can be confident that what you see at creation time is always what you get later.

5.1.1. Identifying Patterns in Code: 'One way to get better at identifying smells is to practice describing the characteristics of code.'

1. Do any methods have the same shape?
2. Do any methods take an argument of the same name?
3. Do arguments of the same name always mean the same thing?
4. If you were to add the private keyword to this class, where would it go?
5. If you were going to break this class into two pieces, where’s the dividing line?
6. Do the tests in the conditionals have anything in common?
7. How many branches do the conditionals have?
8. Do the methods contain any code other than the conditional?
9. Does each method depend more on the argument that got passed,or on the class as a whole?

5.1.2. Squint Test:Changes in indentation reveal the presence of conditionals, resulting in multiple execution paths through the code, which add complexity and make code hard to understand.
Changes in color indicate differences in the level of abstraction. A method that intermixes many colors tells a story that will be difficult to follow.

Having multiple methods that take the same argument is a code smell. It’s important, however, to recognize that here the term "same" means same concept, not identical name. 

5.1.3. Programmers tend to blithely interchange these different comparison operators, confident that if the tests pass, the code is correct. However, having tests that pass doesn’t guarantee the best expression of code, and this is a case where your choice of operator affects future costs.
Testing for equality has several benefits over the alternatives. Most obviously, it narrows the range of things that meet the condition. In the above examples, if unexpected values of number arrive, the else branch executes. Knowing that the only way to get to the true branch is by
supplying an exact value of number makes it easier for future readers to
understand the code. This reduces the difficulty of debugging errors caused by incorrect inputs. Testing for equality also makes the code more precise, and this precision, as you will soon see, enables future refactorings.

5.1.4. ' someone can go to the trouble of injecting a dependency ( number ), but that dependency is too impaired to supply the
needed behavior.'
> full-blown OO mindset is deeply suspicious of conditionals! 
> there’s a big difference between a conditional that selects the correct object and one that supplies behavior. 

5.2. Extracting Classes
Built-in data classes like String , Fixnum , Integer ( Fixnum 's superclass), Array , and Hash are examples of "primitives." Primitive Obsession is when you use one of
these data classes to represent a concept in your domain. Obsessing on a primitive results in code that passes built-in types around, and supplies behavior for them.
The cure for Primitive Obsession is to create a new class to use in place of the primitive. For this operation, the refactoring recipe is Extract Class.

5.2.1 Unlike bottles, numbers aren’t things—they’re ideas, albeit ones so ubiquitous that you’ve likely forgotten how abstract and unlikely they are. Numbers are symbols used to describe quantities of things. They don’t physically exist. You can pick up a bottle, but you cannot pick up a "six."
BottleNumber is more concrete. ContainerNumber is more abstract.
The tie-breaker here is that the "name things at one higher level of abstraction" rule applies more to methods than to classes, but this rule applies more to methods than to classes
Name methods after what they mean, classes can be named after what they are.

5.2.3 Extracting BottleNumber
1. parsethenewcode 
2. parseandexecuteit
3. parse,executeanduseitsresult
4. deleteunusedcode


5.2.4. Steps for removing an argument using one-line changes:
1. Start by changing the existing argument name to anything other than what it currently is. Using delete_me will help you remember to delete the argument when you’ve updated all of the senders. The value of the default does not matter, so it’s common to use nil
```ruby
# def container(number)
def container(delete_me=nil)
```
2. Change every sender of the message to remove the parameter
```ruby
# BottleNumber.new(number).container(number)
BottleNumber.new(number).container
```
3. Finally,deletetheargumentfromthemethoddefinition
```ruby
def container
```

> NOTE by Anton: however I would use class method, but this might contradict with OO mindset?
```ruby
class BottleNumber
  def self.container(number)
  # ...
```

5.3. Re Functional Programming and OO
"Human agreement about the necessity and rightness of change is reflected in the choice of the word variable for use within computer programming languages. What purpose a variable other than to vary? Most object-oriented programmers write code that both expects and relies upon object mutation. Objects are constructed, used, mutated, and then used again.
Regardless of how intuitive and natural it may seem, mutation is not an absolute requirement. It is perfectly possible (as programmers of functional languages will happily inform you) to construct applications from immutable objects, i.e. objects that do not change. For those unused to this idea, it can be disorienting to imagine reality as constructed by the functional programmer. Instead of refilling your existing cup, you discard it in favor of a new one that looks identical but is full of coffee. Rather than changing yourself to be more fit, you swap yourself for the new, fitter, you. As the Himalayas rise, you replace your existing copy with a brand new mountain range that’s a tiny bit taller.
If the idea of immutability is new to you, the examples in the prior paragraph may seem positively alarming."
---

5.4

Phil Karlton’s famous saying "There are only two hard things in Computer Science: cache invalidation and naming things."

. The net cost of caching can be calculated only by comparing the benefit of increases in speed to the cost of creating and maintaining the cache. If you require this speed increase, any cost is cheap. If you don’t, every cost is too much.

5.6. Recognizing Liskov Violations
"Subclasses should be substitutable for their superclasses."
- ...The type of the object has changed, but the successor method still returns the old type. This inconsistency is another violation of the generalized Liskov Substitution
- it is often better to finish horizontal refactorings before undertaking vertical tangents

---

# Chapter 6

6.1 Data Clumps

Data Clump is officially about data, and is defined as the situation in which several (i.e. three or more) data fields routinely occur together.

Having a clump of data usually means you are missing a concept. When a clump gets sent as a set of parameters, the method that receives the clump can easily become polluted with clump management logic. If more than one method takes the same clump as input, some or all of this management logic will inevitably get duplicated in several places. Not only is it a pain to maintain this duplication, but over time the logic might accidentally diverge, introducing errors and confusing everyone involved.

Full-grown Data Clumps are usually removed by extracting a class, but in this small example it makes sense to simply create a new method.

6.2. Making Sense of Conditionals

Fowler offers several curative refactoring recipes. The two main contenders are Replace Conditional with State/Strategy and Replace Conditional with Polymorphism.

The Replace Conditional with State/Strategy recipe removes conditionals by dispersing their branches into new, smaller objects, one of which is later selected and plugged back in at runtime. This recipe results in a code arrangement known as composition.
The Replace Conditional with Polymorphism recipe removes conditionals by creating one class to hold the defaults of the conditionals (the `false`
branches), and adding subclasses for each specialization (the `true`
branches of the various conditions). It then chooses one of these new objects to plug back in at runtime. This recipe solves the conditional problem using inheritance.

6.3. Replacing Conditionals with Polymorphism


In OO, polymorphism refers to the idea of having many different kinds of objects that respond to the same message. Senders of the message needn’t care with which of the possible receivers they are communicating. Polymorphism allows senders to depend on the message while remaining ignorant of the type, or class, of the receiver. Senders don’t care what receivers are; instead, they depend on what receivers do.

