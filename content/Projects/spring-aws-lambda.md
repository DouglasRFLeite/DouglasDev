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

In this article I'll explain how to create and deploy a very simple serverless function with Spring Cloud Function and AWS Lambda. I won't give much effort in thinking of a real-world application for this because it will integrate with my project later, so let us just create a function that receives a name and returns Hello to that name.

## What and Why...

### ... AWS Lambda?

I've been troubled for a few years now by the problem of "_Where can I deploy my pet projects?_".

I mean... who doesn't? I've heard Heroku had a generous free tier, but I didn't get the chance to enjoy it. That's why most of my projects are 99% frontend (because I can deploy for free on Vercel and Netlify) and use Firebase as the backend.

But come on, I'm a backend Java Engineer. I needed to host my very intricate, optimized and crazy backend somewhere.

That's where AWS Lambda comes in.

As a serverless service, it will only charge me by request and after the first million requests a month free-tier. I don't think most projects will ever get 1,000,000 requests, imagine reaching that in a single month. That's too good to pass up.

Besides, AWS Lambda has become an industry standard, so I would have to learn it sooner or later.

### A bit of theory behind it

AWS Lambda, as well as other serverless function services, work differently from dedicated servers. You provide the code (or the docker image) to AWS and it will store it somewhere inside its magic servers.

When triggered by a teigger (which you configure) AWS will start your code up and run it with the provided input, if any.

There are a couple problems with this:

- Cold Starts means that, if it's been a while since your last Lambda activation, there will be some initialization time spent before the result comes back.
- No Dedicated Server means no traditional Database, you will either have to use a Cloud Database or store data in another server somewhere.
- Billing is calculated per request, which is awesome if you have few requests, but can be pretty troublesome otherwise.

So consider these problems and define if, as it is with me, AWS Lambda works for your project deployment.

### ... Spring Cloud Function?

It's come to my understanding that I can simply deploy a REST Server on AWS Lambda and it will work fine. Actually, I have deployed dockerized Python+Flask REST servers on Google Cloud Run before, but that theoretically takes too long to start.

I do understand that, with the advent of the Snapstart option on AWS, it wouldn't be such a big problem.

But really, I just like learning new stuff. So with Spring Cloud Function we go.

So let us get our hands dirty!

## Step 1 - Creating your Spring Cloud Function project

This is actually the easiest step in this whole tutorial.

I'll use Spring Initializr to create my project, but you can do it anyway you want:

1. Create a Maven project
2. Spring Boot Version 3.4.3
3. Java
4. Name it as you wish
5. I've used Java 17 and JAR packaging
6. You'll add dependencies: Spring Web, Function (from Spring Cloud)

You won't do it like this if you're working on a real project, but here we will add our code directly to our Application class.

```
@SpringBootApplication
public class CloudHelloApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudHelloApplication.class, args);
	}

	@Bean
	public Function<String, String> helloFunction() {
		return (name) -> "Hello, " + name;
	}

}
```

We've created a Spring Bean that creates a function. That function will receive a String as argument and return a String. The String, in our case, is the name, and the return is Hello + that name (but you got that from the code, I hope).

And that is mostly it. Codewise, you're good to go. You can even run and test your code locally if you want. You can either use curl:

```
curl -X POST -H "Content-Type: text/plain" -d "Douglas" http://localhost:8080/helloFunction
Hello, Douglas
```

Or via Postman

![Postman POST](/images/spring-aws-lambda/postman.png)

And it should work just fine.

## Step 2 - Packaging your Spring Cloud Function project

There's a reason why AWS deployment was historically a part of the DevOps instead of the backend Engineer. That's because it's mostly not a coding issue. As far as coding goes, as we've mentioned, our job is done.

But, as good Fullstack, Fullcycle, Fullmetal Alchemist Engineers, we are going all the way to deployment here. And the first step is the proper packaging of our code.

### 2.1 - The Cloud-Specific Adapter

As it is right now, our code can be packaged for deployment in any Cloud Serverless service like AWS, Azure or Google.

The first step in our packaging is to define what cloud we're going to use and add the correspondent adapter dependency to our `pom.xml` file.

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-function-adapter-aws</artifactId>
</dependency>
```

### 2.2 - Shade Packaging

Shade packaging is a process in which you create an "uber-JAR" or a fat JAR. This is a single file that contains all the classes from your project along with the class files from all its dependencies.

That process is performed by the **Apache Maven Shade Plugin**, and is necessary for serverless deployment of Java projects. Understand, then, that you should be careful with every dependency you add to your project, because it will create a JAR file that can even be too big for deployment.

Since it's done via a plugin, we will add that plugin to our build configuration on our `pom.xml` file.

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
                <skip>true</skip>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot.experimental</groupId>
                    <artifactId>spring-boot-thin-layout</artifactId>
                    <version>1.0.28.RELEASE</version>
                </dependency>
            </dependencies>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <configuration>
                <createDependencyReducedPon>false</createDependencyReducedPon>
                <shadedArtifactAttached>true</shadedArtifactAttached>
                <shadedClassifierName>aws</shadedClassifierName>
            </configuration>
        </plugin>
    </plugins>
</build>
```

