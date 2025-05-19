---
title: "Build a CRUD App with Spring Boot & AWS DynamoDB: Step-by-Step Practical Guide"
date: 2025-05-19
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

I use the Spring Initializr via the Spring Boot Extension Pack on VSCode. You can always do it via the web, create your project, and unzip it to start coding. But what you're going to do is:

1. Create a Maven project
2. Spring Boot Version 3.4.5
3. Java
4. Name it as you wish
5. I've used Java 17 and JAR packaging
6. You'll add dependencies: Spring Web, Lombok (optional)

And you're good to go. Or almost. Just go to your `pom.xml` file and add these dependencies:

```xml
<!-- AWS SDK for DynamoDB -->
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb</artifactId>
    <version>2.26.27</version>
</dependency>
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>dynamodb-enhanced</artifactId>
    <version>2.26.27</version>
</dependency>
```

Now you're good to go. You can even run your code to check if everything works fine.

You should be careful if trying to add **Spring Boot Dev Tools**, it can break with some Dynamo stuff.

### 2.2 - Create your DynamoDBConfig File

To access AWS and DynamoDB we'll need a DynamoDBEnhancedClient, and that's why we will create the `DynamoDBConfig.java` file. In this file, we will create two `Bean` methods. That way, when Spring starts, it will run those methods once and store the results so that every time I request an object of those classes, it will provide it to me.

```
@Configuration
public class DynamoDbConfig {
  @Bean
  public DynamoDbClient dynamoDbClient() {
    return DynamoDbClient.builder()
        .region(Region.SA_EAST_1)
        .credentialsProvider(DefaultCredentialsProvider.create())
        .build();
  }

  @Bean
  public DynamoDbEnhancedClient dynamoDbEnhancedClient(DynamoDbClient dynamoDbClient) {
    return DynamoDbEnhancedClient.builder()
        .dynamoDbClient(dynamoDbClient)
        .build();
  }
}
```

If you're using AWS CLI, you should be able to access your credentials via the `DefaultCredentialsProvider`. Set your region and you're good to go.

The `DynamoDbClient` is an older way to access our DynamoDB Table, but we'll use the newer one.

### 2.3 - Create your Food Model

I like defining our model before anything else, so that's what we're doing first. As we've defined before, our Food item/row in our Dynamo FoodTable will have only id, food, and quantity values.

```
@Data
@DynamoDbBean
@AllArgsConstructor
@NoArgsConstructor
public class Food {
  private Long id;
  private String food;
  private Integer quantity;

  @DynamoDbPartitionKey
  public Long getId() {
    return id;
  }
}
```

If you've used Spring JPA before you can probably guess why we need both Constructors. If you haven't, I'll explain it a bit further ahead.

Despite having Lombok create our getId, we still need to define it to annotate it with `@DynamoDbPartitionKey` because that's how we let DynamoDB know that's our PK.

### 2.4 - Create your FoodRepository File

DynamoDB is a database, so the part of our code that has to handle it is the Repository. Yeas, the Model as well, but that's how far we can go. The Service and Controller layers shouldn't ever know we're using DynamoDB. They should never even know the word DB. They know about each other and the repository.

So this is where most of our Dynamo logic will be. What's not on the model at least. Let's create it for once.

```
@Repository
public class FoodRepository {
  private final DynamoDbTable<Food> foodTable;

  public FoodRepository(DynamoDbEnhancedClient dbEnhancedClient) {
    this.foodTable = dbEnhancedClient.table("FoodTable", TableSchema.fromBean(Food.class));
  }
}
```

Our @Repository here is mostly semantic, because we will have to do almost all of the work. Our `DynamoDbEnhancedClient` Bean will be injected via constructor and we'll use it to access the table on Dynamo via `DynamoDbTable`.

Let us create our create and update methods next.

```
public Food save(Food food) {
  this.foodTable.putItem(food);
  return food;
}

public Food update(Food food) {
  return this.foodTable.updateItem(food);
}
```

These are pretty standard, but they show that we'll always have to go through the `foodTable`. Let's get a bit more complicated:

