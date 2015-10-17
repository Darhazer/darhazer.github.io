---
layout: post
title: "PHPUnit Anti Patterns: expectations on getters"
description: ""
category: phpunit
tags: []
---
{% include JB/setup %}
# PHPUnit Anti-Patterns: expectations on getters

Often I see code like this:

`$mock->expects($this->once())->method('getSomethig')->willReturn('someValue');`

Sometimes `$this->any()` is used instead of `once`, but I'll discuss this approach later. What bothers me with `once` is: are we really care that the method is called exactly one time? Usually getters don't have any side effects and it's safe to call them multiple times. What if in the same method that we test the same getter is called second time - to log the value (caching the value of the call is an implementation detail - it may or may not happen). In case it's important the method to be called just once - because it performs slow calculation or retrieves data from external service, there should be only one test that verifies this. All other tests that stub the method just so the code works should not provide such expectation. If the method implementation is changed - with added caching inside the method for example, the `once` expectation may not be valid any more. In such case there should be only one test that needs to be removed and all other test should continue to work.

There may be other reason to have `once` expectation - to verify that the method is called, e.g. the data was actually retrieved via this method. The need for such a verification is quesionable, but if there is such a verification, I suggest using `atLeastOnce` - which will verify that the method was called but will allow multiple calls at the same time. Still, if there is verification that the data is retrieved this way, it should be single test - in case the system changes and data is no longer retrieved from this object, there should be only one test that needs to be changed.

Which leads me to the last point - should you even stub getters? After all if you change a collaborator of the class, all stubs of the getter will no longer be of any use. You will still have to modify all tests that rely on some data to be retrieved - even if there are no expectations towards the getter, the return value provided by the test is a precondition for what is verified later in the test. So I see `$mock->expects($this->any())->method('getSomethig')->willReturn('someValue');` as anti-pattern as well. Such code that provides `setup` of the code should be decoupled of the actual implementation - in a way that when you change the collaborator, you will have to change only one support object in the tests, and not the test cases. Usually I refactor this to builders. This allows making the test more readable by expressing the meaning of the data, and not how it is retrieved, allows easy reuse of the setup code and - as mentioned above - allows you to modify single object instead of all of the test cases that use it.
