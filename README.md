# sigmatch
Yet another way to overload functions

## Motivation
I very often find myself in situations where I want to create a function
that supports several alternative call signatures. In javascript (or Coffeescript)
you usually do this by looking at the `arguments`-Object to find out how many and 
what kind of arguments were given. This is typically followed by an awkward bit of shuffeling
where you decide what the actual name and content of each parameter would be before dispatching
to the code that does the actual work.

The `sigmatch` module allows you to do this in a more declarative way. 

## Install

```
npm install --save sigmatch
```

## Usage

This example is based on a real use of this module in the [tbob project](https://github.com/ldegen/tbob/blob/master/src/dsl.coffee).

Let's say you are writing a DSL describe json-like data structures.
You want to create a directive for describing attributes.
The attribute has a name and an associated value type. It is also possible to give a
default value for that attribute or even a strategy that calculates that default value
in dependency to the value of other attributes.
So the most general signature would look something like this:

```coffeescript
createAttribute = (name, type, dependencies, defaultStrategy)->
  # actual code to create the attribute
```

Now, you wouldn't want this in a DSL. So here is what you could do, using sigmatch:


``` coffee
sigmatch = require "sigmatch"
attributeDirective = sigmatch (match)->
  # the most "complete" case:
  match "s,o,a,f", createAttribute
  # use an optional static default value
  match "s,o,.?", (name, typeExpr, defaultValue)-> 
    createAttribute name, typeExpr, [], ->defaultValue
  # do not specify a type
  match "s,a,f", (name, deps, defaultStrategy)->
    createAttribute name, any(), deps, defaultStrategy
  # same, but with static default value
  match "s,.", (name, defaultValue)->
    createAttribute name, any(),[], ->defaultValue
  # catch-all rule to produce an error
  match ".*", -> 
    throw new Error("unsupported call signature")
```

## Rules
As seen in the above example, 
`sigmatch` is taking a function as its single argument. It calls that function with a directive
for creating rules. We called it `match` in the above example.

To create a rule, you call `match` with a *pattern* and an *action*.
The pattern is a comma-separated list of *attribute matchers*. 
The following matchers are supported:

| Matcher | Meaning                                         |
|---------|-------------------------------------------------|
| `.`     | matches any argument (but not `undefined`)      |
|  `s`    | matches a string                                |
|  `n`    | matches a number                                |
|  `a`    | matches any array                               |
|  `o`    | matches any object (but not functions or arrays)|

By default, a single matcher will match exactly one argument.
This can be changed by appending *multiplicity* modifier:

| Modifier | Matches                      | Produces                                |
|----------|------------------------------|-----------------------------------------|
| (none)   | match exactly one argument   | a single value                          |
| `?`      | match zero or one argument   | a single value or null                  |
| `+`      | match one or more arguments  | an array containing at least one element|
| `*`      | match zero or more arguments | an array                                |

When the pattern matches, the values produced by the matchers will be passed to the
action.
`sigmatch` will try the given rule one after another and stop after the first matching
rule. It will return whatever the action of that rule returned.
