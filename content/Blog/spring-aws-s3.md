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

First of all, I would recommend you change the region in the upper right corner to be the closest one to you. I forgot doing it the first time and had simple requests take a coulpe of seconds to return.

Now get to the search bar and type "S3" - or look for it in the menu if you have more time. Access the S3 page.

![AWS S3 Console](images/spring-aws-s3/s3-page.png)

You can see I have a weirdly named and wrong region Bucket created there. The idea of you following this tutorial is that you don't go through that as I did hehe

Create a Bucket.

You will need to provide an unique name, do that as you wish but it can't be:

`image-storage-and-retrieval-s3-test-douglas`

You can leave absolutely everything else as default and click **Create bucket** at the bottom of the page.

You should be back at the S3 Console page. If you click on your bucket's name you can look around on it's specific console and play with it. You can even start storing stuff. But, for the purpose of this tutorial, you just need to remember it's name.

### 1.3 Create your Amazon User

Now will access the **Identity and Access Management (IAM)** Dashboard. Type IAM on the search-bar and click the link. You should see something like this:

![IAM Console](images/spring-aws-s3/iam-dashboard.png)

On the Menu on your left, chose "Users". Then go for "Create User", type a username (anyone) and go for "Next".

On the "Set permissions" page, choose the option to your right: "Attach policies directly" like so:

![User Permissions](images/spring-aws-s3/user-permissions.png)

You will see a bug list of permissions show up in the bottom of the page. Type "**AmazonS3FullAccess**" and check that. 

![AmazonS3FullAccess](images/spring-aws-s3/s3-full-access.png)

Click "Next", review everything and click "Create User". You will be back at the list of your users. Click on the one you've just created to see a details screen for that user.

Once there, you should find the option to "Create Access Key" and click it. 

![Create Access Key](images/spring-aws-s3/create-access-key.png)

At this point, if you're just doing all of this to follow this tutorial and learn as I did, you can just click on "Local Code" and you're good to go. Otherwise, examine all of the options and chose what's best for you.

After a couple more "Next" clicks, you should see a screen like this:

![Keys](images/spring-aws-s3/keys.png)

Copy this keys. Print the screen. Do something with them. I turned them into Environment Variables right away so I could use them in my code, but do as you wish. 

## Step 2 - Creating your Spring Boot Application

### 2.1 Create a Project with Spring Initilzr

I use the Spring Initilizr via the Spring Boot Extension Pack on VSCode. You can always do it via the web, create your project and unzip it to start coding. But what you're going to do is 

1. Create a Maven Project
2. Spring Boot Version 3.4.3
3. Java
4. Name it as you wish
5. I've used Java 17 and Jar packing
6. You'll add dependencies: Spring Web, Spring Boot Dev Tools (Optional), Lombok (Optional)

And you're good to go. Or almost. Just go to your `pom.xml` file and add this dependency:

```
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>1.12.182</version>
</dependency>
```

Now you're good to go. You can even run your code to check if everything works fine. 

### 2.2 Create your S3Config file

The first file we're going to create will be the `S3Config.java`. In this file we will create a `Bean` method. That way, when Spring begins, it will run that method once and store the result in a way that, every time I ask for an object of that class, it will give it to me. 

```
@Configuration
public class S3Config {
    
    @Bean
    public AmazonS3 amazonS3() {
        BasicAWSCredentials awsCredentials = new BasicAWSCredentials( //
                System.getenv("AWS_ACCESS_KEY_ID"), //
                System.getenv("AWS_SECRET_ACCESS_KEY"));

        return AmazonS3ClientBuilder //
                .standard() //
                .withRegion(System.getenv("AWS_REGION")) //
                .withCredentials(new AWSStaticCredentialsProvider(awsCredentials)) //
                .build();
    }
}
```

As I've said, I turned the credentials into environment variables, but you can just paste them here if you want to. 

### 2.3 Create your S3Service file

The `S3Service.java` file will be responsible for adapting images from one place to another. That is, it'll take an image and store and will fetch an image from storage, when requested.

Here you will have access to the amazonS3 object like it were a Repository. You will also hold the bucket information. 

```
@Service
public class S3Service {
    
    @Autowired
    private AmazonS3 amazonS3;
    private final String bucketName = "image-storage-and-retrieval-s3-test-douglas";

    public byte[] getImage(String fileName) throws IOException {
        S3Object s3Object = amazonS3.getObject(bucketName, fileName);
        try (S3ObjectInputStream objectInputStream = s3Object.getObjectContent()) {
            return IOUtils.toByteArray(objectInputStream);
        }
    }

    public String uploadImage(MultipartFile file) throws IOException {
        File tempFile = convertMultiPartFileToFile(file);
        amazonS3.putObject(bucketName, file.getOriginalFilename(), tempFile);
        tempFile.delete();
        return "Imagem carregada com sucesso: " + file.getOriginalFilename();
    }

    private File convertMultiPartFileToFile(MultipartFile file) throws IOException {
        File tempFile = new File(file.getOriginalFilename());
        try (FileOutputStream fos = new FileOutputStream(tempFile)) {
            fos.write(file.getBytes());
        }
        return tempFile;
    }
}
```

Here I decided to use a MultiPart communication, but you can probably do it differently. The purpose of this tutorial is to connect to S3, so I didn't bother much with that.

### 2.4 Create your S3Controller so you can access it from outside

Writing this I remembered when I was in university and we created little terminal menus to access the software's functions. Teachers would've done us so much good if they thaught us how to do it with APIs instead, but sure...

Let's go. Here you have 3 options: 

1. Create the Get Method and upload your images via S3 Console;
2. Create the Post Method and check if they're available on the S3 Console;
3. Create both Get and Post Methods.

All 3 options will give you a Spring Boot Application that connects with a S3 Bucket, but I will do the third. 

```
@RestController
@RequestMapping()
public class S3Controller {
    @Autowired
    private S3Service s3Service;

    @GetMapping("/image/{fileName}")
    public ResponseEntity<byte[]> getImage(@PathVariable String fileName) throws IOException {
        byte[] imageBytes = s3Service.getImage(fileName);
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.IMAGE_JPEG);

        return ResponseEntity.ok()
                .headers(headers)
                .body(imageBytes);
    }

    @PostMapping("/upload")
    public String uploadImage(@RequestParam("file") MultipartFile file) {
        try {
            return s3Service.uploadImage(file);
        } catch (Exception e) {
            e.printStackTrace();
            return "Erro no upload da imagem!";
        }
    }
}
```

You can change the mapping if you want... I really don't care much. 

"*You're code is wrong! What if the image is not JPEG?*"

You can fetch the metadata from the bucket, but I only tested it with JPEG images so I didn't care. 

You're code should be running smoothly by know. 



