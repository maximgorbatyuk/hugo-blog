---
layout: post
title: "How to Bring Pet Projects to Market"
category: development
tags: [pet-projects, development, original]
date: "2026-02-20"
description: "Pet projects often die because no one uses them, and no one uses them because they never reach release. In this article, I explain why that happens and what to do about it."
cover:
  image: "/images/blog/development/2026-02-19/how-to-build-pet-projects-cover.png"
---

![](/images/blog/development/2026-02-19/how-to-build-pet-projects-cover.png)

Many of us dreamed of building our own game when we entered this profession. Over time, that dream often changes when we realize how much math and effort games require. That happened to me too, but the core dream stayed the same: I wanted a project where I could test ideas in architecture and development.

I am sure I am not the only one. That is why developers start pet projects. The problem is that many projects do not survive long enough to be released. Let us break down why.

## Reasons

### 1. Not enough time

Many people say they do not have enough time. Limited time has always been a problem, and it always will be. The question is almost philosophical: "Where do I get more time?"

There is a great book by Maxim Dorofeev, *Jedi Techniques*, about this topic. In short, to keep up with everything, you need to save not time, but mental fuel. Mental fuel is what you need for complex, non-trivial tasks. The book explains what it is and how to manage it.

The practical advice is simple: work on your project in small, consistent steps. Set a goal to spend at least N hours per week on your pet project. Delegate routine tasks to AI agents so you do not burn out on boilerplate. Think through your architecture, write clear tasks for the agent, and learn how to evaluate the results.

### 2. It is unclear how to bring it to production

Many developers rush into coding but do not think about distribution. As a result, the only people who ever see the project are visitors to their GitHub profile. What should you do instead? Set up your release pipeline first.

As soon as you have a Hello World version, set up release and deployment. For a simple website, GitHub Pages works well. If the site is more complex, deploy to DigitalOcean or another platform. If it is a mobile app, set up publishing to TestFlight. If you are building desktop apps, study release and update tools early, before features pile up.

Why does this matter? You solve version and environment issues from day one, including different secrets and configuration variables:

- Configure the pipeline so secrets are injected automatically, and protect your site from leaking them.
- Plan how to avoid polluting analytics with localhost or local Xcode runs.
- Configure your server, DNS, and Let's Encrypt.

Even if you do not know all these processes yet, you will learn a lot. This knowledge stays with you even if the pet project never ships. Knowledge is never wasted. One hour spent on deployment setup at the beginning can save a week of fixes at the end.

### 3. It takes many resources

