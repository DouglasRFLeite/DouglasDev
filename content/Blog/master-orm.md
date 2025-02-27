---
title: "Every Back-end Developer Must Master their ORM"
date: 2025-02-26
weight: 994
# aliases: ["/first"]
tags: ["ORM", "Database", "Java", "Hibernate", "Back-end Development"]
author: "Douglas"
showToc: true
TocOpen: false
draft: false
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
    image: "images/cool-setup-004.jpg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunetly not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

**No experience is ever completely exclusive.**

Even though I believe I belong to a minority of back-end developers who work more directly with the database than with the ORM, I’ll still share my experience so that it doesn’t happen to anyone else.

---

## What's an ORM?

For those who might be new to the term, **ORM (Object-Relational Mapping)** is a technique or technology that allows developers to map application objects to relational database tables.

In simple terms, it means you can work with database data using the familiar constructs of your programming language, without writing tons of SQL. The benefits are clear: less boilerplate, easier maintenance, and a more intuitive connection between your code and your data.

That means code that is faster, cleaner and easier to mantain. Sounds like any Back-end developers dream doesn't it? That's why you should know it inside-out.

## Embracing ORM Advantages

When I first started using ORM frameworks, I thought it was just a way for me to have database entities represented on the code somehow.

Having a solid background in SQL, I even wrote native queries in my Repositories and DAOs instead of relying more on the ORM.

I've used Flyway Migrations to basically create everything through SQL. Hibernate (the ORM I use with Java) was just there to validate if everything matched.

...So many lost oportunities.

## Database Agnostic

Perhaps the main quality of the ORM is making your code **Database Agnostic**. That means that if you develop everything using the ORMs and don't ever mess with native SQL, you can switch between DB providers with ease (not considering data migration here).

**And that was what made me look diferently at ORMs for the first time** when I had to switch from Microsoft SQL Server to PostgresSQL in a huge project. I did that in a single-week Sprint thanks to Hibernate!

- All queries already using HQL (Hibernate Query Language) remained unchanged.
- Most queries that weren’t were easy to switch to HQL.
- I could literally let the Java structure of the project form the basis for the new database schema, only needing to adjust the indexes manually

At this point, there are literraly 5 lines of code in this huge project that I need to change if I want, say, switch to MySQL.

That is the power of knowing your ORMs power.

## Encapsulation of the Database

My second awesome experience with the ORM happened just this week.

I needed to create a new feature involving two tables with a many-to-many relationship. AAs usual, I first created the two tables via a migration—including the table to map that relationship (let’s call it `fruit_juice`). My default process is to create the migration first, then start building the corresponding Java objects.

Result: when I got a hold of me, I had created a `FruitJuice.java` file to map every entry of the relations table. I did that because I had already done it that way in the code at another point.

Thank God I had studied a bit more about the ORM after my first awesome experience with it. **It just had a far better way of getting it done!** By just using Hibenate's annotations—like `@ManyToMany` and `@JoinColumn`—I could let the ORM manage the relationship without having to maintain an extra class in my code.

**It felt like magic.**

## More Benefits of Fully Embracing the ORM

Other than what I have already talked about, these are some more benefits of using your ORM at it's full potential:

- **Cleaner Code**: Write less boilerplate and more meaningful business logic.
- **Faster Development**: Spend less time on manual database tasks and more on solving real problems.
- **Enhanced Readability**: Maintain a clear, object-oriented structure that’s easy to understand and modify.

## It Ain't All Flowers

I should still note that, like any tool, **an ORM isn't without pitfalls**. Over-reliance on ORM abstractions can be more dangerous than an under-reliance on it. Afer all, all the listed benefits are good, but there's nothing that could not be done directly on the database.

My point on letting it handle relationships was done like that because the tables are pretty small (supposed to be less than 100 rows each). If I was talking about my 400,000-thousand-registers-a-day table I would probably still rely directly on the database to make things as performatic as possible.

It's really important to periodically review the SQL generated by your ORM and ensure that you're not missing out on crucial database optimizations—like proper indexing.

---

## Final Thoughts

Mastering your ORM is not just about avoiding headaches with SQL—it's about empowering you to build applications more efficiently and elegantly. By embracing the ORM fully, you can focus on what truly matters: delivering great software.

For more insights, software development and productivity content, follow up for more posts here in my blog and let us connect in [**LinkedIn!**](https://www.linkedin.com/in/douglas-rocha-leite)
