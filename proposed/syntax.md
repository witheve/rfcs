Discussion: https://github.com/witheve/rfcs/issues/4

# RFC - Developer Syntax

(A note before we get started: this is our first official Request for Comment (RFC). RFCs are meant to be an informal way to start a discussion on a particular feature, design, protocol, process, or anything else relating to Eve and the Eve community. As the discussion evolves, the RFC will become a form of documentation, explaining the genesis of a particular feature. You can read more about our RFC process [here](https://github.com/witheve/rfcs)).

## Summary

Today we are asking community feedback on our current developer syntax proposal. In this RFC I'll present a complete program written in the syntax and explain how it works. We hope to specifically hear opinions on the following:

1. How easy is it to read and understand code you haven't written in the proposed syntax?
2. How easy is it to write code in the proposed syntax?
3. Is the programming model clear?
4. Does the syntax support working and thinking in this model?
5. What changes would make writing or reading easier?
6. Did you experience any "ah ha!" moments while reading about the syntax? i.e. Was there something we said or an example we gave that made the whole thing click?

## Motivation

In our [Jan/Feb dev diary](http://incidentalcomplexity.com/2016/06/10/jan-feb/) we talked a little about the need for a syntax during our development, even if ultimately most users will never see it. To summarize: while the graphical interface is under development, a textual syntax helps us test the platform, share code, and find/report/reproduce bugs.

We are specifically calling this a "developer syntax" because it is meant for software developers who know how to program. We want to make sure you understand this isn’t how we will present Eve to people who don’t know how to program.

In recent months, the semantics of the Eve language have stabilized to something we're very happy with, and quite excited about. However, the graphical model for interacting with the Eve language is still very much in flux. We've now reached a point where we want to share our work with the community, and we can’t do that without a proper interface to the language. 

Thus, to get Eve out to early adopters sooner, we have developed a textual syntax, quite different from what we've [shown so far](http://incidentalcomplexity.com/2016/06/30/apr/).

## Design

### Design Goals

Here are the broad design goals we identified when designing the syntax:

1. **For Humans** - This syntax is designed for humans, not compilers.
2. **Readable** - Since code is read more than written, we want the syntax to be eminently readable.
3. **Familiar** - Users unfamiliar with Eve should be able to read an Eve program and figure out what's going on at a high level.
4. **Different** - This one is purposefully in contention with goal (3); we want the syntax to be familiar but not *too* familiar. After all, Eve itself is very different from most languages out there, so we don't want to se the appropriate expectations.

### Programming Model

To really understand a syntax, you have to also understand the semantics of the underlying programming model, so that's where we will start. This discussion will be very high-level, so don't worry about the details for now. First, remember that Eve is not just a language; it is also a database. These are not separate components that interact; they are actually one and the same. However, sometimes we use "Eve DB" to refer to the underlying facts in the database.

At its core, Eve only responds to two commands:

1. What facts do you know about `x`?
2. Remember a new fact about `x`.

Communication with Eve happens through "objects", which are key-value pairs attached to a unique ID (object is a pretty generic and overloaded term, so let us know if you have ideas for what to call these guys). To access facts in the Eve DB, you use an object. To insert/remove facts into/from the Eve DB, you also use an object.

Computation, occurs as a result of relationships between objects. For example, I might model myself as an object with an `age` and a `birth year`. There might also be an object representing the `current year`. Then I could compute my `age` as my `birth year` subtracted from the `current year`.

A key concept here is that age is a derived fact, supported by two other facts: `birth year` and `current year`. If either of those supporting facts are removed from the Eve DB, then `age` can no longer exist as well. For intuition, think about modeling this calculation in a spreadsheet using three cells, and what would happen to the `age` cell if you deleted one of the other two cells.

Thus the presence or absence of facts can be used to control the flow of a program. To see how this works, consider how a navigation button on a webpage might work. When the button is clicked, a fact is inserted into the Eve DB noting the element that was clicked. This click fact would support an object that defines the page to be rendered. This in turn would support the page renderer, whose job it is to display the page.

One last thing to note about control flow is that we have no concept of a loop in Eve. Recursion is one way to recover looping, but set semantics and aggregates often obviate the need for recursion. In Eve, every value is actually a set. With operators defined over sets (think `map()`) and aggregation (think `reduce()`) we can actually do away with most cases where we would be tempted to use a loop.

#### Set Semantics

One other thing to know about Eve is that objects follow [set semantics](https://en.wikipedia.org/wiki/Set_(mathematics)). Sets are collections where every element of the collection is unique. This is in contrast to bag semantics, where elements can be duplicated. We’ll see the implications of this later, but it’s important to keep in mind.

### A Working Program - Party Planning

Through the rest of this document, we'll refer to the following complete Eve program. Don't worry about not understanding it; we'll go over what all the parts mean, and then hopefully the program will become clear. Let’s dive right in:

```
This program is used to plan the number of burgers I need for my birthday party.
I will be inviting all my friends and their spouses, so I need to figure out
how much food I need to buy!

Count the number of guests coming to the party
 party = [name: "my party", date]
 guest = if p = [#friend busy-dates: not(party.date)] then p
         if [#friend spouse busy-dates: not(party.date)] then spouse

 total = count(given guest)
 maintain
    party.guest-count := total
    party.guest += guest

How many burgers do I need?
 party = [@"my party" guest]
 burgers = if guest = [#hungry] then 2
           else if guest = [@arthur] then 3 // my friend arthur eats too many burgers
           else if guest = [#vegetarian] then 0
           else 1
 total = sum(burgers given burgers, guest)
 maintain
           guest.burgers += burgers
    party.burgers := total

Calculate a time difference
 [#time date: today]
 time-remaining = targetdate - today
 maintain
    [#timediff targetdate? time-remaining]

How long until my party?
 party = [@"my party" date]
 [#timediff targetdate: date, time-remaining]
 maintain
    party.timeleft = time-remaining

My party is on my birthday!
 [@Corey birthday]
 freeze
    [@"my party" date: birthday]
```

#### Program Structure

The first thing to note is the broad structure of the program. An Eve program consists of any number of “blocks”. In this program, we have five blocks, which are are delineated by indentation; unindented text is treated as a comment, while indented text is treated as code. Blocks are terminated at EOF or the next unindented line. Inline comments are possible using the `//` prefix anywhere in the program.

At the beginning of each block is a "block header" e.g. "How many burgers do I need?". These are meant primarily for documentation, and by convention they are a brief description of the purpose of the block. Note that these are not "function handles" or anything that you call in code.

### Objects

Objects are the predominant datatype in Eve. In the proposed syntax, objects are a set of attribute:value pairs enclosed in square brackets:

```
object = [ attribute1: value attribute2: value ...  attributeN: value]
```

Objects are essentially pattern matches against the Eve DB, i.e. objects ask Eve to find all the entities that match the supplied attribute shape. For example, our first object in the program is `party = [name: "my party", date]`. The resulting `party` object will consist of all the facts matching a `name` attribute with value "my party" and a date attribute with any value.

The `party` object also binds `date` to the top level `date` variable, accessible outside of the object (but only within the block). If you want to use `date` to mean something else, then you can alias it using the bind operator (see the next section). You can also access the unmatched attributes of an object using dot notation e.g. `party.date`.

### Binding, Equivalence, and Names

Our syntax has two binding operators: colon ( `:` ), and equals ( `=` ). By convention, colon is used within objects, and equals is used outside of objects. Either way, binding works the same: binding asserts that the left hand side of the operator is equivalent to the right hand side. This is distinct from assignment, which is not a concept in Eve (therefore, we have no need for an `==` operator).

Names are another way to say one thing is equivalent to another; within a block, variables with the same name represent the same object or attribute of an object. For instance:

```
People who are 50 year old
  [tag: "person" age]
  age = 50

The same query as above
 [#person age: 50]

Never true
 [#person age: 10]
 person.age = 20
```

Names are a little more permissive in our syntax than other languages. We allow most symbols in a name (with the exception of space, @, #, //, period, question, comma, colon, and grouping symbols). So operators like `-` and `+` are valid symbols in a name. This comes at the cost of requiring whitespace in expressions. For example `friend-age` is a name. By contrast `friend - age` is subtracting age from friend.

### Names and Tags

We’ve identified two attributes that are generally useful, so we’ve given them special syntax. These attributes are `name` and `tag`.

#### Name Selector ( `@` )

The name selector is used to select a specific named object from the Eve DB. Named objects are just objects with a `name` attribute specified. In the example program,`[@"my party"]` is shorthand for `[name: "my party"]`.

#### Tag Selector ( `#` )

The tag selector is used for selecting a group of similar objects i.e. objects with the same tag attribute. In the above example, we used `[#friend]`, which is shorthand for `[tag: "friend"]`.

### Block structure

Let's focus in on the first block:

```
count guests coming to the party
 // Phase 1: Collect
 party = [@"my party" date]
 guest = if p = [#friend busy-dates: not(party.date)] then p
         if [#friend spouse busy-dates: not(party.date)] then spouse
 total = count(given guest)
 // Phase 2: Mutate
 maintain
    party.guest-count := total
    party.guest += guest
```

Blocks themselves have their own structure as well. Each block is written in two phases: collect then mutate. These mirror the Eve commands outlined above. So you can see, an Eve program is just continued repetition of collect/mutate operations.

In the collect phase, we ask Eve for known facts, and we might transform those facts using temporary variables. In the mutate phase we tell Eve to remember new facts. Let's look at each of those phases here:

#### Phase 1: Collect

The collect phase is used to gather all the information you need to complete your block. In the following block, we want to count all the guests coming to the party. To do this, we need the date of the party, a list of all my friends and their availability, and then a count of the guests on the list. Below, I've annotated what's going on in the collect phase.

```
 // Select the party
 party = [@"my party" date]

         // Add my friends to the guest list if they are not busy the day of the party
 guest = if p = [#friend busy-dates: not(party.date)] then p

         // Add my friends' spouses to the list if they are not busy
         if [#friend spouse busy-dates: not(party.date)] then spouse

 // Count all the total number of guests
 total = count(given guest)
```

Let's take a closer look at what you can do in the collect phase:

##### **Aggregates**

Aggregates are functions that take an input set and produce an output set, typically with a different cardinality than the input set. For example, `count` takes an input set of cardinality `N` and produces a set of cardinality `1` as a result. A familiar analogue in other languages is the `reduce()` function. Here is an example of an aggregate in use:

```
 total = count(given guest)
```

Aggregates are called like functions in other languages, but there is a slight difference. Here, the keyword `given` specifies the set we are counting. A better example of this is the `sum` aggregate:

`total = sum(burgers given burgers, guest)`

Here, we are summing `burgers`, given the set of `burgers` and `guest`. This is important because of our set semantics. If we had just `sum(given burgers)`, then the duplicate elements of `burgers` would be removed and we would arrive at the wrong number. By adding `guest` to the set, we ensure that every guest’s burger quantity properly counted.

##### **If**

`If` allows conditional equivalence, and works a lot like `if` in other languages. Our `if` has two parts: `if` followed by a conditional; and `then` followed by a return object(s). In every case, an arm of an `if` returns only if the condition contains results. In the above example, we add guests to the list if they are a friend and not busy, or if they are a spouse of a friend and not busy. 

`If` can be used in two ways. First, you can chain a series of `if` statements, like in this example:

```
 guest = if p = [#friend busy-dates: not(party.date)] then p
             if [#friend spouse busy-dates: not(party.date)] then spouse

```

This is equivalent to a `union` operator, which combines elements from disparate sets into the same set. The second way to use `if` is in combination with the `else` keyword:

```
 burgers = if guest = [#hungry] then 2
           else if guest = [@arthur] then 3
           else if guest = [#vegetarian] then 0
           else 1
```

In this case, `if` is used as a `choose` operator, selecting only the first branch with a non-empty body. If some guest is tagged both `#hungry` and `#vegetarian`, that guest will actually receive 2 burgers. Therefore, while order of statements usually does not matter in Eve, `if` statements are one area where it does.

##### **Not**

Not is an anti-join operator, which takes a body of objects. In our example, we've specified the date of "my party", and each `#friend` has specified the dates they are busy. A `#friend` can only go to the party if the date of the party is not on her list of busy dates. e.g. `[#friend busy-dates: not(party.date)]`.

##### **Expressions**

Expressions are used to perform calculations using constants and attributes of objects e.g. `party.burgers / party.guest-count` would calculate the ratio of burgers to the number of guests. Note that we are not subtracting `count` from `party.guest`. In expressions, you are required to add whitespace between operators. This helps with readability, but it also allows us to add more characters to names.

Operators are defined over sets, so you could do something like `cheese-slices = guest.burgers * 2`, which would multiply every guest’s burger count by two. There is a pitfall here: if you perform an operation on disjoint sets with differing cardinality, the result will be a [cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) of the two sets. This is usually not a desired behavior, but sometimes it is what you want to do e.g. if you wanted to calculate the distance from every point in a set to every other point in the set, a cartesian product would be useful.

##### **String Interpolation**

We support string interpolation using double curly braces e.g. {{ }}. For instance, consider the following:

```
print the guests and burgers
  [#party-guest guest burgers]
  maintain
    [#div text: "{{guest.name}} is going to eat {{burgers}} burgers"]
```

In this block of code, we select the guests of the party and the number of burgers they are going to eat at the party. The last line of the block prints this information in a sentence. Notice that because of set semantics, we only wrote code to print a single sentence; when the code is executed, the sentence will be printed once for every guest.

#### Phase 2: Mutate

(We're also not happy with the mutate nomenclature. Any suggestions are appreciated).

In the mutate phase, we're ready to change the Eve DB in some way. The mutate phase is fenced off with either the `freeze` or `maintain` keywords (explained below). By convention, the mutate phase is indented twice, as a way to indicate the change in modality. The fence tells Eve that we're not longer interested in asking questions about objects. Thus we're no longer able to use any statements applicable to the collect phase e.g. `if`, `not`, aggregates, expressions, etc.

Let's look at what happens in the mutate phase (this one comes from the first block in our party program):

```
 // maintain tells us that we are updating the database continually
 maintain

    // We set the guest-count to be the total number of guests we found
    // in the collect phase. This will overwrite whatever the total
    // was before
    party.guest-count := total

    // We add new guests to the guest list. This will add to the
    // existing list    
    party.guest += guest
```

##### Adding and Removing Objects

Objects can be freezed to the Eve DB just by adding them after a mutation fence:

```
 [@Corey birthday]
 freeze
    [@"my party" date: birthday]
```

Objects can be removed from Eve using the `none` keyword. For example, we could remove `@"my party"` like so:

```
freeze
 [@"my party"] := none
```

##### Mutation Operators

We have three operators for mutating objects in the Eve DB: add, set, and remove:

- Add ( `+=` ) - adds attributes to an object.
- Set ( `:=` ) - sets the value of attributes on an object
- Remove ( `-=` ) - removes attributes from an object

Mutation operators can be used in two ways. First, you can add/set/remove a specific attribute on an object. E.g. `object.attribute = value`. Values can be anything, including objects. The alternative way allows you to add/set/remove multiple attributes. E.g. `object += [attribute1: value, attribute2: value, ...]`.

Mutations follow set semantics. If an attribute exists on an object, using += will just add it to the set. For instance, if `person.age = 10`, and `person.age += 20`, then `person.age = {10, 20}` (note, the curly braces are not part of the syntax, but are a standard way of indicating a set).

##### Freeze vs. Maintain

We have two fences for the mutation phase, with differing behaviors. When you use `freeze`, you're telling Eve that you want the subsequent facts to persist in the database, irrespective of the supporting data. To see what I mean by this, For example, look at the last query:

```
my party is on my birthday!
 [@Corey birthday]
 freeze
    [@"my party" date: birthday]
```

We use `freeze`, so even if `@Corey` or his birthday are deleted from the database, `@"my party"` will still remain.

By contrast, consider:

```
 guest = [#friend busy-dates: not(party.date)]
 maintain
    party.guest += guest
```

In this case, the fence is `maintain`, so we're specifying that `party.guest` will be automatically kept up-to-date. The behavior of this code is that a `guest` is a `#friend` who is not busy on the date of the party. Let’s say my friend Sam’s calendar is initially clear, and so he is originally on the guest list. Then some time later, Sam suddenly adds the party date to his list of busy dates. Now, he no longer satisfies the conditions of the block, and he is removed from the guest list (also subsequently lowering the burger count). Had we used the freeze fence, then his initial admittance to the guest list would be permanent, and we would have too many burgers for the party.

##### freeze all

By default, any mutations made to the database are per session, meaning any facts you add to the database, are only visible to the session that added them. Both `freeze` can be optionally followed by the `all` keyword, which indicates that the subsequent mutations are available globally, to any all sessions connected to Eve.

This is useful if you want to create a networked application, like chat:

```
create a message on enter
  [#user name]
  event = [#keydown element, key: "enter"]
  element = [#channel-input channel value]
  freeze all
    [#message event name, message: value, channel]
```

Here we get the username, and the message, and we create a message when enter is pressed. The `all` keyword makes this message available to all sessions on the server. If we ommitted the keyword, then no other users would actually receive the message.

### Code Reuse

Finally, Eve has no concept of a function, but code reuse can be achieved through objects. Note that we have several statements that look like function application (e.g. aggregates and not), but these are purely syntax sugar for convenience, and not user definable.

To see how code reuse works, let's go back to our party example:

```
Calculate a time difference
 [#time date: today]
 time-remaining = targetdate - today
 maintain
    [#timediff targetdate? time-remaining]
```

Here we define a reusable block by leaving some variables unbound. The `?` calls out that the preceding variable `targetdate` is unbound in the block. According to our semantics, this block isn't functional until `targetdate` is bound, which we can do in another block:

```
How long until my party?
 party = [@"my party" date]
 [#timediff targetdate: date, time-remaining]
 maintain
    party.timeleft = time-remaining
```

This completes the first block, so `party.timeleft` will be a continuously updating countdown to the date of the party.

## Other Examples

Feel free to check out some more examples of programs written in the proposed syntax [here](https://github.com/witheve/eve/tree/master/examples). In particular, the following applications are complete applications: [chat](https://github.com/witheve/eve/blob/master/examples/chat.eve), [clock](https://github.com/witheve/eve/blob/master/examples/clock.eve), and [todo-mvc](https://github.com/witheve/eve/blob/master/examples/todomvc.eve).

## Drawbacks

One of the lines we are balancing is making the syntax familiar yet different. The proposed syntax is indeed very different from what people are used to.

Another drawback is having a syntax at all, in that it forces us to maintain two disjoint interfaces for Eve: this textual syntax and a graphical interface for non-programmers. However, given the reasons outlined above, we believe this cost is worth paying.

## Alternatives

We’ve tried various syntaxes before this one. Earlier this year we had a syntax based on s-expressions, which you can read about [here](http://incidentalcomplexity.com/2016/06/10/jan-feb/) and [here](http://incidentalcomplexity.com/2016/06/22/mar2/). You can also look at our earliest syntax [here](https://github.com/witheve/eve-experiments/tree/syntax/examples). 

## Risks

The biggest risk here is that we will soon be introducing Eve to a much wider audience, and this syntax will be the face of Eve. Many people conflate a programming language with a syntax, and so a poor syntax will leave a poor lasting impression of Eve in general.
