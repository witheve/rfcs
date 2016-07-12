# RFC - Developer Syntax

(A note before we get started: this is our first official Request for Comment (RFC). Taking a page out of the Rust playbook, we are not really going to define a rigid process here. RFCs are meant to be an informal communication for starting a discussion on a particlar feature, design, protocol, process, or anything else relating to Eve and the Eve community. RFCs will 

## Summary

Today we are soliciting community feedback on our current developer syntax proposal. In this RFC I'll present a complete program written in the syntax and explain how it works. We hope to specifically solicit comments on the following:

1. How easy is it to read and understand code you haven't written in the proposed syntax?
2. How easy is it to write code in the proposed syntax (despite that right now you can't actually execute much code)?
3. Is the programming model clear? 
4. Does the syntax support working and thinking in this model?
5. What changes would you make to writing or reading easier?
6. Did you experience any "ah ha!" moments while reading about the syntax? i.e. Was there something we said or an example we gave that made the whole thing click?

## Motivation

In our [Jan/Feb dev diary](http://incidentalcomplexity.com/2016/06/10/jan-feb/) we talked a little about the need for a syntax for development, even if ultimately most users will never see it. We are specifically calling this a "developer syntax" because it is meant for software developers who know how to program.  

To summarize: while the graphical interface is under development, a textual syntax helps us test the platform, share code, and find/report/reproduce bugs.

In recent months, the semantics of the Eve language have stabilized to something we're very happy with, and quite excited about. However, the graphical model for interacting with the Eve language is still very much in flux. We've now reached a point where we want to share our work with the community, but we can't wait for the graphical model to catch up.

Thus, to get Eve out to early adopters sooner, we have developed a textual syntax, quite different from what we've [shown so far](http://incidentalcomplexity.com/2016/06/30/apr/).

## Design

### Design Goals

Here are the broad design goals we identified when designing the syntax:

1. For Humans - This syntax is designed for humans, not compilers.
2. Readable - Since code is read more than written, we want the syntax to be eminently readable.
3. Familiar - Users unfamiliar with Eve should be able to read an Eve program and figure out what's going on at a high level.
4. Different - This one is purposefully in contention with goal (3); we want the syntax to be familiar but not *too* familiar. After all, Eve itself is very different from most languages out there, so we don't want to se the appropriate expectations.

With these design goals in mind, I will walk you through a program written in our proposed syntax, pointing out the major features along the way. Hopefully you can let us know where we succeeded, and where we need to focus more work. Let's just dive right in:

### Programming Model

To really understand a syntax, you have to also understand the semantics of the underlying programming model, so that's where we will start. This discussion will be very high-level, so don't worry about the fine details for now. First, remember that Eve is not just a language; it is also a database. These are not separate components that interact; they are actually one and the same. However, sometimes we use "Eve DB" to refer to the underlying facts in the database.

At its core, Eve only reponds to two commands:

1. What facts do you know about `x`?
2. Remember a new fact about `x`.

Communication with Eve happens through "objects", which are key-value pairs attached to a unique ID (object is a pretty generic and overloaded term, so let us know if you have ideas for what to call these guys). To access facts in the Eve DB, you use an object. To insert/remove facts into/from the Eve DB, you also use an object. 

Computation, occurs as a result of relationships between objects. For example, I might model myself as an object with an `age` and a `birth year`. There might also be an object representing the `current year`. Then I could compute my `age` as my `birth year` subtracted from the `current year`. 

A key concept here is that age is a derived fact, supported by two other facts: `birth year` and `current year`. If either of those supporting facts are removed from the Eve DB, then `age` can no longer exist as well. For intuition, think about modeling this calculation in a spreadsheet using three cells, and what would happen to the `age` cell if you deleted one of the other two cells.

Thus the presence or absence of facts can be used to control the flow of a program. To see how this works, consider how a naviagtion button on a webpage might work. When the button is clicked, a fact is inserted into the Eve DB noting the element that was clicked. This click fact would support an object that defines the page to be rendered. This in turn would support the page renderer, who's job it is to display the page.

One last thing to note about control flow is that we have no concept of a loop in Eve. Recursion is one way to recover looping, but set semantics and aggregates often obviate the need for recursion. In Eve, every value is actually a set. With operators defined over sets (think `map()`) and aggregation (think `reduce()`) we can actually do away with most cases where we would be tempted to use a loop.

### A Working Program - Party Planning

Through the rest of this document, we'll refer to the following complete Eve program. Don't worry about not understanding it; we'll go over what all the parts mean, and then hopefully the function will become clear.

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
  save
    [@"my party" date: birthday]
```

### Program Structure

The first thing to note is the broad structure of the program. An Eve program consists of any number of blocks. In this program, we have three blocks, which are are delineated by indention. Any text at zero-indent is treated as a comment. Anything at greater than one-indent is treated as code. Blocks are terminated at EOF or the next zero-indent line. Inline comments are possible using the `//` prefix anywhere in the code.

At the beginning of each block is a "block header" e.g. "How many burgers do I need?". These are meant primarily for documentation, and by convention they are a brief description of the purpose of the block. These are not "function handles" or anything that you call in code.

### Objects

As noted in the Programming Model section, objects are the prevailing datatype in Eve. In this syntax, objects are a set of attribute:value pairs enclosed in square brackets:

```
object1 = [ attribute1: value attribute2: value ...  attributeN: value]
```

Objects are essentially pattern matches against the Eve DB, i.e. objects ask Eve to find all the entities that match the supplied attribute shape. For example, our first object in the program is `party = [name: "my party", date]`. The resulting `party` object will consist of all the facts matching a `name` attribute with value "my party" and a date attribute with any value.  

The `party` object also binds `date` to the top level `date` variable, accessible outside of the object. If you want to use `date` to mean something else, then you can alias it using the bind operator (see the next section). You can also access the unmatched attributes of an object using dot notation e.g. `party.date`.

### Binding, Equivalence, and Names

Our syntax has two binding operators: colon ( `:` ), and equals ( `=` ). By convention, colon is used within objects, and equals is used outside of objects. Either way, binding works the same: binding asserts that the left hand side of the operator is equivalent to the right hand side. This is distinct from assignment, which is not a concept in our language.

Names are another way to say one thing is equivalent to another; within a block, variables with the same name represent the same object. This includes including attributes of objects. For instance:

```
People age 50
   [tag: "person" age]
   age = 50

The same query as above
  [#person age: 50]

Never true
  [#person age: 10]
  person.age = 20
```

Names are a little more permissive in our syntax than other languages. We allow most symbols in a name (with the exception of space, @, #, //, period, question, comma, colon, and grouping symbols). So operators like '-' and '+' are valid symbols in a name. This comes at the cost of requiring whitespace in expressions. For example `friend-age` is a name. By contrast `friend - age` is subtracting age from friend.

### Name Selector ( `@` )

The name selector is used to select a specific named entity from the Eve DB. Named entities are just entities with a `name` attribute specified. In the example program,`[@"my party"]` is shorthand for `[name: "my party"]`.

### Tag Selector ( `#` )

Tag selectors are used for selecting groups of similar entities i.e. entities with the same tag attribute. In the above example, we used `[#friend]`, which is shorthand for `[tag: "friend"]`.

### Block structure

Let's focus in on the first query:

```
count guests coming to the party
  party = [@"my party" date]
  guest = if p = [#friend busy-dates: not(party.date)] then p
          if [#friend spouse busy-dates: not(party.date)] then spouse
  total = count(given guest)
  maintain
    party.guest-count := total
    party.guest += guest
```

Blocks themselves have their own structure as well. Each block is written in two phases: collect then mutate. These mirror the Eve commands outlined above. In the collect phase, we ask Eve for known facts, and in the mutate phase we tell Eve to remember new facts. Let's look at each of those phases here:

#### Collect

The collect phase is used to gather all the information you need to complete your block. In the following block, we want to count all the guests coming to the party. To do this, we need the date of the party, a list of all my friends and their availability, and then a summation of the result. Below, I've annotated what's going on in the collect phase.

```
  // Select the party
  party = [@"my party" date]

          // Add my friends to the guest list if they are not busy
  guest = if p = [#friend busy-dates: not(party.date)] then p

          // Add my friends' spouses to the list if they are not busy
          if [#friend spouse busy-dates: not(party.date)] then spouse

  // Count all the total number of guests
  total = count(given guest)
```

Let's talk about what you can do in the collect phase:

##### Aggregates

That last line there is an aggregate. Aggregates are called like functions in other languages, but there is a slight difference. Here, the keyword `given` specifies the set we are counting. In the second query, take a look at this aggregate:

`total = sum(burgers given burgers, guest)`

Here, we are summing `burgers`, given the set of `burgers` and `guest`. This is important because of our set semantics. If we have just `sum(given burgers)`, then the duplicate elements of burgers would be removed and we would arrive at the wrong number. By adding `guest` to the set, we ensure that every guest’s burger quantity properly counted.

##### If

`If` allows conditional equivalence, and works a lot like `if` in other languages. In the above example, we add guests to the list if they are a friend and not busy, or if they are a spouse of a friend and not busy.

`If` can be used in two ways: one, you can chain a series of `if` statements, like in the example above. This is equivalent to a union operator. The second way is demonstrated here:

```
  burgers = if guest = [#hungry] then 2
            else if guest = [@arthur] then 3
            else if guest = [#vegetarian] then 0
            else 1
```

In this case, `if` is used as a `choose` operator, selecting only the first branch with non-empty results.

##### Not

Not is an anti-join operator, which takes a body of objects. In our example, we've specified the date of "my party", and each `#friend` has specified the dates they are busy. A `#friend` can only go to the party if the date of the party is not on her list of busy dates. e.g. `[#friend busy-dates: not(party.date)]`.

##### Expressions

Expressions are used to perform caluclations using constants and attributes of objects e.g. `employee.salary * employee.bonus * 1.9`. Expressions are defined over sets, so the preceeding expression will work even if the cardinality (the number of elements in the set) of `employee.salary` is greater than one.

There is a potential pitfall here: if the cardinality of the attributes in an expression are different, then the cardinality of the expression result will be a cartesian product of the attribute sets. For instance, if `|employe.salary| = 3` and `|employee.bonus| = 4`, then the result of the expression will have cardinality 12. This may actually be the desired behavior for some applications, but it could lead to unexpected behavior.      

#### Mutate

(We're also not happy with the mutate nomenclature. Any suggestions are appreciated).

In the mutate phase, we're ready to change the Eve DB in some way. The mutate phase is fenced off with either the `save` or `maintain` keywords (explained below). By convention, the mutate phase is indented twice. The implication of the fence are that we are telling Eve that we're not longer interested in asking questions about objects. Thus we're no longer able to use any statements applicable to the collect phase e.g. `if`, `not`, aggregates. If you feel like you need to use one of these things in the mutate phase, then 

Let's look at what the first query does in the mutate phase:

```
  // maintain tells us that we are updating the database
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

Objects can be saved to the Eve DB just by adding them after a mutation fence:

```
  [@Corey birthday]
  save
    [@"my party" date: birthday]
```

Objects can be removed from Eve using the `none` keyword. For example, we could remove `@"my party"` like so:

```
save
  [@"my party"] := none
```

##### Mutation Operators

We have three operators for mutating objects in the Eve DB: add, set, and remove:

- Add ( `+=` ) - adds attributes to an object.
- Set ( `:=` ) - sets the value of attributes on an object
- Remove ( `-=` ) - removes attributes from an object

Mutation operators can be used in two ways. First, you can add/set/remove a specific attribute on an object. E.g. `object.attribute = value`. Values can be anything, including objects. The alternative way allows you to add/set/remove multiple attributes. E.g. `object += [attribute1: value, attribute2: value, ...]`.

Mutations follow set semantics. If an attribute exists on an object, using += will just add it to the set. For instance, if `person.age = 10`, and `person.age += 20`, then `person.age = {10, 20}` (note, the curly braces are not part of the syntax, but are a standard way of indicating a set). 

##### Save vs. Maintain

We have two fences for the mutation phase, with differing behaviors. When you use `save`, you're telling Eve that you want the subsequent facts to persist in the database, irrespective of the supporting data. To see what I mean by this, For example, look at the last query:

```
my party is on my birthday!
  [@Corey birthday]
  save
    [@"my party" date: birthday]
```

We use `save`, so even if `@Corey` or his birthday are deleted from the database, `@"my party"` will still remain.

By contrast, consider:

```
  guest = [#friend busy-dates: not(party.date)]
  maintain
    party.guest += guest
```

In this case, the fence is `maintain`, so we're specifying that `party.guest` will be automatically kept up-to-date. The behavior of this code is that a `guest` is a `#friend` who is not busy on the date of the party. Let’s say my friend Sam’s calendar is initially clear, and so he is originally on the guest list. Then some time later, Sam suddenly adds the party date to his list of busy dates. Now, he no longer satisfies the conditions of the block, and he is removed from the guest list (also subsequently lowering the burger count). Had we used the save fence, then his initial admittance to the guest list would be permanent, and we would have too many burgers for the party.

##### Code Reuse

Eve has no concept of a function, but code reuse can be achieved through objects. Note that we have several statements that look like function application (e.g. aggregates and not), but these are not user definable.  

To see how code reuse works, let's go back to our party example:

```
Calculate a time difference
  [#time date: today]
  time-remaining = targetdate - today
  maintain
    [#timediff targetdate? time-remaining]
```

Here we define a resuable block by leaving some variables unbound. The `?` calls out that the preceeding variable `targetdate` is unbound in the block. According to our semantics, this block isn't functional until `targetdate` is bound, which we can do in another block:

```
How long until my party?
  party = [@"my party" date]
  [#timediff targetdate: date, time-remaining]
  maintain
    party.timeleft = time-remaining
```

This completes the first block, so `party.timeleft` will be a continuously updating countdown to the date of the party.

## Examples

### Clock

### Chat

### TodoMVC

## Drawbacks

## Alternatives

## Unresolved questions