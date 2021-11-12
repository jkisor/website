---
layout: post
title:  "Writing a testing library, test-first, using itself"
date:   2021-08-30 00:00:00 -0400
---

# Writing a testing library, test-first, using itself.

See the code on Github: [here](https://github.com/jkisor/assert/blob/a3712d0327ebacaa26e889580d7c875064e17de5/test.rb)

I've always wondered about authors of testing libraries. They clearly have some experience with testing. How do they test their testing library? Is it necessary to use a competing testing library to start?
Is it possible to "dog-food"? Wouldn't it be cool if I could write a small testing library, test-first, using itself.

I had so much fun doing this exercise that I thought it would make great content to walk through. It's also brief, allowing me to show some of the subtile small steps to pull it off.

## Test #1 - No tests succeeds
Using a test-first workflow, we start with a failing test. The first test would be to run a ruby script that doesn't exist yet.

```
ruby test.rb
ruby: No such file or directory -- test.rb (LoadError)
```

A runtime error. This is how we will expect tests to fail in general.
To make this test pass, we create the file and print out "Success".

```
# test.rb
puts "Success"
```

```
# ruby test.rb

Success
```

## Test #2 - Successful tests succeed

Next we test that asserting true will pass.

```
# test.rb

assert(true)
puts "Success
```

```
# ruby test.rb

Traceback (most recent call last):
test.rb:1:in `<main>': undefined method `assert' for main:Object (NoMethodError)
```

The test fails with a `NoMethodError`. We listen to it's advice and create the method.

```
# test.rb

def assert(value)
end

assert(true)

puts "Success
```

```
# command-line

ruby test.rb

Success
```

Though the method does nothing, it satisfies our passing critia of no runtime errors.

## Test #3 - Failing tests fail

The feeling of incompleteness is worth listening to. It's telling us the next move to make. We create a test that checks if false assertions fail.

```
# test.rb

# Failing tests fail
assert(false)
```

```
# command-line

ruby test.rb

Success
```

The test passes without needing to do anything, but we need failing test before continuing. We are forced to write the body of the method. The two tests are triangulating the problem.

If assertion is false, we want to raise an error. If assertion is true, we don't want to raise an error.

```
# test.rb

def assert(value)
  raise unless value
end
```

```
# ruby test.rb

Traceback (most recent call last):
        1: from test.rb:9:in `<main>'
test.rb:2:in `assert': RuntimeError (RuntimeError)
```

We wanted a failure and we got it. This is a weird situation that is unique to this exercise: We do want false assertions to raise errors, but we want our test NOT to raise errors to pass.

We need a way to flip a raised error to a no error for testing purposes.

``` test.rb
begin
  assert(false)
rescue
end
```

We wrap the false assertion test in a `begin`/`rescue` to rescue from the error. It's a bit of a brain teaser, but it comes with the situation.

```
# ruby test.rb

Success
```

## Refactor: Extract method

The previous move feels temporary. It's not very obvious or elegant, but it got the tests passing. This is a feeling worth listening to. This feedback is telling us to consider refactoring.

In a refactoring mindset, We should move in small steps and use the tests we've built for quick feedback. This is where test-first really shines. We are guarenteed everything we've built up is covered by the tests. Keeping us safe from unexpected changes in behavior.

The first move we can make is to give our blocky `begin`/`rescue` concept a name. This can be achieved by extracting a method.


```
def assert_error
  assert(false)
rescue
end

# Failing tests fail
assert_error

```

Not only have we introduced a descriptive name, we also notice we could make use of `assert_error` in other situations. Listening to this is telling us leads to the next move, but with passing tests we should capture this movement in a commit.

## Refactor: introduce callable lambda

The functionality of `assert_error` is limited. It's too specific to a single situation. It will always raise and rescue from its internal error. We should aim to make it more dynamic by letting the caller have more control by injection. This also can be achieved in a series of small, micro steps.

We identity the part we wish to be defined by the outside caller: `assert(false)` and wrap it in a lambda so it can eventually be passed in.

```
def assert_error
  -> { assert(false) }.call
rescue
end

# Failing tests fail
assert_error

```

Tests pass. It's good enough to continue.

## Refactor: Extract variable

Sticking to our goal, we can make another micro step by introducing a variable.

```
def assert_error
  callable = -> { assert(false) }
  callable.call
rescue
end

assert_error
```

## Refactor: Parameterize

Finally the pay off of the last few steps. We lift the assertion lambda into a parameter so the outside world can inject it's own assertions.

```
def assert_error(callable)
  callable.call
rescue
end

assert_error(-> { assert(false) })
```

## Refactor: Lambda to block

Next we switch from lambda to block as it's a bit more conventional in ruby than passing a lambda. It's also a little less heavy on the syntax.

```
def assert_error(&block)
  block.call
rescue
end

assert_error { assert(false) }
```

## Test #4: Fail when expected error never happens

This one is a neat one but a little bit of a brain teaser due to this exercise.
We can use `assert_error` wrapped in an `assert_error` to test this.

`assert_error { assert_error { 1+1 } }`
To explain, i'll work inside out:

The addition, `1+1`, definitely won't raise an error, but it's wrapped in `assert_error`, which from a testing library perspective WILL raise an error. So we wrap it in another `assert_error`.

```
def assert_error(&block)
  block.call
rescue
else
  raise
end

# Fail when expect error never happens
assert_error { assert_error { 1+1 } }
```

## Test #5: Comparing equivalent values succeeds

Techically we could use `assert(0 == 0)` but this will mean that the evaulation is squashed into a boolean. This will mean that test failures won't have enough context to show the difference between the values as a failure message.

This is where `assert_equal` will come in. It takes two arguments: actual and expect.

The success case we can get by with an empty method for now.

```
def assert_equal(actual, expected)
end

# Comparing equivalent values succeeds
assert_equal(0, 0)

```

## Test #6: Comparing different values fails

To get a real implementation for `assert_equals` we make use of `assert_error` we built previously. The implementation is then straight forward.

```
def assert_equal(actual, expected)
  raise if actual != expected
end

# Comparing different values fails
assert_error { assert_equal(0, 999) }
```

See the code on Github: [here](https://github.com/jkisor/assert/blob/a3712d0327ebacaa26e889580d7c875064e17de5/test.rb)