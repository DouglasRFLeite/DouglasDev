---
title: "Every Back-end Developer Must Master their ORM"
date: 2025-02-26
weight: 994
# aliases: ["/first"]
tags: ["ORM", "Database", "Java", "Hibernate", "Back-end Development"]
author: "Douglas"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Why understanding your Data Mapper matters so much for Back-end Developers"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "images/cool-setup-004.jpeg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunetly not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

No experience is ever completely exclusive.

So, even though I believe I belong to a minority of Back-end developers that do more stuff directly on the database than with the ORM, I will still share my experience with you all. Just so that it doesn't happen to anyone else.

## What's an ORM?

For those who might be new to the term, **ORM (Object-Relational Mapping)** is a technique or technology that allows developers to map application objects to relational database tables.

In simple terms, it means you can work with database data using the familiar constructs of your programming language, without writing tons of SQL. The benefits are clear: less boilerplate, easier maintenance, and a more intuitive connection between your code and your data.

That means code that is faster, cleaner and easier to mantain. Sounds like any Back-end developers dream doesn't it? That's why you should know it inside-out.

## Embracing ORM Advantages

When I first started using ORM frameworks, I thought it was just a way for me have database entities present on the code somehow.

That's probably because I was pretty familiar with SQL, but at first I even wrote native-language queries on my Repositories and DAOs instead of just relying on the ORM to build them.

I used Flyway Migrations to basically create everything trough SQL and Hibernate (the ORM) was just there to validate if everything matched. So many lost oportunities.

## Database Agnostic

Perhaps the main quality of the ORM is making your code **Database Agnostic**. That means that if you develop everything using the ORMs and don't ever mess with native SQL, you can switch between DB providers with ease; (not taking data migration in consideration here...)
