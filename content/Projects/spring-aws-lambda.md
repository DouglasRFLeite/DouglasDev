---
title: "Tutorial - Spring Cloud Function & AWS Lambda"
date: 2025-05-07
weight: 997
tags: ["Java", "Spring", "Spring Boot", "Spring Cloud Function", "AWS", "Cloud", "Lambda"]
author: "Douglas"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A tutorial on how to create a simple serverless function and deploy it to AWS Lambda"
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
    image: "images/cool-setup-001.jpg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunately not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

This is supposed to be the **Part 1** of a series of smaller tutorials I'll publish, all related to the same project. I'll get into more detail about the project specifics when I make an article about it, but it should include **AWS Lambda, AWS DynamoDB and a React front-end**, so stay tuned.

In this article I'll explain how to create and deploy a very very simple serverless function with Spring Cloud Function and AWS Lambda. I won't give much effort in thinking of a real world application for this because it will integrate my project later, so let us just create a function that receives a name and returns Hello to that name.

## What and Why...

### ... AWS Lambda?

I've been troubled for a few years now by the problem of "_Where can I deploy my pet projects?_".

I mean... who doesn't? I've heard Heroku had generous free tier, but I didn't get the change to enjoy it. That's why most of my projects are 99% frontend (because I can deploy for free on Vercel and Netflify) and use Firebase as the Backend.

But come on, I'm a Backend Java Engineer. I needed to host my very intricate, optimized and crazy backend somewhere.

That's where AWS Lambda comes in place.

As a serverless service, it will only charge me by request and after the million requests a month free-tier. I don't think most projects will ever get 1,000,000 requests, imagine in a single month. That's too ideal to pass.

Besides, AWS Lambda has become an industry standard, so I would have to learn it sooner or later.

### A bit of theory behind it

AWS Lambda, as well as other serverless function services, work differently from dedicated servers. You provide the code (or the docker image) to AWS and it will store it somewhere inside it's magic servers.

When triggered by something (you should configure) AWS will start your code up and run it with the provided input, if any.

There are a couple problems with this:

- Cold Starts means that, if it's been a while since your last Lambda activation, there will be some initialization time spent before the result comes.
- No Dedicated Server means no traditional Database, you will either have to use a Cloud Database or store in another server somewhere.
- Billing is calculated per request, wich is awesome if you have little requests, but can be pretty troublesome otherwise.

So reason over this problems and define if, as it is with me, AWS Lambda works for your project deployment.

### ... Spring Cloud Function?

It's come to my understanding that I can simply deploy a Rest Server on AWS Lambda and it will work fine. Actually, I have deploied dockerized Python+Flask Rest servers on Google Cloud Run before, but that theoretically takes too long to start.

I do understand that, with the advent of the Snapstart option on AWS, it wouldn't be such a big problem.

But really, I just like learning new stuff. So with Cloud Function we go.

So let us get our hands dirty!

## Step 1 - Creating your Spring Cloud Function project

## Step 2 - Packaging your Spring Cloud Function project

## Step 3 - Deploying your Function to AWS Lambda

## Step 4 - Testing your AWS Lambda Function

## Conclusion

Next steps... link Lambda to API Gateway, multiple functions in a project
