---
layout: post
title:  "Finding flaky RSpec tests"
date:   2021-10-14 12:31:21 -0400
---

Automated tests are a beautiful thing to have. Unfortunately, not all tests are created equally. Poorly written tests could be slow, fragile, or flaky. Today I'll share a way to locate test that fail at irregular intervals aka "flaky" tests. One test run an example will pass, another run it will fail. It is a flickering that can add unhelpful waste to your workflow such as surprising failures in your continious delivery (CD) builds.

See the full code on [GitHub](https://github.com/jkisor/flaky_finder)

## Collecting Samples
The strategy that I'll suggest here is to run your test suite multiple times (perhaps 10) and output the results to a JSON file for later analysis. This approach makes use of RSpecs default random order and it's [JSON formatter](https://relishapp.com/rspec/rspec-core/docs/formatters/json-formatter) (`--format j`)

## Analyzing Samples
Once you have all the results in JSON format, You can iterate over the test runs looking for any test outcomes that were inconsistent. If `untitled_spec.rb:22` only passed 8 out of 10 times, Congratulations you've found a troublemaker.

## Writing code to help
This process can be captured in code. Here's a snippet of high-level code that expresses what I've explained above.

```
Runs.new(
  (1..sample_size)
    .map { |_| `bundle exec rspec #{path} --format j` }
    .map { |results| JSON.parse(results).fetch("examples") }
    .map { |examples| Run.new(examples) }
).unstable_examples
```

See the full code on [GitHub](https://github.com/jkisor/flaky_finder)
