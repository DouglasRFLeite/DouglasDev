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
description: "A tutorial on how I created a Spring Boot application to store and retrieve images from an AWS S3 Bucket"
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
    image: "images/cool-setup-007.jpg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunately not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

So... my wife was busy on a Sunday afternoon. Do you know what that means? Yeah... It means I'm going to code my a\*s off.

I've been wanting to start playing with AWS for some time now, and this week I thought of an interesting use for AWS S3 at work and wanted to test it out.

The idea is to store and retrieve images on S3 using a Spring Boot application.

### What is AWS S3?

I really don't want to get into too much detail here; the purpose of this article is to showcase this project and give you a step-by-step tutorial on how to replicate it yourself.

But S3 is one of AWS's many cloud storage services, designed specifically to store simple files, like an actual file system. In this tutorial, I'm storing files in the root, so I really won't get into much more detail than that. But you can check some of the references if you want more information.

**Disclaimer!** I've just mentioned that I'm new to cloud and AWS, so pardon me if I said anything stupid.

### What is Spring Boot?

Again, I don't want to get into too much detail, and I assume that if you're here, you've probably heard of Spring.

But, again, Spring Boot is a Java framework (let's not get into too many details, folks) that quickens the initialization and growth of complex Java applications. Here, specifically, I only use Spring Core (obviously) and Spring Web for the API.

### Prerequisites

If you want to follow me through every step of this project, you are probably going to need:

- Java 17+ (installed on your machine or environment)
- Visual Studio Code (to be referenced as VSCode) or any other IDE
- Internet connection

That's about it. Spring is a Java dependency, so you don't need to download it separately.

## Step 1 - Creating your AWS S3 Bucket

### 1.1 - Create your account

First of all, you should access the [**AWS Console**](https://aws.amazon.com/console/) and create a free account.

I'm not going into detail on this step, but it does take some time, so do it carefully. I created a free account, but if you're Elon Musk's son, you may want to go ahead and create a paid one.

### 1.2 - Create your S3 Bucket

After successfully logging in, you should be redirected to the actual AWS Console.

It should look like this if this is your first time on it:

![AWS Console](/images/spring-aws-s3/aws-console.png)

First of all, I would recommend you change the region in the upper right corner to the closest one to you. I forgot to do it the first time and had simple requests take a couple of seconds to return.

Now go to the search bar and type "S3" - or look for it in the menu if you have more time (or just click [here](https://sa-east-1.console.aws.amazon.com/s3/home)). Access the S3 page.

![AWS S3 Console](/images/spring-aws-s3/s3-page.png)

You can see I have a weirdly named and incorrectly set region bucket created there. The idea of you following this tutorial is that you don't go through that as I did, hehe.

Create a bucket.

You will need to provide a unique name. Do that as you wish, but it can't be:

`image-storage-and-retrieval-s3-test-douglas`

You can leave absolutely everything else as default and click **Create bucket** at the bottom of the page.

You should be back at the S3 Console page. If you click on your bucket's name, you can look around its specific console and play with it. You can even start storing stuff. But for the purpose of this tutorial, you just need to remember its name.

### 1.3 - Create your Amazon User

Now we will access the **Identity and Access Management (IAM)** Dashboard. Type IAM in the search bar and click the link (or click [here](https://us-east-1.console.aws.amazon.com/iam/home)). You should see something like this:

![IAM Console](/images/spring-aws-s3/iam-dashboard.png)

On the menu on your left, choose "Users." Then click "Create User," type a username (any one), and click "Next."

On the "Set permissions" page, choose the option to your right: "Attach policies directly," like so:

![User Permissions](/images/spring-aws-s3/user-permissions.png)

You will see a big list of permissions show up at the bottom of the page. Type "**AmazonS3FullAccess**" and check that.

![AmazonS3FullAccess](/images/spring-aws-s3/s3-full-access.png)

Click "Next," review everything, and click "Create User." You will be back at the list of your users. Click on the one you've just created to see a details screen for that user.

Once there, you should find the option to "Create Access Key" and click it.

![Create Access Key](/images/spring-aws-s3/create-access-key.png)

At this point, if you're just doing all of this to follow this tutorial and learn as I did, you can just click on "Local Code" and you're good to go. Otherwise, examine all of the options and choose what's best for you.

After a couple more "Next" clicks, you should see a screen like this:

![Keys](/images/spring-aws-s3/keys.png)

Copy these keys. Print the screen. Do something with them. I turned them into environment variables right away so I could use them in my code, but do as you wish.

## Step 2 - Creating your Spring Boot Application

### 2.1 - Create a Project with Spring Initializr

I use the Spring Initializr via the Spring Boot Extension Pack on VSCode. You can always do it via the web, create your project, and unzip it to start coding. But what you're going to do is:

1. Create a Maven project
2. Spring Boot Version 3.4.3
3. Java
4. Name it as you wish
5. I've used Java 17 and JAR packaging
6. You'll add dependencies: Spring Web, Spring Boot Dev Tools (optional), Lombok (optional)

And you're good to go. Or almost. Just go to your `pom.xml` file and add this dependency:

```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <version>1.12.182</version>
</dependency>
```

Now you're good to go. You can even run your code to check if everything works fine.

### 2.2 Create Your S3Config File

The first file we're going to create is `S3Config.java`. In this file, we will create a `Bean` method. That way, when Spring starts, it will run that method once and store the result so that every time I request an object of that class, it will provide it to me.

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

As I mentioned, I turned the credentials into environment variables, but you can just paste them here if you prefer.

### 2.3 Create Your S3Service File

The `S3Service.java` file will be responsible for adapting images from one place to another. That is, it will take an image, store it, and retrieve an image from storage when requested.

Here, you will have access to the `amazonS3` object as if it were a repository. You will also store the bucket information.

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
        return "Image loaded successfully: " + file.getOriginalFilename();
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

Here, I decided to use multipart communication, but you can probably do it differently. The purpose of this tutorial is to connect to S3, so I didn’t focus much on that.

### 2.4 Create Your S3Controller to Access It Externally

Writing this, I remembered when I was in university, and we created little terminal menus to access the software’s functions. Teachers would have done us a great favor if they had taught us how to do it with APIs instead, but sure...

Let’s go. Here you have three options:

1. Create the GET method and upload your images via the S3 Console.
2. Create the POST method and check if they’re available in the S3 Console.
3. Create both GET and POST methods.

All three options will give you a Spring Boot application that connects with an S3 bucket, but I will do the third.

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

You can change the mapping if you want... I really don’t care much.

_"Your code is wrong! What if the image is not JPEG?"_

You can fetch the metadata from the bucket, but I only tested it with JPEG images, so I didn’t care.

Your code should be running smoothly by now.

## Step 3 - Test Your Application

Yeah, so... who knows if I lied and nothing is working? You’d be pretty mad at me for the lost time if that were the case, wouldn’t you? So let’s actually test it out.

### 3.1 - Store an Image

First, you need an image to store. I love Pokémon, so this will be my image:

![Pikachu](/images/spring-aws-s3/pikachu.jpg)

Now, you need something to make a POST request to your application. Can you create another application just to do that? Sure. Can you use Curl? Sure. I'm lazy, so I’m using Postman’s VSCode extension.

On the "New HTTP Request" page, you should change the method to **POST**, go to the **Body** section, and choose **form-data**. Add a new key **file**, set its type to **File**, and select your file. Now just hit **Send**. It will look like this:

![VSCode Postman](/images/spring-aws-s3/vscode-postman.png)

If you now go back to your S3 Console, you should see it there like this:

![S3 Console with data](/images/spring-aws-s3/s3-console-pikachu.png)

### 3.2 Retrieve an Image

Now, we’re going to retrieve that image via the GET method.

You can, if you want, make that GET request with Postman. But I just love Google Chrome, so I want to see the image in the browser:

![Pikachu showing on browser](/images/spring-aws-s3/pikachu-browser.png)

## Conclusion

There you go!

Now you can say you’re an expert in Cloud, Data Storage, and Spring + AWS integration (I sure will).

Jokes aside, I’m glad to have had my first real contact with Amazon AWS using Spring. And I am also very happy to share this with all of you.

That said, I did use Spring, but you could just as well instantiate every class in your main method and use plain Java. You can even take advantage of the AWS setup part and develop your application with Python. Feel free to experiment.

If you have any questions or want to leave any comments, I recommend reaching out to me on [**LinkedIn.**](https://www.linkedin.com/in/douglas-rocha-leite). There, I also share more insights, software development, and productivity content.

### References

1. Spring Boot Documentation
   https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/
   The official Spring Boot documentation for configuration, service layers, and REST controllers.

2. AWS SDK for Java
   https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/welcome.html
   The official AWS SDK documentation for working with S3 and other AWS services in Java.

3. Amazon S3 Documentation
   https://docs.aws.amazon.com/s3/index.html
   Official documentation for working with Amazon S3, including best practices and API references.