You can just replace your "build" section with this code and it should work. This will provide you with the AWS-ready version and the "normal" Spring application.

### 2.3 - Maven Packaging

Now you can package your project as usual: with the `mvn clean package` terminal command (or other CLI/IDE tool you use).

In case you see a `BUILD SUCCESS` message you should also have some files inside your target folder.

![Build Files](/images/spring-aws-lambda/built-files.png)

The non-aws file is your standard Spring JAR that can be run standalone as usual. The aws file is what we're going to use to deploy our project in AWS.

## Step 3 - Deploying your Function to AWS Lambda

### 3.1 - Create your account

First of all, you should access the [**AWS Console**](https://aws.amazon.com/console/) and create a free AWS account.

I'm not going into detail on this step, but it does take some time, so do it carefully. I created a free account, but if you're Elon Musk's child, you may want to go ahead and create a paid one.

### 3.2 - Create your Lambda Function

After successfully logging in, you should be redirected to the actual AWS Console.

It should look like this if this is your first time on it:

![AWS Console](/images/spring-aws-s3/aws-console.png)

First of all, I would recommend you change the region in the upper right corner to the closest one to you. I forgot to do it the first time and had simple requests take a couple of seconds to return.

Now go to the search bar and type "Lambda" - or look for it in the menu if you have more time (or just click [here](https://sa-east-1.console.aws.amazon.com/lambda/home?region=sa-east-1#/discover)). Access the Lambda page.

![Lambda Console](/images/spring-aws-lambda/lambda-console.png)

Here you should see slightly different info than what I see, but there should be a **"Create Function"** button available. Click it.

Leave it as "Author from scratch", create any name you'd like and choose Java 17 as your runtime. The result should look like this:

![Create Function](/images/spring-aws-lambda/create-function.png)

If it's alright, you can go ahead and **Create Function**. This is what you'll be looking at next.

![Function Console](/images/spring-aws-lambda/function-console.png)

Pay attention to those 3 marked buttons, you'll click them in no time.

**(1) Upload your Project**

On the "Upload from" button, select the `.jar` option. Upload your AWS build file just like so.

![Upload Jar](/images/spring-aws-lambda/upload-jar.png)

Hit Save and wait a couple of seconds.

**(2) Add the Runtime Handler**

Hit that "Edit" button down below (number 2 from the previous screenshot) and change the Handler to:

```
org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest
```

![Handler](/images/spring-aws-lambda/handler.png)

This should be your default option every time you deploy Spring Cloud Function to AWS Lambda.

## Step 4 - Testing your AWS Lambda Function

Now you should have a fully working AWS Lambda Function. We can test it using the **(3)** button marked in that screenshot. Click that and you should see something like this at the bottom of your screen.

![Test Lambda](/images/spring-aws-lambda/test-lambda.png)

You should, as I have, change the event data on the input at the bottom. And then just click Test.

In no time you should see a message right above that your test is executing and waiting for response. It can take a while because the whole project is starting up, but in a couple of seconds your response should arrive and look something like this:

![Test Result](/images/spring-aws-lambda/test-result.png)

As you can see, it took almost 4 seconds to start up. That can be a serious problem in some scenarios, but not in mine and on a lot of other cases as well.

Also, if you fire another Test not too long after, it should still be running and you won't have to wait for that Init time.

## Conclusion

That's it!

Now you can call yourself a Spring Cloud Function, Serverless and AWS Lambda specialist (I will at least).

As with my other current tutorials, this is an article based on my first few interactions with Spring Cloud Function and AWS Lambda, so I appreciate and suggestions via my [**LinkedIn**](https://www.linkedin.com/in/douglas-rocha-leite) profile.

This is a very basic, quick and easy to follow tutorial, but I will go into more advanced topics and integrations in further articles.

Next steps include exposing this Lambda via Function URL (or API Gateway, I will decide it as the project moves forward), having multiple functions in a single project, connection to DynamoDB with a function and, finally, accessing the DynamoDB data via Lambda on a React front-end app. Stay tuned if any of that interests you.

### References

1. Spring Cloud Function Documentation
   https://docs.spring.io/spring-cloud-function/docs/current/reference/html/
   The official documentation for Spring Cloud Function.

2. AWS Lambda Documentation
   https://docs.aws.amazon.com/lambda/latest/dg/
   The official documentation for AWS Lambda.

3. Apache Maven Shade Plugin Documentation
   https://maven.apache.org/plugins/maven-shade-plugin/
   The official documentation for the Maven Shade Plugin.

4. Serverless Spring: Deploy serverless functions to any platform using Spring Cloud Function
   https://www.youtube.com/watch?v=gj1DDymw5iY&ab_channel=DanVega
   This is a tutorial on Youtube that helped me get this thing up and running for the first time.
