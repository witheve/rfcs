# Standard Library

## Summary

In this RFC we present our idea for the standard library in Eve. Most languages come with some set of built-in functions to provide basic utility. These functions can include math primitives, string manipulation, time and date utilities, etc.

## Motivation

Our goal for Eve is to be as useful as possible straight out of the box. When you download Eve, you should have everything necessary to go from a blank document to a database-driven dynamic webapp. In order to get there, we need to have a robust standard library -- something that provides functionality that is generally useful to a wide array of domains.

## Design

Eve's standard library will be invisible and included by default. There is no need to import modules.

Proposed standard library

- Math - includes math primitives, basic math functions, trigonometry
- Strings - Includes functions that manipulate strings
- Date & Time - Includes functions that provide and manipulate date and time records
- Statistics - Includes functions that calculate statistical measures on records

## Drawbacks

A large standard library can be onerous for a small project.

## Alternatives

## Risks