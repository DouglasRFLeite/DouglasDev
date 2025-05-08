---
title: "[Tutorial] Using Multiple Functions with Spring Cloud Function and AWS Lambda"
date: 2025-05-08
weight: 996
tags: ["Java", "Spring", "Spring Boot", "Spring Cloud Function", "AWS", "Cloud", "Lambda"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A tutorial on using multiple functions in a single AWS Lambda deployment with Spring Cloud Function"
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
    image: "images/cool-setup-002.jpeg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunately not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

This is the **Part 2** of a series of smaller tutorials I'm publishing, all related to the same project. I'll get into more detail about the project specifics when I make an article about it, but it should include **AWS Lambda, AWS DynamoDB and a React front-end**, so stay tuned.

This is supposed to be a pretty small tutorial, expanding on the content from the [**Part 1**](/projects/spring-aws-lambda) of this series that teaches the basic means of creating, deploying and testing a Spring Cloud Function project on AWS Lambda.

The main focus of this article is to explain how you can have multiple functions in a single project and access them via **Function URLs**. Sound simple, but it took me some research to figure it out.

To test it, we're expanding our hello function to be a hello-bye function.

### What and Why...

### ... AWS Lambda and Spring Cloud Function

Well, we're not doing that again. Refer to [**Part 1**](/projects/spring-aws-lambda) for more on this topic.

### ... Multiple Functions?

There are a few possible architectures for more complex serverless applications. A common one is the idea to separate every domain, or even function, into a single project and deploy them as separate Lambdas (or other serverless functions from a different provider). This is a very Microservice oriented thinking, and it's not wrong.

A problem with this architecture is the need to replicate a lot of code, business logic and, in our case, database connectivity code.

_What do you mean with "in our case"? This code doesn't connect to the any database..._

Not yet, little Kid Flash. But it will.

And yeah, even in those cases, it may be worth to separate the project into multiple services and deploy them as multiple functions.

Let's say you have 5 domains in your project: A, B, C, D and E. Let's say they communicate in pairs,so Function A would need some B code, function B would need some C code, and so on. You would have 5 functions, each with code for almost 2 whole domains. But that's still far smaller functions than a single function with code from all 5 domains. Then it may be worth it.

In our case, there are only 2 domains, 2 kinds of data and a single database. We would need to copy almost 90% of the code just to separate the entrypoints. It doesn't make sense. **So, multiple functions it is.**

### ... Function URLs?

A Lambda Function URL is a direct, dedicated HTTPS endpoint for a single Lambda function. It's a simpler way to invoke your Lambda function over HTTP without needing to configure an API Gateway.

Is it better than the API Gateway? If with better you mean more powerfull and complete, definetly not. The AWS API Gateway is a fully managed service that acts as a unified "front door" for all of your backend services, including Lambda Functions. It can be so much more than just an endpoint.

Now, if with better you mean **easier, simpler and thus more suited to this specific case**, than yeah... it is.

## Step 1 - Creating and Deploying Spring Cloud Function project with multiple Functions

### 1.1 - Stealing from Part 1

I won't get into much detail about how to create a Spring Cloud Function project. Actually, I'm going to mostly copy each step from [**Part 1**](/projects/spring-aws-lambda) and change the project name to cloud-hello-bye.

This is what you should have if you did the same:

```
@SpringBootApplication
public class CloudHelloByeApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudHelloByeApplication.class, args);
	}

	@Bean
	public Function<String, String> helloFunction() {
		return (name) -> "Hello, " + name;
	}
}
```

If you want, you should already be able to run and test your project via HTTP requests (using curl, Postman or whatever). You can even deploy this to AWS as a Lambda and see if it works. But that's the topic of the Part 1, so let's move on.

### 1.2 - Creating the Bye function

Now we're expanding our project by adding another function, right here on our Application class (don't do that in a real project, for God sake).

```
@SpringBootApplication
public class CloudHelloByeApplication {

	public static void main(String[] args) {
		SpringApplication.run(CloudHelloByeApplication.class, args);
	}

	@Bean
	public Function<String, String> helloFunction() {
		return (name) -> "Hello, " + name;
	}

	@Bean
	public Function<String, String> byeFunction() {
		return (name) -> "Bye bye, " + name;
	}

}
```

There you go! Now you have a project with two functions. You can test it now, changing the url to see them working separatly.

```
curl -X POST -H "Content-Type: text/plain" -d "Douglas" http://localhost:8080/helloFunction
Hello, Douglas

curl -X POST -H "Content-Type: text/plain" -d "Douglas" http://localhost:8080/byeFunction
Bye bye, Douglas
```

### 1.3 - Lambda Deployment

As before, please refer to [**Part 1**](/projects/spring-aws-lambda) for this content. It's the exact same steps. Once you get to Testing, come back here.

If you've tried testing it as we did in Part 1, you probably failed. Let's fix it.

## Step 2 - Creating, Exposing and Testing with Function URLs

### 2.1 - Creating

So, first of all, we are going to create and Expose the Function URLs so we can test our code.

Hit the Configuration tab and go for Function URL, than Create Function URL, following this order:

![Function URL](/images/lambda-multi-function/function-url.png)

For this example, I'm leaving the URL completly open to be accessed from the outside, which is not a good practice in production. (obviously, I took the function down so you don't bankrupt me)

![Function Permissions](/images/lambda-multi-function/function-permissions.png)

With that, your URL should appear on the function console like so:

![Function Endpoint](/images/lambda-multi-function/function-endpoint.png)

Hitting that won't work because both our Functions expect a post, so let's head back to Postman with that in hand.

### 2.2 - Testing

You can definetly test this with Curl, but I'm using Postman.

![Bye Test](/images/lambda-multi-function/bye-test.png)
