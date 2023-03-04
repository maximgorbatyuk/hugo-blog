---
layout: post
title: Антипаттерны тестирования программ с примерами
category: development
tags: [.net, elegant_objects]
date: "2023-03-04"
description: ""
draft: true
---

0. Странник

Тест, который хоть и тестирует методы одного класса, но по факту проверяет функционал другого класса, нахордящегося внутри тестируемого.

1. Один тест на каждый метод

Иметь по одному тесту на каждый метод рабочего кода - это хорошее начало, но так функционал системы и пограничные кейсы не проверишь. Останавливаться на этом не стоит.

2. Анальная пробка

Тестирование приватных полей, методов. тест знает слишком много о тестируемом классе и лезет не в свое дело.

3. Happy path

Тесты проверяют только прохождение "частливого пути", не затрагивая пограничные кейсы и ошибочные сценарии

4. Слоупок

Тесты, которые воспроизводятся очень долго. Например, после старта тестов можно смело пойти заварить себе чай или пойти в курилку

5. Гигант

Тестфайл содержит в себе тысячи строк кода. Такое может быть, когда тестируемый код


Cuckoo1 (aka Stranger3). This is a test method that sits in the same unit test but doesn’t really belong there.

Test-per-Method1. Although a one-to-one relationship between test and production classes is a reasonable starting point, a one-to-one relationship between test and production method is almost always a bad idea.

Anal Probe2. A test that has to use unhealthy ways to perform its task, such as reading private fields using reflection.

Conjoined Twins2. Tests that are called unit tests but are really integration tests since there is no isolation between the system-under-test and the tests.

Happy Path. The tests stay on happy paths (i.e. expected results, e.g. 18 years old) without testing for boundaries and exceptions (e.g. -2 years old).

Slow Poke3. A unit test that runs incredibly slow. When developers kick it off, they have time to go to the bathroom, grab a smoke, or worse, kick the test off before they go home at the end of the day.

Giant3. A unit test that, although it is validly testing the object under test, can span thousands of lines and contain many many test cases. This can be an indicator that the system-under-test is a God Object.

Mockery3. Sometimes mocking can be good, and handy. But sometimes developers can lose themselves in their effort to mock out what isn’t being tested. In this case, a unit test contains so many mocks, stubs, and/or fakes that the system under test isn’t even being tested at all, instead data returned from mocks is what is being tested.

Inspector3. A unit test that violates encapsulation in an effort to achieve 100% code coverage, but knows so much about what is going on in the object that any attempt to refactor will break the existing test and require any change to be reflected in the unit test.

Generous Leftovers3 (aka Chain Gang, Wet Floor). An instance where one unit test creates data that is persisted somewhere, and another test reuses the data for its own devious purposes. If the “generator” is ran afterward, or not at all, the test using that data will outright fail.

Local Hero3 (aka Hidden Dependency, Operating System Evangelist, Wait and See, Environmental Vandal). A test case that is dependent on something specific to the development environment it was written on, in order to run. The result is that the test passes on development boxes, but fails when someone attempts to run it elsewhere.

Nitpicker3. A unit test which compares a complete output when it’s really only interested in small parts of it, so the test has to continually be kept in line with otherwise unimportant details.

Secret Catcher3. A test that at first glance appears to be doing no testing due to the absence of assertions, but as they say, “the devil is in the details.” The test is really relying on an exception to be thrown when a mishap occurs, and is expecting the testing framework to capture the exception and report it to the user as a failure.

Dodger3. A unit test which has lots of tests for minor (and presumably easy to test) side effects, but never tests the core desired behavior. Sometimes you may find this in database access related tests, where a method is called, then the test selects from the database and runs assertions against the result.

Loudmouth3. A unit test (or test suite) that clutters up the console with diagnostic messages, logging, and other miscellaneous chatter, even when tests are passing.

Greedy Catcher3. A unit test which catches exceptions and swallows the stack trace, sometimes replacing it with a less informative failure message, but sometimes even just logging (cf. Loudmouth) and letting the test pass.

Sequencer3. A unit test that depends on items in an unordered list appearing in the same order during assertions.

Enumerator3 (aka Test With No Name). Unit tests where each test case method name is only an enumeration, e.g. test1, test2, test3. As a result, the intention of the test case is unclear, and the only way to be sure is to read the test case code and pray for clarity.

Free Ride3 (aka Piggyback). Rather than write a new test case method to test another feature or functionality, a new assertion rides along in an existing test case.

Excessive Setup3 (aka Mother Hen). A test that requires a lot of work to set up in order to even begin testing. Sometimes several hundred lines of code are used to setup the environment for one test, with several objects involved, which can make it difficult to really ascertain what is being tested due to the “noise” of all of the setup.

Line hitter. At first glance, the tests cover everything and code coverage tools confirm it with 100%, but in reality the tests merely hit the code, without doing any output analysis.

Forty-Foot Pole Test (see). Afraid of getting too close to the class they are trying to test, these tests act at a distance, separated by countless layers of abstraction and thousands of lines of code from the logic they are checking.

The Liar4 (aka Evergreen Tests, Success Against All Odds3). A test doesn’t validate any behaviour and passes in every scenario. Any new bug introduced in the code will never be discovered by this test. It was probably created after the implementation was finished, so the author of this test couldn’t know whether the test actually tests something.

https://stackoverflow.com/questions/333682/unit-testing-anti-patterns-catalogue
https://archive.is/3acB#selection-119.0-119.17
https://www.yegor256.com/2018/12/11/unit-testing-anti-patterns.html
https://blog.codepipes.com/testing/software-testing-antipatterns.html