```
  public List<Food> findAll() {
    return this.foodTable.scan().items().stream().toList();
  }

  public Food findById(Long id) {
    return this.foodTable.getItem(requestBuilder -> requestBuilder.key(keyBuilder -> keyBuilder.partitionValue(id)));
  }

  public void delete(Long id) {
    foodTable.deleteItem(requestBuilder -> requestBuilder.key(keyBuilder -> keyBuilder.partitionValue(id)));
  }
```

The `findAll` method uses the `scan` method from the `DynamoDbTable`. That method, provides a lot of functionallity for pagination and iteration, that's why we need a couple of steps before returning a simple List.

Both `findById` and `delete` methods use a nested lambda way of defining that our query is based on the PK. We could go into more detail on that, but we won't. It would take over a good part of this article.

### 2.5 - Create the Service and Controller Layers

As I've just said, our Service and Controller layers don't know we use DynamoDB, so they'll be pretty standard. If you don't understand something, look for a Spring Web tutorial, not for a DynamoDB with Spring tutorial.

```
@Service
@RequiredArgsConstructor
public class FoodService {
  private final FoodRepository repository;

  public Food create(Food food) {
    return repository.save(food);
  }

  public Food findById(Long id) {
    return repository.findById(id);
  }

  public List<Food> findAll() {
    return repository.findAll();
  }

  public Food update(Food food) {
    return repository.update(food);
  }

  public void delete(Long id) {
    repository.delete(id);
  }
}

@RestController
@RequiredArgsConstructor
public class FoodController {
  private final FoodService service;

  @GetMapping("{id}")
  public Food getByID(@PathVariable(value = "id") Long id) {
    return service.findById(id);
  }

  @GetMapping
  public List<Food> getAll() {
    return service.findAll();
  }

  @PostMapping
  public Food create(@RequestBody Food food) {
    return service.create(food);
  }

  @PutMapping
  public Food update(@RequestBody Food food) {
    return service.update(food);
  }

  @DeleteMapping("{id}")
  public void deleteByID(@PathVariable(value = "id") Long id) {
    service.delete(id);
  }
}
```

## Step 3 - Test your Application

Our software should already be working perfectly, so let us run it and use Postman to test it out. You can use `curl` or other software as well.

### 3.1 - Fetch all existing data

If you remember it well, when we created our database, we tested creating a sample data. That means we should be able to fetch all data and see that:

![Fetch All](/images/spring-dynamo-db/fetch-all.png)

### 3.2 - Create a new data

Now we should try creating data entries with a POST method. We will follow with another fetch all to confirm that our creation worked.

![Create](/images/spring-dynamo-db/create.png)

![Fetch All Create](/images/spring-dynamo-db/fetch-all-create.png)

### 3.3 - Update data

Let's now tryn reduce the amount of Apples by one using our update method.

![Put](/images/spring-dynamo-db/put.png)

![Fetch All Put](/images/spring-dynamo-db/fetch-all-put.png)

### 3.4 - Fetch specific data

Let's now try to access only the burger data with the fetch by id:

![Fetch By Id](/images/spring-dynamo-db/fetch-by-id.png)

### 3.5 - Delete data

Finally, let us delete the Burgers. We really should eat healthier!

![Delete](/images/spring-dynamo-db/delete.png)

![Fetch All Delete](/images/spring-dynamo-db/fetch-all-delete.png)

## Conclusion

And that's how you create a simple CRUD using Spring and AWS DynamoDB. That's also how you connect to your AWS account and create a DynamoDB Table using AWS CLI.

If you have questions or want to share your experience, feel free to reach out to me on [**LinkedIn**](https://www.linkedin.com/in/douglas-rocha-leite). There, I share more insights on software development and productivity.

I'd like to remind you that this is a part of a series of articles for a bigger project I'm working on. We've already covered AWS Lambdas and Spring Cloud Function on [**Part 1**](/projects/spring-aws-lambda), as well as a multi function Lambda that works with Function URLs on [**Part 2**](/projects/lambda-multi-function). Our next and final step is to add a front-end to this backend cloud crazyness. So stay tuned for that!
