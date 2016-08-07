Discussion: https://github.com/witheve/rfcs/issues/4

# RFC - Developer Syntax

(A note before we get started: this is our first official Request for Comment (RFC). RFCs are meant to be an informal way to start a discussion on a particular feature, design, protocol, process, or anything else relating to Eve and the Eve community. As the discussion evolves, the RFC will become a form of documentation, explaining the genesis of a particular feature. You can read more about our RFC process [here](https://github.com/witheve/rfcs).)

## Summary

Today we are asking community feedback on our current developer syntax proposal. In this RFC I'll present a complete program written in the syntax and explain how it works. We hope to specifically hear opinions on the following:

1. How easy is it to read and understand code you haven't written in the proposed syntax?
2. How easy is it to write code in the proposed syntax?
3. Is the programming model clear?
4. Does the syntax support working and thinking in this model?
5. What changes would make writing or reading easier?
6. Did you experience any "ah ha!" moments while reading about the syntax? I.e., was there something we said or an example we gave that made the whole thing click?

## Motivation

In our [Jan/Feb dev diary](http://incidentalcomplexity.com/2016/06/10/jan-feb/) we talked a little about the need for a syntax during our development, even if ultimately most users will never see it. To summarize: while the graphical interface is under development, a textual syntax helps us test the platform, share code, and find/report/reproduce bugs.

We are specifically calling this a "developer syntax" because it is meant for software developers who know how to program. We want to make sure you understand this isn't how we will present Eve to people who don't know how to program.

In recent months, the semantics of the Eve language have stabilized to something we're very happy with, and quite excited about. However, the graphical model for interacting with the Eve language is still very much in flux. We've now reached a point where we want to share our work with the community, and we can't do that without a proper interface to the language.

Thus, to get Eve out to early adopters sooner, we have developed a textual syntax, quite different from what we've [shown so far](http://incidentalcomplexity.com/2016/06/30/apr/).

## Design

### Design Goals

Here are the broad design goals we identified when designing the syntax:

1. **For Humans** - This syntax is designed for humans, so decisions regarding the ergonomics of the syntax are of primary concern.
2. **Readable** - Since code is read more than written, we want the syntax to be eminently readable.
3. **Consistent** - The syntax should be consistent with prior knowledge, so that users unfamiliar with Eve can read an Eve program and figure out what's going on at a high level without explicitly knowing the syntax.
4. **Distinct** - This one is purposefully in contention with goal (3); we want the syntax to be familiar but not too familiar. If our syntax is too close to other languages (e.g. if we used C-style curly braces), we might project that our semantics are similar, when in fact they are very different.

### Programming Model

To really understand a syntax, you have to also understand the semantics of the underlying programming model, so that's where we will start. This discussion will be very high-level, so don't worry about the details for now. First, remember that Eve is not just a language; it is also a database. These are not separate components that interact; they are actually one and the same. However, sometimes we use "Eve DB" to refer to the underlying facts in the database.

At its core, Eve only responds to two commands:

1. What facts do you know about this "object"?
2. Remember a new fact about this "object".

Communication with Eve happens through "objects", which are key-value pairs attached to a unique ID (object is a pretty generic and overloaded term, so let us know if you have ideas for what to call these guys). To access facts in the Eve DB, you use an object. To insert/remove facts into/from the Eve DB, you also use an object.

Computation occurs as a result of relationships between objects. For example, I might model myself as an object with an `age` and a `birth year`. There might also be an object representing the `current year`. Then I could compute my `age` as my `birth year` subtracted from the `current year`.

A key concept here is that age is a derived fact, supported by two other facts: `birth year` and `current year`. If either of those supporting facts are removed from the Eve DB, then `age` can no longer exist as well. For intuition, think about modeling this calculation in a spreadsheet using three cells, and what would happen to the `age` cell if you deleted one of the other two cells.

Thus the presence or absence of facts can be used to control the flow of a program. To see how this works, consider how a navigation button on a webpage might work. When the button is clicked, a fact is inserted into the Eve DB noting the element that was clicked. This click fact would support an object that defines the page to be rendered. This in turn would support the page renderer, whose job it is to display the page.

One last thing to note about control flow is that we have no concept of a loop in Eve. Recursion is one way to recover looping, but set semantics and aggregates often obviate the need for recursion. In Eve, every value is actually a set. With operators defined over sets (think `map()`) and aggregation (think `reduce()`) we can actually do away with most cases where we would be tempted to use a loop.

#### Set Semantics

One other thing to know about Eve is that objects follow [set semantics](https://en.wikipedia.org/wiki/Set_(mathematics)). Sets are collections where every element of the collection is unique. This is in contrast to bag semantics, where elements can be duplicated. We'll see the implications of this later, but it's important to keep in mind.

### A Complete Program - Party Planning

Through the rest of this document, we'll refer to the following complete Eve program. Don't worry about understanding it right now; we'll go over what all the parts mean, and then hopefully the program will become clear. Let's dive right in:

    # Planning my birthday party

    This program figures out who can attend my party, and calculates how many burgers
    I need to buy.

    First I invite friends to the party. I can only invite friends who are not busy
    on the date of the party. Each one of my friends has marked their busy dates for
    me.

    ```
    match
      party = [@"my party" date]
      friend = [#friend busy-dates != date]
    bind
      friend += #invited
    ```

    Guests are allowed to bring their spouses, so I have to invite them as well. I
    won't invite spouses of friends who can't come to the party though.

    ```
    match
      [#invited spouse]
    bind
      spouse += #invited
    ```

    I'm only serving burgers at my party. Guests can have between 0 and 3 burgers.
    Vegetarians don't want any burgers, the standard amount is 1, and `#hungry`
    guests get 2. My friend `@Arthur` gets 3 burgers, because he's exceptionally
    hungry. I need to keep track of the total number of burgers needed, as well as
    how many each guest prefers.

    ```
    match
      guest = [#invited name]
      party = [@"my party"]
      burgers = if guest = [@Arthur] then 3
                else if guest = [#hungry] then 2
                else if guest = [#vegetarian] then 0
                else 1
      total-burgers = sum(burgers given burgers, guest)
    bind
      party.burgers := total-burgers
      guest.burgers := burgers
    ```

    Finally, I need to display the guest list and how many burgers I need total.

    ```
    match
      [@"my party" burgers]
      guest = [#invited name]
      burger-switch = if guest.burgers = 1 then "burger"
                      else "burgers"
    bind
      [#div children:
        [#h1 text: "Guest List"]
        [#div text: "{{name}} will eat {{guest.burgers}} {{burger-switch}}" sort: name]
        [#h2 text: "Total burgers needed: {{burgers}}"]]
    ```

    ## Data

    The rest of the code establishes the data needed to run the above program. I
    instantiate the party here.

    ```
    match
      [#session-connect]
    commit
      [@"my party" date: 2]
    ```

    I also need to add facts about my friends in order for this to all work.

    ```
    bind
      [#friend name: "James", busy-dates: 1 #hungry, spouse: [@Sam]]
      [#friend name: "Betty", busy-dates: 2, spouse: [@Joe], #hungry]
      [#friend name: "Carol", busy-dates: 3 #vegetarian]
      [#friend name: "Duncan", busy-dates: 4]
      [#friend name: "Arthur", busy-dates: 3, #hungry]
      [name: "Sam" busy-dates: 3]
      [name: "Joe" busy-dates: 4]
    ```

    ## Results

    Expected result:

    ---

    # Guest List

    Arthur will eat 3 burgers
    Carol will eat 0 burgers
    Duncan will eat 1 burger
    James will eat 2 burgers
    Sam will eat 1 burger

    ## Total burgers needed: 7

**Note: At the moment, this program does not execute correctly. We're working on adding the necessary features**

#### Program Structure

The first thing to notice is the broad structure of the program. In the spirit of [Literate Programming](https://en.wikipedia.org/wiki/Literate_programming), Eve programs are primarily prose, interleaved with Eve code. Donald Knuth explains Literate Programming in his [influential paper](http://www.literateprogramming.com/knuthweb.pdf):

> The practitioner of literate programming can be regarded as an essayist, whose main concern is with exposition and excellence of style. Such an author ...  strives for a program that is comprehensible because its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reinforce each other.

This description fits with the ethos of Eve - that programming is primarily meant to communicate with other humans, not the computer. You'll notice the above Eve program is actually written in two languages: Markdown, used to format the prose; and Eve, which is delineated by standard Markdown code blocks. Only the content within a block is compiled, while everything else is disregarded as a comment.

Writing code this way has several properties that result in higher quality programs:

- Literate programming forces you to consider a human audience. While this is usually the first step in writing any document, in programming the audience is typically a machine. For an Eve program, the audience might be your collaborators, your boss, or even your future self when revisiting the program in a year. By considering the audience of your program source, you create an anchor from which the narrative of your program flows, leading to a more coherent document.
- The human brain is [wired to engage](https://blog.bufferapp.com/science-of-storytelling-why-telling-a-story-is-the-most-powerful-way-to-activate-our-brains) with and remember stories. Think back to a book you read (or maybe a show you watched) last year. You probably remember in great detail all of the characters and their personalities, the pivotal moments of the plot, the descriptions of the various settings, etc. But how much can you remember of a piece of code you haven't looked at for a year? Literate programming adds another dimension to your code that will help you keep more of your program in working memory.
- Since Eve code blocks can be arranged in any order, literate programming encourages the programmer to arrange them in an way that makes narrative sense. Code can have a beginning, middle, and end just like a short story. Or like an epic novel, code can have many interwoven storylines. Either way, the structure of the code should follow an order imposed by a narrative, not one imposed by the compiler.
- It's often said that you don't really understand something until you explain it to someone else. Literate programming can reveal edge cases, incorrect assumptions, gaps in understanding the problem domain, and shaky implementation details before any code is even written.

Literate programming is a first-class design concept in Eve. We will be writing all of our programs in this manner, and will encourage others to do the same for the reasons above. That said, there is nothing in the syntax that specifically requires literate programming; you can write your program as a series of code blocks without any prose, and it will be perfectly valid.

### Objects

Objects are the predominant datatype in Eve. In the proposed syntax, objects are a set of attribute:value pairs enclosed in square brackets:

```
object = [ attribute1: value1 attribute2: value2 ...  attributeN: valueN]
```

Objects are essentially pattern matches against the Eve DB, i.e. objects ask Eve to find all the entities that match the supplied attribute shape. For example, our first object in the program is `[@"my party" date]`. The resulting object will consist of all the facts matching a `name` attribute with value "my party" and a date attribute with any value.

The object also binds `date` to the top level `date` variable, accessible outside of the object (but only within the block). If you want to use `date` to mean something else, then you can alias it using the bind operator (see the next section).

Objects can be bound to a variable, e.g. `party = [@"my party" date]`. This provides a handle to the object, allowing you to access and mutate attributes using dot notation, e.g. `party.date`.

### Binding, Equivalence, and Names

Our syntax has two binding operators: colon ( `:` ), and equals ( `=` ). By convention, colon is used within objects, and equals is used outside of objects. Either way, binding works the same: binding asserts that the left hand side of the operator is equivalent to the right hand side. This is distinct from assignment, which is not a concept in Eve (therefore, we have no need for an `==` operator).

Names are another way to say one thing is equivalent to another; within a block, variables with the same name represent the same object or attribute of an object. For instance:

```
People who are 50 years old
  [tag: "person" age]
  age = 50

The same query as above
 [#person age: 50]

Never true
 [#person age: 10]
 person.age = 20
```

Names are a little more permissive in our syntax than other languages. We allow most symbols in a name (with the exception of space, @, #, //, period, question, comma, colon, and grouping symbols). So operators like `-` and `+` are valid symbols in a name. Furthermore, we support Unicode, so you can include symbols (such as letters from the Greek alphabet). Such permissive naming comes at the cost of requiring whitespace in expressions. For example `friend-age` is a name, whereas `friend - age` is subtracting `age` from `friend`.

### Object Names and Tags

We've identified two attributes that are generally useful, so we've given them special syntax. These attributes are `name` and `tag`.

#### Name Selector ( `@` )

The name selector is used to select a specific named object from the Eve DB. Named objects are just objects with a `name` attribute specified. In the example program,`[@"my party"]` is shorthand for `[name: "my party"]`.

#### Tag Selector ( `#` )

The tag selector is used for selecting a group of similar objects, i.e. objects with the same tag attribute. In the above example, we used `[#friend]`, which is shorthand for `[tag: "friend"]`.

### Block structure

Blocks themselves have their own structure. Let's focus in on the first block:

```
match
  party = [@"my party" date]
  friend = [#friend busy-dates != date]
bind
  friend += #invited
```

Each block is written in two phases: `match` then `action`. These phases mirror the two Eve commands outlined earlier. In the `match` phase, we ask Eve for known facts, and we might transform those facts using temporary variables. In the `action` phase we perform some mutation on the Eve DB to either add or remove facts. Let's look at each of those phases now:

#### Phase 1: Match

The `match` phase is used to gather all the information you need to complete your block. The `match` phase is prefaced with the `match` keyword, and can potentially be omitted.

In the following block, we want to count all the guests coming to the party. To do this, we need the date of the party, a list of all my friends and their availability, and then a count of the guests on the list. Below, I've annotated what's going on in the `match` phase.

```
match
  party = [@"my party" date]              // Select the party and the date of the party
  friend = [#friend busy-dates != date]   // Select friends who are not busy during the date of the party
```

Let's take a look at other things you can do in the `match` phase:

##### **Aggregates**

Aggregates are functions that take an input set and produce an output set, typically with a different cardinality than the input set. For example, `count` takes an input set of cardinality `N` and produces a set of cardinality `1` as a result. A familiar analogue in other languages is the `reduce()` function. Here is an example of an aggregate in use:

```
 total-burgers = sum(burgers given burgers, guest)
```

Aggregates are called like functions in other languages, but there is a slight difference; the keyword `given` specifies the set we are summing over.

Recall that a set is an unordered collection of unique elements. In our example, `burgers = (3, 0, 1, 2, 1)`, which as a set is `{3, 0, 1, 2}`. Thus `sum(burgers given burgers) = 6`, which is not what we expect. However, when we say `sum(burgers given burgers, guests)` then each burger is associated with a corresponding guest, making each element unique, e.g. `(burgers, guest) = {{3, Arthur}, {0, Carol}, {1, Duncan}, {2, James}, {1, Sam}}`. Summing burgers given this set yields the expected result of `7`, because the duplicated one is now unique.

##### **If**

`If` allows conditional equivalence, and works a lot like `if` in other languages. Our `if` has two components: The keyword `if` followed by a conditional; and the keyword `then` followed by one or more return objects. An optional `else` keyword indicates the default value:

```
burger-switch = if guest.burgers = 1 then "burger"
                else "burgers"
```

This block is used to switch between the singular and plural for displaying the number of burgers each guest is eating. `If` statements can be composed, permitting the creation of complex conditional statements. For instance, instead of inviting friends and their spouses in two blocks (the first two blocks in the example program), I could have done it in a single block using an `if` statement:

```
[@"my party" date]
friend = [#friend busy-dates != date]
guest = if friend then friend
        if friend.spouse then friend.spouse
```

This is equivalent to a `union/and` operator, which combines elements from disparate sets into the same set. The second way to use `if` is in conjunction with the `else` keyword:

```
burgers = if guest = [@Arthur] then 3
          else if guest = [#hungry] then 2
          else if guest = [#vegetarian] then 0
          else 1
```

This is equivalent to a `choose/or` operator, selecting only the first branch with a non-empty body. A bug in this program is that if some guest is tagged both `#hungry` and `#vegetarian`, that guest will actually receive two burgers. Therefore, while order of statements usually does not matter in Eve, `if` statements are one area where it does.

A final feature of the if statement is multiple returns. For instance, we could have done this:

```
(burgers, status) = if guest = [@Arthur] then (3, #fed)
                    else if guest = [#hungry] then (2, #fed)
                    else if guest = [#vegetarian] then (0, #needsfood)
                    else (1, #fed)
```

##### **Not**

Not is an anti-join operator, which takes a body of objects. For example, we can get a list of people who are not invited to the party:

```
friends not invited to the party
  friends = [#friend]
  not(friend = [#invited])
```

##### **Expressions**

Expressions are used to perform calculations using constants and attributes of objects, e.g. `party.burgers / party.guest-count` would calculate the ratio of burgers to the number of guests. Note that `guest-count` is a variable, not an expression. In expressions, you are required to add whitespace between operators. This helps with readability, but it also allows us to add more characters to names.

Operators are defined over sets, so you could do something like `cheese-slices = guest.burgers * 2`, which would multiply every guest's burger count by two. There is a pitfall here: if you perform an operation on disjoint sets (they have no attribute in common) with differing cardinality (their sets have a different number of elements), the result will be a [Cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) of the two sets. This is usually not a desired behavior, but sometimes it is what you want to do, e.g. if you wanted to calculate the distance from every point in a set to every point in another set, a Cartesian product would be useful.

##### **Functions**

**Note: This section of the proposal is not implemented yet**

It turns out that functions as they exist in other languages are mostly obviated by Eve's tag semantics. Consider the following two statements

```
x = sin(90)                // A typical function call
[#sin deg: 90, return: x]  // An Eve object
```

These statements accomplish the same objective, of storing the sine of an angle in a result variable. The Eve syntax is at a disadvantage though, because it cannot be composed into an expression like a typical function. Therefore, we propose the following syntax for functions in Eve:

```
x = sin[deg: 90]
```

which is sugar for:

```
[#sin #function deg: 90, return: x]
```

The return attribute is implicitly the value of `sin[deg]`, so now the object can be used and composed like functions in other languages. We're proposing this syntax for several reasons.

- Square brackets draw attention to the fact that the function call is nothing more than a regular object.
- Explicit parameters are self-documenting, which makes the code more readable if you're not familiar with the function signature.
- Explicit parameters permit arguments in any order, which makes optional arguments easy to implement.
- Finally, since functions are really just objects, you can extend a function so it can be used in new ways. For example, we could extend the `sin` function to support radians:


        Calculate the sine of an angle given in radians
        ```
        match
          return = sin[angle: value? * π / 180]
        bind
          sin[rad: value?, return]
        ```

The `?` notation here indicates that the value is an input. We can use the extended function like so:

```
x = sin[rad: π / 2]
```

##### **String Interpolation**

We support string interpolation (concatenation) using double curly braces embedded in a string, e.g. `"{{ }}"`. In the party program, when we print output, we use string interpolation to format the results:

```
bind
  [#div children:
    [#h1 text: "Guest List"]
    [#div text: "{{name}} will eat {{guest.burgers}} {{burger-switch}}" sort: name]
    [#h2 text: "Total burgers needed: {{burgers}}"]]
```

The `div` that prints the guest name is only written once, but because of set semantics, it will be printed as many times as there are elements in the set (in this case we'd expect it to be printed five times). For the same reason, the total burger count will only be printed once.

#### Phase 2: Action

The `action` phase of a block indicates that we are changing the Eve DB in some way. This phase is preceded by either the `bind` or `commit` fences (explained below). While the `match` phase can be omitted, omitting the `action` phase is probably an error, i.e. the block doesn't do anything without an `action`.

The transition to the `action` phase means we're no longer able to use any statements available in the `match` phase, e.g. `if`, `not`, aggregates, expressions, etc.

##### Adding and Removing Objects

Objects can be added to Eve after an `action` fence:

```
commit
  [@"my party" date: 2]
```

Objects can be removed from Eve using the `none` keyword. For example, we could remove `@"my party"` like so:

```
commit
 [@"my party"] := none
```

##### Mutation Operators

We have four operators for mutating objects in the Eve DB: add, set, remove and merge:

- Add ( `+=` ) - adds values to an attribute
- Set ( `:=` ) - sets the value of an attribute
- Remove ( `-=` ) - removes attribute with value from an object
- Merge (`<-`) - merges one object with another

###### Add, Set, and Remove

Add, set and remove all work similarly. On the left hand side of the operator you provide an object and attribute through dot notation. On the right hand side, you provide a value, that will either add to, set, or remove from the right hand side attribute. For example:

```
party.burgers := total-burgers
```

sets the `burgers` attribute on the `party` object to the value `total-burgers`. An exception to this is when the value is a tag or name. In either case, you don't have to specify an attribute on the right hand side. For example, the following adds the tag `#invited` to the `friend` object.

```
friend += #invited
```

Mutations follow set semantics. If an attribute exists on an object, using `+=` will just add it to the set. For instance, if `person.age = {10}`, and `person.age += 20`, then `person.age = {10, 20}` (note: the curly braces are not part of the syntax, but are a standard way of indicating a set).

###### Merge

Merge works differently from the other three operators. The purpose of the merge operator is to merge two objects together, i.e. you're taking the set union of the objects. On the left hand side, you just provide an object handle. On the right hand side, you provide a new object. For instance:

```
guest <- [burgers: 3]
```

This merges the new object into `guest`, setting `burgers` to `3`.

##### Commit vs. Bind

We have two fences for the `action` phase, with differing semantics. The `commit` fence tells Eve that any object behind the fence should persist in the database, even if the supporting data in the `match` phase is removed. For example:

```
match
  [#session-connect]
commit
  [@"my party" date: 2]
```

The use of `commit` here means that if `#session-connect` ever exists in the database, then the object `[@"my party" date: 2]` exists even if `#session-connect` is removed in the future. In fact `#session-connect` exists for only a single tick of compiler time, so if we didn't use `commit`, then the party would only exist for an instant and disappear.

By contrast, the `bind` fence tells Eve that any object behind the fence is bound to the data in the `match` phase, and therefore only exists if that data exists. For example:

```
match
  party = [@"my party" date]
  friend = [#friend busy-dates != date]
bind
  friend += #invited
```

The behavior of this code is that a `friend` is `#invited` as long as they are not busy during the date of the party. Let's say my friend `@Arthur`'s calendar is initially clear, and so he is originally `#invited`. Then some time later, `@Arthur` suddenly adds the party date to his list of busy dates. Now, he no longer satisfies the conditions of the block. Therefore, Eve removes `#invited` from `@Arthur`, and he no longer shows up on the guest list, which also subsequently lowers the burger count. Had we used the `commit` fence, then his initial invitation would be permanent until we explicitly remove it.

##### Commit global

By default, any changes made to the database are per session. This means any facts added to the database are only visible to the session that added them. `Commit` can be optionally followed by the `global` keyword, which indicates that the fenced objects are available globally to all sessions connected to Eve.

This is useful if you want to create a networked application. For our example, I might ask all my friends to write the following query:

    Hi friends. Please edit the following Eve code for my party planning app.
    Just fill in your name, the dates you are busy, and add your spouse as well.
    You can also add either of the following tags: #hungry, and #vegetarian.

    ```
    match
      [#session-connect]
    commit global
      [#friend name busy-dates spouse]
    ```

Now, when my friends execute that block (filled in with their details), their data is available to my party planning application.

## Other Examples

Feel free to check out some more examples of programs written in the proposed syntax [here](https://github.com/witheve/eve/tree/master/examples). In particular, the following are complete applications: [chat](https://github.com/witheve/eve/blob/master/examples/chat.eve), [clock](https://github.com/witheve/eve/blob/master/examples/clock.eve), and [todo-mvc](https://github.com/witheve/eve/blob/master/examples/todomvc.eve).

## Drawbacks

One of the lines we are balancing is making the syntax familiar yet different. The proposed syntax is indeed very different from what people are used to.

Another drawback is having a syntax at all, in that it forces us to maintain two disjoint interfaces for Eve: this textual syntax and a graphical interface for non-programmers. However, given the reasons outlined above, we believe this cost is worth paying.

## Alternatives

We've tried various syntaxes before this one. Earlier this year we had a syntax based on s-expressions, which you can read about [here](http://incidentalcomplexity.com/2016/06/10/jan-feb/) and [here](http://incidentalcomplexity.com/2016/06/22/mar2/). You can also look at our earliest syntax [here](https://github.com/witheve/eve-experiments/tree/syntax/examples).

## Risks

The biggest risk here is that we will soon be introducing Eve to a much wider audience, and this syntax will be the face of Eve. Many people conflate a programming language with a syntax, and so a poor syntax will leave a poor lasting impression of Eve in general.

