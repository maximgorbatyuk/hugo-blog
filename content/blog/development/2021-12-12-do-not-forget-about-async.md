---
layout: post
title: Do not forget about Async postfix
category: development
tags: [.net, opinion]
date: "2021-12-12"
---

There is a recommendation: to write a postfix "...Async" in the end of async methods in .NET. I always follow this rule, and also I recommend everyone to do this as well as I do. In this article, I want to share my thoughts why this is important rule when you dela with asynchronous programming.

## You can call the async method withour await and it will not throw any error

In my opinion, it is a problem that .NET allows us to call asynchronous method without `await` keyword and without awaiting a result as well. and no compile error will be thrown. Also, runtime error will not be thrown as well. Therefore, you should mark async methods with "...Async" postfix to help youself and your teammates to not miss possible errors to the production.

## You read the code not only with IDE

That's suprise for junior developers but members of their team may read the code not only with IDE. Github, git merge tool, etc are used to do it as well. There is no  intellisense or warning from your programming language. Then, if you left async method without Async postfix and passed it to code review, your colleague might be tired and miss the method to the production. Of course, it would be his mistake, but I hope you work for the project success, not only for moving Jira tickets from the "Development" column to "Code review" and that's all what you do on the project. Just help your senior - do not do code reviews harder than they should be.

## It obviously says that the method is asynchronous

It might sound like obvious advice, but it is important to write explicit code which is not ambiguous. The code is the thing that you should not guess about. The code is something that is supposed to be explicit and to show its intention. In my opinion, the async approach is one of the intenion. 

## Conclusion

If you don't mind about leaving "..Async" posfix in the end of the async methods you write, please, change your mind. That's makes code review session easier, and your code more transparent and clean.