I have a pet project, [techinterview.space](https://techinterview.space), deployed on DigitalOcean. Infrastructure alone costs a bit more than $50 per month: VPS, PostgreSQL, and S3. On top of that, I paid for design services: a mascot and Telegram stickers cost nearly $600. I did all this to grow my personal brand by giving users a useful service and useful statistics.

Not everyone is ready to spend that much. I recommend starting with the cheapest or free options: GitHub Pages for deployment, Resend's free plan (100 emails per day), Auth0's free tier for authentication, and so on.

If your pet project is a mobile or desktop app, it may be easier. A released app can live on its own with less constant attention, lower cost, and less effort than a website. Besides techinterview, I currently have two iOS apps (as of February 19, 2026). Their feature sets are stable, and they are no longer in active development.

Another point many people miss at the start is shutdown conditions. Ask yourself: "How will I know it is time to stop, and what will I do then?" For mobile apps, I can simply stop paying for Apple Developer and stop shipping new versions. For techinterview, the plan is:

1. Announce on the homepage and key traffic pages that the project is nearing completion.
2. Turn off the backend so new questionnaires and company reviews are no longer accepted.
3. Convert the site to static pages. Save all collected data up to the shutdown date in JSON files and render it directly.

I plan to keep the site in that static state for another six months. After that, the project will remain as a dataset in a GitHub repository.

By designing this plan, I give myself permission to stop spending time, effort, and money when needed. Closing a project is normal. Define your completion criteria and action plan in advance, and the question "What if it does not work out?" becomes much easier to answer.

Define your budget, your free starter stack, and your shutdown scenario upfront. This keeps a pet project manageable even with limited resources.

### 4. The product is not built for yourself

The longest-living project is usually the one built for yourself. Your pet project should solve your own pain first. Then it will always have at least one user: you.

Approach your problem from a product perspective and ask:

1. Are the patterns I use really the most effective?
2. Am I treating symptoms instead of solving the root cause?
3. How many other ways could solve this task, and are any of them better?

These questions help you build more effective automation and encourage honest reflection. It is important to ask yourself the right questions, even when they are uncomfortable. There is a method called "for what": ask yourself "for what?" five times, and you will often reach the root cause.

Building for yourself helps you look at your tasks and solution patterns from a new angle and spot opportunities for automation.

For example, I have two mobile apps: one for tracking EV charging and one for travel planning. Here is the background behind each app.

#### EV Charge Tracker

I like tracking things. When I started driving in 2014, I began recording fuel fill-ups and expenses in an app. In 2025, I switched to an electric car and noticed there were not many apps for EV expense tracking. Instead of fuel fill-ups, you now track charging sessions and consumed kilowatt-hours.

There were few free options, and even those lacked features I needed. That is why I built my own app, [EV Charge Tracker](https://mgorbatyuk.dev/ev-charging-tracker). Based on analytics, I am not the only person with this need.

#### Journal Wallet

In summer 2025, I planned a Europe trip for three people. I had to collect visa documents, buy flights, and book hotels in different cities. The data ended up scattered everywhere: some in Notion, some in email, and some as photos on my phone.

That gave me the idea to build a travel planning app, [Journal Wallet](https://mgorbatyuk.dev/journal-wallet), so I could keep everything for one trip in one place.

While building both apps, I studied other products, analyzed their strengths and weaknesses, and reviewed my own needs: what I need, what existing solutions miss, and what I can live without. I had strong motivation to release these apps because I needed them for myself, not for someone else. Until I sell my electric car and stop traveling, I will keep using them.

### 5. Motivation fades when you do not know who else uses the product

Even if you build for yourself first, it still feels great to know others use your product. The solution is simple: set up analytics early.

This matters for two reasons:

- You need separate environments from the beginning so localhost activity does not pollute analytics.
- It is easier to add tracking gradually while building features than to add everything later in one big batch.

If you have never integrated analytics before, Google Analytics already gives useful data out of the box: active users, session duration, and events. For my projects, here are the numbers for the last 30 days (as of February 19, 2026):

| Project | Users | Average session |
|---|---|---|
| Techinterview.space | 1.9K | 4 minutes 31 seconds |
| EV Charge Tracker | 107 | 2 minutes 6 seconds |
| Journal Wallet | 63 | 1 minute 59 seconds |

A nice surprise was direct feedback from users of my EV app. People asked me to add their currency, reported bugs, and requested features because their charging habits differed from mine.

The funniest case: someone from Turkey sent me a Turkish localization file on Telegram with no context. They found the app's GitHub repository, downloaded the existing localization file, translated it, and sent it back. That is how Turkish appeared in my app.

Give users an easy way to contact you. On my website and in my mobile apps, I leave a link to my Telegram account.

Connect analytics and provide a convenient feedback channel. Metrics support motivation, and feedback speeds up product growth.

## Conclusion

Growing a pet project is expensive and not always rewarding, so do not start with large, costly ideas. Start with small needs. If your ideal task tracker does not exist, build your own.

Learn on small projects first, and you will gain the skills to build bigger ones.

In the era of vibe coding, every developer can publish a project on GitHub. But not everyone can build a product, set up testing pipelines, configure static analysis, write solid documentation, and package a release for market. Remember: an app on your friends' phones is better than a repository on GitHub.
