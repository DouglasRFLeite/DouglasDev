---
title: "How to Connect, Write and Read Data to a DynamoDB Database with Spring Boot"
date: 2025-05-12
weight: 995
tags: ["Java", "Spring", "Spring Boot", "Database", "NoSQL", "AWS", "Cloud", "DynamoDB", "AWS CLI"]
author: "Douglas"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A tutorial on creating a CRUD application with AWS DynamoDB and AWS CLI"
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
    caption: "Unfortunately not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

Have you ever wondered how DynamoDB, the famous AWS NoSQL Database, works? Even more, how you can connect to it via Java and Spring? That's what we're seeing on this tutorial. As a bonus, we're using the AWS CLI to create and configure our DynamoDB table.

This is **Part 3** of a series of smaller tutorials I’m publishing, all related to the same project. I’ll go into more detail about the project specifics in a dedicated article later, but it will include **AWS Lambda, AWS DynamoDB, and a React front-end**, so stay tuned.

We're not going to build on the content from the previous 2 parts on this tutorial, because they focus more on AWS Lambda and Spring Cloud Functions instead of DynamoDB. We'll use Spring Web MVC to access our database externally. We'll also use AWS CLI for the first time in this "series" to access our AWS environment, create and configure the DynamoDB database.

### What and Why...

### ... DynamoDB?

DynamoDB is a fully managed NoSQL database service provided by Amazon Web Services (AWS). It is designed for high performance, scalability, and low latency, making it ideal for applications that require rapid access to large amounts of data. DynamoDB stores data in a flexible, schema-less format using key-value and document data models, allowing for easy and dynamic data structures.

People use DynamoDB because it handles heavy read and write loads seamlessly, offers automatic scaling, and requires minimal maintenance. It's particularly useful for real-time applications like gaming, IoT, mobile apps, and web services that need quick, reliable access to data without worrying about infrastructure management.

Structurally, DynamoDB organizes data into tables, with each table containing items (rows) that have attributes (columns). Each item is uniquely identified by a primary key, which can be a simple partition key or a composite key with a sort key. This structure allows for efficient querying and flexible data organization, supporting both simple lookups and complex queries at scale.

I can't really say it's the best idea for our final project. It's not a bad one tho, and I really want to learn it, that's why I'm going for it.

### ... AWS CLI

The focus of this article is on DynamoDB, not AWS CLI, but since we're using it, I feel like I should explain why.

The AWS Command Line Interface (CLI) is a powerful tool that allows us to manage and automate AWS services through simple commands in a terminal or script, rather than using the AWS Management Console's graphical interface.

It is especially useful for scripting repetitive tasks, deploying resources quickly, and integrating AWS operations into CI/CD pipelines, saving time and reducing manual effort.

## Step 1 - Create a DynamoDB Table using AWS CLI

### 1.1 - Create your AWS User Access Keys

I've gone into more detail on this matter on this [**Spring Boot & AWS S3 for Image Storing**](/projects/spring-aws-s3) tutorial, specially on topics 1.1 and 1.3. Take a look there and you should finish those steps with your hands on your AWS Access Key Id and AWS Secret Access Key. Also be aware of the region.

**Important!** Make sure to give Admin permissions for DynamoDB to this user, instead of S3 like is done in that tutorial.

### 1.2 - Install and configure AWS CLI

Installing AWS CLI should be pretty easy. Check the [**Installing or updating to the latest version of the AWS CLI**](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) documentation on the AWS website. I've just done this:

```
sudo snap install aws-cli --classic
```

After installation, check it with:

```
aws --version
```

Now we're configuring it. Have your AWS Keys at hand now. You will run the command and it will query you for the keys, the region and an output pattern, json is standard. This is the command:

```
aws configure
```

You will be able to check your authentication with:

```
aws sts get-caller-identity
```

### 1.3 - Create the DynamoDB Table via AWS CLI

For this tutorial, we're using a very simple domain as example: a Food Market stock. Our data will basically follow this model:

| Name     | Type   |
| -------- | ------ |
| id (PK)  | number |
| food     | string |
| quantity | number |

If our introduction to DynamoDB didn't make it clear enough for you, we can make it clearer with this example. Our id will be our Partition Key (no, not Primary, Partition), and it defines food as our main data in this table, and it will be ordered by that id.

All of our data will be on this table. Yes, on this single table. "_What if I have other resources and relationships?_". In that case, you can use Partition + Sorting Keys, we will do that in our bigger project.

But yeah, we're not going much deeper into the theory behing DynamoDB.

To create this table on DynamoDB via AWS CLI we use this command:

```
aws dynamodb create-table \
    --table-name FoodTable \
    --attribute-definitions AttributeName=id,AttributeType=N \
    --key-schema AttributeName=id,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --region sa-east-1
```

If you look into this, you'll notice we didn't define our food nor our quantity columns. That's right, we didn't. We don't really have to. DynamoDB is NoSQL, not Structured, we can add mostly any column we want and it won't break the table, as long as the id is stable.

We can verify our table creation with this command:

```
aws dynamodb describe-table --table-name FoodTable --region sa-east-1
```

```
{
    "Table": {
        "AttributeDefinitions": [
            {
                "AttributeName": "id",
                "AttributeType": "N"
            }
        ],
        "TableName": "FoodTable",
        "KeySchema": [
            {
                "AttributeName": "id",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2025-05-15T11:28:29.949000-03:00",
        "ProvisionedThroughput": {
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:sa-east-1:851725254651:table/FoodTable",
        "TableId": "95fc0628-ad0a-4e36-ae44-f909cc8b1e23",
        "DeletionProtectionEnabled": false,
        "WarmThroughput": {
            "ReadUnitsPerSecond": 5,
            "WriteUnitsPerSecond": 5,
            "Status": "ACTIVE"
        }
    }
}
```

We can insert some mock data into our table to see if it works:

```
aws dynamodb put-item \
    --table-name FoodTable \
    --item '{"id": {"N": "1"}, "food": {"S": "Apple"}, "quantity": {"N": "10"}}' \
    --region sa-east-1
```

```
aws dynamodb scan \
    --table-name FoodTable \
    --region sa-east-1
```

```
{
    "Items": [
        {
            "id": {
                "N": "1"
            },
            "quantity": {
                "N": "10"
            },
            "food": {
                "S": "Apple"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

## Step 2 - Creating your Spring Boot Application

### 2.1 - Create a Project with Spring Initializr

### 2.2 - Create your DynamoDBConfig File

### 2.3 - Create your FoodTable Model

### 2.4 - Create your FoodTableRepository File

### 2.5 - Create the Service and Controller Layers

## Step 3 - Test your Application

### 3.1 - Fetch all existing data

### 3.2 - Create a new data

### 3.3 - Update data

### 3.4 - Fetch specific data

### 3.5 - Delete data

## Conclusion
