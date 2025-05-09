---
title: "How to Deploy Multiple Serverless Lambda Functions with One Spring Cloud Function Project on AWS"
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

Have you ever had trouble deploying multiple functions in an easy, cost-effective, and efficient way? In this tutorial, you'll learn how to deploy a single Spring Cloud Function project to AWS with the capability to host multiple functions and endpoints.

This is **Part 2** of a series of smaller tutorials I’m publishing, all related to the same project. I’ll go into more detail about the project specifics when I write a dedicated article, but it will include **AWS Lambda, AWS DynamoDB, and a React front-end**, so stay tuned.

This tutorial aims to be concise, building upon the content from [**Part 1**](/projects/spring-aws-lambda), which covers the basics of creating, deploying, and testing a Spring Cloud Function project on AWS Lambda.

The main goal here is to show how you can have multiple functions within a single project and access them via **Function URLs**. It sounds simple, but it took some research to figure out the best approach.

To illustrate this, we’ll expand our hello function from the first article into a hello-bye function.

### What and Why...

### ... AWS Lambda and Spring Cloud Function

We’re not revisiting this topic here. For that, refer to [**Part 1**](/projects/spring-aws-lambda).

### ... Multiple Functions?

There are several architectures for more complex serverless applications. A common one is to separate each domain, or even each function, into a single project and deploy them as individual Lambdas (or other serverless functions from different providers). This is very much aligned with a microservices mindset, and there’s nothing wrong with it.

However, one challenge with this approach is the duplication of code, business logic, and, in our case, database connectivity code.

_What do you mean by "in our case"? This code doesn’t connect to any database..._

Not yet, little Kid Flash. But it will.

And yes, even in these scenarios, it might be worth splitting the project into multiple services and deploying them as separate functions.

Suppose you have 5 domains in your project: A, B, C, D, and E, which communicate in pairs—meaning Function A needs some code from B, Function B needs code from C, and so on. You’d end up with 5 functions, each containing code for nearly two entire domains. While this may seem like a lot of duplication, it’s still far less than having a single function with code from all 5 domains. In that case, splitting makes sense.

In our situation, there are only 2 domains, 2 types of data, and a single database. Duplicating nearly 90% of the code just to separate entry points doesn’t make sense. **So, multiple functions it is.**

### ... Function URLs?

A Lambda Function URL is a dedicated, straightforward HTTPS endpoint for a specific Lambda function. It offers a simpler way to invoke your Lambda over HTTP without needing to set up an API Gateway.

Is it better than API Gateway? If by “better” you mean more powerful and full-featured, then no. AWS API Gateway is a fully managed service that acts as a unified "front door" for all your backend services, including Lambda functions. It can do much more than just provide an endpoint.

But if you mean **easier, simpler, and more suited to this specific case**, then yes—Function URLs are a better fit.

## Step 1 - Creating and Deploying a Spring Cloud Function Project with Multiple Functions

### 1.1 - Recap of Part 1

I won’t go into detail about creating a Spring Cloud Function project here, since that’s covered in [**Part 1**](/projects/spring-aws-lambda). I’ll just show the code and modify the project name to `cloud-hello-bye`.

Here’s what your project should look like if you followed along:

```java
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

You should be able to run and test this locally via HTTP requests (using curl, Postman, etc.). You can even deploy it to AWS as a Lambda and verify it works. But that’s the topic of Part 1, so let’s move forward.

### 1.2 - Adding the Bye Function

Now, let’s expand our project by adding another function directly in the same Application class (note: don’t do this in production code!).

```java
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

Done! Now your project hosts two functions. You can test them by adjusting the URL:

```bash
curl -X POST -H "Content-Type: text/plain" -d "Douglas" http://localhost:8080/helloFunction
# Output: Hello, Douglas

curl -X POST -H "Content-Type: text/plain" -d "Douglas" http://localhost:8080/byeFunction
# Output: Bye bye, Douglas
```

### 1.3 - Deploying to Lambda

Refer to [**Part 1**](/projects/spring-aws-lambda) for the deployment steps—they are exactly the same. Once deployed, test your functions there.

If you tried testing as in Part 1, you might have run into issues. Let’s fix that.

## Step 2 - Creating, Exposing, and Testing via Function URLs

### 2.1 - Creating the Function URL

First, we’ll create and expose Function URLs so we can test directly.

Go to the **Configuration** tab, select **Function URL**, then click **Create Function URL**:

![Function URL](/images/lambda-multi-function/function-url.png)

For this example, I will leave the URL openly accessible (not recommended for production, so I've removed it after releasing this article). 

![Function Permissions](/images/lambda-multi-function/function-permissions.png)

Once set up, your URL will appear in the Lambda console:

![Function Endpoint](/images/lambda-multi-function/function-endpoint.png)

Testing this URL directly won’t work immediately because both functions expect POST requests. So, let’s move on to testing with Postman or curl.

### 2.2 - Testing

You can definetly test this with Curl, but I'm using Postman.

![Bye Test](/images/lambda-multi-function/bye-test.png)
