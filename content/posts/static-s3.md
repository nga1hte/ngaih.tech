---
layout: post
title: "Hosting a static site in AWS S3 bucket" 
slug: "statics3"
date: "2023-03-28"
comments: false
categories:
  - cloud
tags:
  - aws
  - s3
  - static
  - cloud
---

## Static Site
A static site refers to a website that is made up of static HTML, CSS, and JavaScript files, which are delivered to the user's web browser exactly as they are stored on the web server. In other words, the content of a static site is fixed and does not change dynamically in response to user input or other external factors.

Unlike dynamic websites, which rely on server-side processing to generate content on the fly, static sites are pre-built and require no server-side scripting or database access. This makes them faster, more secure, and less resource-intensive than dynamic sites, which can be more complex to build and maintain.

Static sites are typically used for simple websites that don't require advanced functionality or frequent updates, such as blogs, personal portfolios, or brochure-style websites. They can be hosted on any web server that supports static file hosting, including popular platforms like GitHub Pages, Netlify, or Amazon S3.

## Amazon S3

Amazon S3 (Simple Storage Service) is a cloud-based object storage service provided by Amazon Web Services (AWS). It allows users to store and retrieve large amounts of data from anywhere on the internet.

Amazon S3 provides a simple interface that can be used to store and retrieve any amount of data, at any time, from anywhere on the web. It is designed to be highly scalable, secure, and reliable. S3 is often used as a backend storage service for web and mobile applications, for data backup and archival purposes, and for hosting static websites.

## Hugo 
Hugo is an open-source static site generator written in the Go programming language (Golang). It is designed to be simple, fast, and flexible, allowing users to create static websites quickly and easily.

Hugo takes Markdown files, templates, and a configuration file, and generates a static website in HTML, CSS, and JavaScript. It is based on a powerful template engine that allows users to customize the appearance and layout of their websites without having to write complex code.

### Hosting this blog in AWS S3
Since this site is just a blog which requires no dynamic content, it is a static site generated using hugo. While this site technically use Amazon Web Services it does not utilise the simple storage service (S3). In this blog post let us migrate the site to S3 bucket.

You can migrate any static site to s3 bucket provided that you maintain the same directory structure inside the buckets. This is the directory and file structure of my blog and to host it on the ```S3``` bucket, I just have to upload the files exactly in the same structure. 

![img](/images/statics3/files.png)

First let us login to our aws management console, search ```S3``` bucket among the services and then create a s3 bucket. We can leave most of the default options, but create a bucket name, choose the appropriate region for your site (closest to you probably), but make sure to disable ```block all public access```, but make sure to disable ```block all public access``` and tick the acknowledgement. You can then create the bucket.

![img](/images/statics3/createbucket.png)
![img](/images/statics3/disableblock.png)

After getting redireted to the bucket dashboard click on the newly created bucket.

![img](/images/statics3/bucket.png)

Now click upload and upload all the files, you can upload multiple files too.

![img](/images/statics3/addfiles.png)

Wait for all files to be uploaded. I have slow internet speed so took me quite a while.

![img](/images/statics3/uploading.png)

Now that all the files have been uploaded go to the properties tab.

Enable static website hosting.

![img](/images/statics3/enablestatic.png)

Enter the index document and error document if present in your static site files and save changes.

![img](/images/statics3/enterindex.png)

Now go to the permissions tab to edit a bucket policy. A bucket policy allows us to enable permissions so that our ```S3``` bucket files can be accessed by the public.

Add the following lines of json configuration to the policy and save it.


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPublicAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::ngaihtech/*"
        }
    ]
}
```
- sid: Way to identify your policy.
- Effect: It is either Allow or Deny. 
- Principal: "*" is a wildcard to allow everyone.
- Action: "s3:GetObject" is a verb that allows to read the files from the s3 bucket.
- Resource: this is a way to identify your bucket, adding "/*" means giving access to all files inside the bucket.

![img](/images/statics3/editpolicy.png)


Now that we have given public access to the bucket files. Let us find the url of our bucket to access our hosted static site. Navigate to properties and scroll all the way to the bottom.

![img](/images/statics3/properties.png)

Copy that and you can access your hosted static site in s3 using the url.

![img](/images/statics3/web.png)

> http://ngaihtech.s3-website.eu-central-1.amazonaws.com

The link won't probably work anymore.

> Now that we have hosted our static site, we can also purchase a domain name and make it point to the url of our s3 bucket.




