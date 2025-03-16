---
title: "Tutorial - Spring Boot & AWS S3 for Image Storing"
date: 2025-03-16
weight: 999
tags: ["Java", "Spring", "Spring Boot", "AWS", "Cloud", "S3"]
author: "Douglas"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A Tutorial of how I created a Spring Boot Application to store and retrieve images from an AWS S3 Bucket"
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
    image: "images/cool-setup-007.jpeg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunetly not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

So... my wife was busy on a Sunday afternoon. Do you know what that means? Yeah... It means I'm going to code my a*s off. 

I've been wanting to start playing with AWS for sometime now, and this week I thought of an interesting use for AWS S3 at work and wanted to test it out. 

The idea is to store and retrieve images on S3 using a Spring Boot Application.

## What is AWS S3?

I really don't want to get into too much detail here, the purpose of this article is to showcase this project and give you a step-by-step tutorial of how to replicate it yourself. 

But S3 is one of many AWS's Cloud Storage services, this one made specially to store simple files, like an actual File System. In this tutorial, I'm storing files on the root, so I really won't get into much more detail than that. But you can check some of the references if you want more information.

**Disclaimer!** I've just mentioned I'm new to Cloud and AWS, so pardon me if I said anything stupid. 

## What is Spring Boot?

Again, I don't want to get into too much detail, and I assume that if you're here you've probably heard of Spring. 

But, again, Spring Boot is a Java Framework (let's not get into so many details folks) that quickens the initialization of complex Java Applications and their growth. Here, specifically, I only use Spring Core (obviously) and Spring Web for the API.

## Prerequisites

If you want to follow me through every step of this project, you are probably going to need:
 * Java 17+ (installed on you machine or environment)
 * Visual Studio Code (to be referenced as VSCode) or any other IDE
 * Internet Connection

That's about it. Spring is a Java dependecy so you need not to download it.

## Step 1 - Creating you AWS S3 Bucket

### 1.1 - Create your account

First of all, you should access the [**AWS Console**](https://aws.amazon.com/console/) and create a free account. 

I'm not going into detail on this step, but it does take sometime so do it carefully. I created a free acount but if you're Elon Musk's son you may want to go ahead and create a paid one. 

### 1.2 - Create your S3 Bucket

After successfully Login-in, you should be redirected to the actual AWS Console. 

It should look like this if this is your first time on it:

![AWS Console](images/spring-aws-s3/aws-console.png)