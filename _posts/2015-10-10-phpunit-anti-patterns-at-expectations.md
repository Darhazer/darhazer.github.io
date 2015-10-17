---
layout: post
title: "PHPUnit Anti Patterns: at() expectations"
description: ""
category: phpunit 
tags: []
---
{% include JB/setup %}

# PHPUnit Anti-Patterns: at() expectations

Too often I see `at()` expectation used in test cases. Usually it's not expectation at all - it is used just to provide different return values on the same method, when called with different parameters. Fortunatelly this can be easily refactored with `returnValueMap` - it makes clear what is the return value for what input and best of all it doesn't depend on the order of the execution. `returnValueMap` has its own quircks, I have to admit, but it's easily to extend it and overcome them. I did it (see my [pull request](https://github.com/sebastianbergmann/phpunit-mock-objects/pull/254)), but even if you don't want to extend phpunit-mock-objects, there's other methods that can solve the problem with returning different values for different calls - see `onConsecutiveCalls` and `returnCallback`. 

So what are the problem with `at()` expectation, it's (seemingly) legit use and the proposed solution?

When `at` is used the test code is actually coupled to the implementation. Add inanother call to the same object somewhere in between and your test fails for no valid reason. Moreover it seems that a lof of code exists just to support the `at` behaviour and as a side effect you can't override stub return value (I'm not 100% sure about this, but you can read my RFC [here](https://github.com/sebastianbergmann/phpunit-mock-objects/issues/260)). So you should never use `at` when you don't care about the order of execution, but just that different things should happen on each call to the same method. For those better methods exists. Even if you want to vseerify the parameters send at each call, you can use `withConsecutive`.

So why `at` expectation exists in the first place? It's possible that historycally it existed to support expectations/stubs on consecutive calls (`onConsecutive` exists since version 3.0, while `withConsecutive` was added in 4.1). Nowadays the only valid use would be verifying the order of execution - if one call were made before another. Whether this is something a test should verify is another story - for the purpose of this post I'll take it for granted that such verification is needed. The problem with `at` is that it doesn't verify just whether the call was made before another one - it verifies exactly the position of the call (whether it was the 3rd call to a method of the object given or 4th). You can make your test more reliable if you just verify the relative order of the two methods - for which phpmock-objects doesn't have a good syntax yet. But you can hack `withCallback` expectation and use variables to track whether the method you are interested was already called or not. I would suggest more explicit API for such verification - probably `before` and `after` methods (just like the `with` method that verifies the arguments, `before` and `after` would verify the order of execution. In mockery there is `ordered` method that does the job.

One argument agains using `at` expectation even for verifying the order of execution is that if you need to verify the order of calling the methods of different objects, `at` doesn't do the job. You will have to hack `withCallback` anyway.

I suspect that the only reason for supporting the `at` expectation is backwards compatability. As stated in the beginning of the post, I've seen a lot of code that uses it. So the first step for removing it - and perhaps developing a specific API for ordered verifications - is to stop using it at all. 
