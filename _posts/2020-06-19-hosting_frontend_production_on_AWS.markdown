---
layout: post
title: "AWS for Beginners: Hosting Frontend Production on AWS"
date: 2020-06-19 16:59:19 +0200
category: dev
tags: dev aws
---

![Cable network](/assets/2020-06-19-aws-for-beginners/taylor-vick-M5tzZtFCOfs-unsplash.jpg)

_Originally posted on [codestories](https://www.netguru.com/codestories/aws-for-beginners-hosting-frontend-production-on-aws)_

Amazon Web Services is an on-demand cloud computing services platform offered by Amazon. In simple words - they offer many services like file storage, databases, DNS, and servers you can use on their infrastructure.

Configuring infrastructure on Amazon Web Services (AWS) for the first time may seem painful. At first, it is not easy to get around the AWS web console as there are a lot of services and configuration options. Especially when we compare it to alternative services like Netlify or Vercel. But is it really that hard? I'll try to show you that it actually isn't.

This article is aimed at beginners who want to gain hands-on experience with AWS. I will show you how to create infrastructure for your single page application by clicking through the AWS web console. I believe the web console is a great introduction for first-timers as it makes it easier to understand all the services you will use. You can use that experience later if you would like to define your infrastructure with code. With Terraform and infrastructure as code, you will be able to spin up all those services with just one command.

You may think: Why can't I just use the static website serving option in S3? The most important differences are cost, performance, and configuration options. While S3 was designed as a simple (hence "simple" in the name) object storage service, CloudFront is more like a CDN - it will allow you to cache and serve your content from the location closest to the client. It is also cheaper and offers a lot of useful configuration options, including the powerful Lambda@Edge. Speaking about costs - if you want to serve a small website, the most expensive thing will be a Route 53 hosted zone - \$0.50/month :) You can find more about the costs on the AWS website.

## Objective

In this article, we will create a fully functional production environment for a frontend application. To achieve that, we will add our domain to Route 53, create an S3 bucket, CloudFront distribution, and generate an SSL certificate. I will guide you through the AWS web console step-by-step. After following the instructions in this article you should get hands-on knowledge about a few AWS services and the AWS web console. Note: I will not cover continuous integration configuration here and we will deploy by just uploading files to the bucket.

Before we start you will need:

- Domain with access to name servers configuration
- AWS account
- Some app to deploy (it can be just an index.html file)

If you want to play around with AWS you can create an account and take advantage of [AWS Free Tier](https://aws.amazon.com/free/) which will allow you to test AWS services for free. You can also create free domains on freenom.com.

## Step 1: domain to Route 53

Domain Name System is the phonebook of the internet. The general purpose of a nameserver is to translate domain names to IP addresses. So when you type in netguru.com in your browser, first there will be a request to a DNS server to obtain the site's IP address. Using the Route 53 DNS service from AWS you can configure your domain's DNS records or even buy a new domain there.

We will start by adding existing domain name servers to Route 53, which will allow us to manage DNS Records within AWS.

1. From the Services menu select **Route 53**
2. From the left menu in Route 53 select **Hosted zones**
3. Click **Create Hosted Zone**
4. A form will appear on the right side. Fill in your domain name (mine is depguru.ml). Comment is optional and Type should be set to **Public Hosted Zone**. After filling out the form click **Create** at the bottom.
![Aws console 4](/assets/2020-06-19-aws-for-beginners/4.png)
5. You are redirected to the hosted zone of your domain view. We will get back here later. Now click **Back to Hosted Zones**.
![Aws console 5](/assets/2020-06-19-aws-for-beginners/5.png)

Now it's time to “connect” our domain to our hosted zone.

**Warning: If you have other services attached to your domain - like email or subdomains - those will stop working after changing nameservers. So unless you know what you are doing I suggest creating a new domain to play with AWS - you can get a free domain on freenom.com. It is possible to retain all existing services but it requires different configuration.**

1. On the Hosted zones main page click on your domain (which you’ve just created)
2. A side panel with details will appear. Under **Name Servers** you can find name servers which you have to set in your domain configuration (ns-231.awsdns-28.com, ns-1414.awsdns-48.org, … in my case)
![Aws console 6](/assets/2020-06-19-aws-for-beginners/6.png)
3. In another tab, go to your domain registrar page (a website where you registered your domain), and head to domain settings. I got my domain from freenom.com, so I headed there and from **My domains** I selected **Manage Domain**. You have to find an option to set custom **nameservers**, and provide nameservers for your domain listed in Route 53. In some cases there will only be 2 fields for nameservers - don’t worry, you can just fill in the first two.
![Aws console 7](/assets/2020-06-19-aws-for-beginners/7.png)
4. Save your new nameservers and go back to the AWS console

## Step 2: Create S3 Bucket
AWS Simple Storage Service, or S3, is an object storage service from Amazon - basically you can store files there. Objects in S3 are organised into Buckets. You can think of it as the highest-level directory for your files. Usually we create a bucket per app, or per context (environment) - eg. 1 for app files and 1 for app logs. Let’s create a bucket where we will store our app.

1. From the Services menu select S3
2. Create a bucket
3. In the **Name and region** tab type a unique bucket name (mine will be: **depguru**) and select a region: **EU (Frankfurt)**. Click next
![Aws console 1](/assets/2020-06-19-aws-for-beginners/1.png)
4. In the **Configure options** tab, leave all options as is and click next
5. In **Set permissions**
    * Uncheck **Block all public access**
    * Check **I acknowledge that the current settings may result in this bucket and the objects within becoming public** which appears after unchecking Block all public access
    * Click next
![Aws console 2](/assets/2020-06-19-aws-for-beginners/2.png)
    * Click **Create bucket**
![Aws console 3](/assets/2020-06-19-aws-for-beginners/3.png)

Now you have it on your bucket list :)

## Step 3: Request SSL Certificate
An SSL Certificate enables secure (encrypted) communication between a user's browser and your server. It also authenticates the identity of the website - that’s why it is issued per domain name. Certificates are issued by Certificate authorities (CA). You can find out more here: [https://howhttps.works/](https://howhttps.works/)

We are setting up production, so SSL is a must. Thankfully, we can request certificates for free from AWS. Using ACM - a certificate manager provided by AWS - you can request and manage SSL certificates for your domain.

1. In the AWS console, find **Certificate Manager** in the **Services** menu
2. As we will be using a certificate with CloudFront, make sure that you have the **N.Virginia** region selected (dropdown in the top right corner).
3. Click **Request a certificate** (note: you can also import an existing certificate if you have one)
![Aws console 8](/assets/2020-06-19-aws-for-beginners/8.png)
4. A wizard form will appear. Select **Request a public certificate** and click **Request a certificate** button
![Aws console 9](/assets/2020-06-19-aws-for-beginners/9.png)
5. Now you need to specify domain names for your certificate. So you need to list all domain names you want to secure - in my case **depguru.ml** and **www.depguru.ml**. You can add any other subdomains here or even a wildcard (\*.depguru.ml) which will cover all subdomains. So specify the domain names and click **Next**.
![Aws console 10](/assets/2020-06-19-aws-for-beginners/10.png)
6. On the **Select validation method screen** you can choose between **DNS validation** (which requires you to have access to your domain’s DNS zone) or email validation (which requires you to have access to the email account listed as the contact address for this domain in WHOIS). We will select DNS validation as we have a DNS zone configured in our **Route 53** which makes validation really easy.
![Aws console 11](/assets/2020-06-19-aws-for-beginners/11.png)
7. You can skip adding Tags unless you need them
![Aws console 12](/assets/2020-06-19-aws-for-beginners/12.png)
8. On Step 4 review provided data, and if the domain names seem ok, click **Confirm and request**
![Aws console 13](/assets/2020-06-19-aws-for-beginners/13.png)
9. Now you can see the validation screen.
![Aws console 14](/assets/2020-06-19-aws-for-beginners/14.png)
10. To validate a domain managed in **Route 53** just expand each domain row and click **Create record in Route 53** - it will add the required validation record for us. There is also a DNS validation record listed if you want to add it manually.
![Aws console 15](/assets/2020-06-19-aws-for-beginners/15.png)
11. You should see success messages next to each domain
![Aws console 16](/assets/2020-06-19-aws-for-beginners/16.png)
12. After that click **Continue**. Now you will see a certificates screen with **Pending validation** next to your domain. It is fine - you just have to wait patiently till your domain is verified. You can click the refresh button next to a table and see if validation is done. For my domain using Route 53 it takes only a minute.
![Aws console 17](/assets/2020-06-19-aws-for-beginners/17.png)
13. If you see Status: **Issued** next to your certificate, you are good to go
![Aws console 18](/assets/2020-06-19-aws-for-beginners/18.png)

## Step 4: Create CloudFront distribution

From the user’s perspective it is best to receive data from the closest (geographically) location. So if I’m in Warsaw, Poland a request to North Virginia will take longer than a request to Frankfurt. To achieve that we will use CloudFront - a Content Delivery Network from AWS. A CDN is a bunch of globally-distributed servers that cache your content to improve access speed for end users. CloudFront distributes content using Amazon's edge locations which are located in major cities around the world to reduce latency.

We are now ready to configure CloudFront distribution (finally!).

1. As usual select **CloudFront** from the Services menu
2. Select **Create Distribution**
![Aws console 19](/assets/2020-06-19-aws-for-beginners/19.png)
3. On the next screen in the **Web** section click **Get Started**
![Aws console 20](/assets/2020-06-19-aws-for-beginners/20.png)
4. Now you will see a huuuuuge form to fill in. Thankfully we will leave most options default, and next to each option you have a hint if you click on the i sign

### Fill in Origin Settings:
![Aws console 21](/assets/2020-06-19-aws-for-beginners/21.png)

1. **Origin Domain Name**: the S3 bucket for your site (the input has a suggestion list)
2. **Origin Path**: directory within the bucket which you want to serve. I will upload files to the www directory so i type /www here
3. **Origin ID** should be filled automatically
4. **Restrict Bucket Access**: select **Yes** as we want users to be able to reach us **only** through CloudFront (they won't be able to use direct S3 links ← it is useful if you want to password protect your site)
5. (options after selecting Yes in **Restrict Bucket Access**)
    * **Origin**: Create a New Identity
    * **Comment**: leave as is or type anything that will help you identify this identity  
    * **Grant Read Permissions on Bucket**: select **Yes, Update Bucket Policy** -it will create a proper access policy and attach it to our bucket.
6. **Origin Custom Headers**: you can leave it for now. You will be able to add custom headers later

Fill in **Default Cache Behavior Settings**. I will leave most of the options with default values so here are the ones you should change:
![Aws console 22](/assets/2020-06-19-aws-for-beginners/22.png)

1. **Viewer Protocol Policy**: Select Redirect HTTP to HTTPS as we want to only serve our site through HTTPS
2. **Compress Objects Automatically**: Set to **Yes** which will enable gzip
3. in **Lambda Function Associations** you can attach Lambda@Edge functions (think hooks for requests and responses) which can for example ask the user for a password.

Fill in **Distribution Settings**:
![Aws console 24](/assets/2020-06-19-aws-for-beginners/24.png)

![Aws console distributions](/assets/2020-06-19-aws-for-beginners/distributionv2.png)

1. **Price Class**: Select which edge locations you want to deploy. More locations → higher cost, so I will select Use Only U.S., Canada and Europe as I don’t target my app for Asia or Africa.
2. **AWS WAF Web ACL**: skip
3. **Alternate Domain Names (CNAMEs)**: Put your domain names here (separated by commas or line-by-line). In my case: depguru.ml,www.depguru.ml. If you don’t have your own domain you can use one generated by Cloudfront like d12323.cloudfront.net, but it's not the best for production
4. **SSL Certificate**: Select Custom SSL Certificate and then select the certificate we’ve generated earlier in the input below
5. **Custom SSL Client Support**: leave as is (SNI), because LCI costs \$600 per month xD
6. **Security policy**: as is
7. **Supported HTTP Versions**: as is (HTTP/2 …)
8. **Default Root Object**: A file which will be returned when a user requests “/”. In our case index.html
9. **Logging**: Choose if you want CloudFront to log requests - it will then create log files with all requests to your app and persist them in the selected bucket. Probably a good idea to have it on production, but you will be charged for it.
10. If you choose Logging: **On** you also have to select:
    * **Bucket for Logs**: S3 bucket where your logs will be stored. It probably makes sense to create a separate bucket for that. I have **depx-logs**.
    * **Log prefix**: you can specify a directory within the bucket where logs will be stored
    * **Cookie Logging**: If you select Yes, the client’s cookies will be included in the logfile - we probably don’t need that, so I choose No
11. **Enable IPv6**: Yes, why not  
12. **Comment**: You can add any comment you feel like.
13. **Distribution State**: If you want to publish CloudFront Distribution right away - select **Enabled**. Otherwise you will be able to enable it later

Now you can click **Create Distribution** to submit the form. After that you can see your distribution after selecting **Distributions** from the left menu within the CloudFront section.

![Aws console 25](/assets/2020-06-19-aws-for-beginners/25.png)

## Step 5: Point domain to CloudFront distribution
We have to make our domain point to CloudFront distribution. To do that, we will create DNS records for our domain in Route 53. If you want to learn more about DNS records, take a look at [https://www.cloudflare.com/learning/dns/dns-records/](https://www.cloudflare.com/learning/dns/dns-records/). We will create A alias and AAAA alias records - in general A/AAAA records point to IP addresses, but AWS also offers alias records to route traffic to AWS resources using a domain name. You can read more here: [https://aws.amazon.com/route53/faqs/](https://aws.amazon.com/route53/faqs/).

Let’s go back to Route53 using the services menu. In the **Route53 Hosted zones** area click on your domain name (in my case: depguru.ml).

![Aws console 26](/assets/2020-06-19-aws-for-beginners/26.png)

We need to create 2 record sets (IPv4 and IPv6) for each subdomain. I have 2 subdomains (depguru.ml and www.depguru.ml) so i will create 4 records in total.

So repeat steps listed below for:

- Main domain (depguru.ml in my case)
    * name: <leave empty>
    * type: A and AAAA
- Subdomain (www.depguru.ml)
    * name: www
    * type: A and AAAA

1. Click **Create Record Set**. A side panel with a form will appear
![Aws console 27](/assets/2020-06-19-aws-for-beginners/27.png)
2. Fill in a name (or leave empty if you want to point to the main domain - in my case depguru.ml)
3. Select a type (listed above)
4. Alias: **Yes**
5. Alias target: select your CloudFront distribution from the auto suggestion list
![Aws console 28](/assets/2020-06-19-aws-for-beginners/28.png)
6. click **Create**

After that you should have 4 A/AAAA records.
![Aws console 29](/assets/2020-06-19-aws-for-beginners/29.png)

But wait, what's going on here?

We’ve set our DNS server to redirect users to our CloudFront Distribution.

So when a user asks

> Where can i get the depguru.ml website?

He will get the answer:

> A <IP of CF distribution>

We had to set it for both IPv4 and IPv6, and also for the **www.** subdomain as I wanted www.depguru.ml to point to my CloudFront Distribution. (note: www.depguru.ml and depguru.ml are in fact different domains and can point to two different sites).

## Step 6: Handle SPA routing in CloudFront
Our app is working now, but as this is a single page application the frontend takes care of routes. So it works fine when the user enters the app using the “/” path, but doesn't work when it starts from another page like “/users”. That’s because CloudFront serves index.html on / (we set the default Root Object to index.html) but doesn’t know what to do with other paths.

We can solve that with setting a fallback - tell CloudFront to redirect to index.html if it can’t find a file. This is probably not the best solution (may hurt SEO in some cases as it may return 200 for non-existent pages) but it is the simplest, so I’ll go with that.

To redirect error pages to index, you have to:

1. In the AWS console, select **CloudFront** from the Services menu
2. From the CloudFront Distributions list select yours and click **Distribution Settings**
![Aws console 39](/assets/2020-06-19-aws-for-beginners/39.png)
3. Go to Error Pages tab
![Aws console 40](/assets/2020-06-19-aws-for-beginners/40.png)
4. Click **Create Custom Error Response**, and fill in the form
![Aws console 41](/assets/2020-06-19-aws-for-beginners/41.png)
5. **Http Error Code**: 403 Forbidden
6. **Customize Error Response**: Yes
7. **Response**: /index.html
8. **HTTP Response Code**: 200 OK
9. Click **Create** and _repeat those steps_ for Http Error Code: 404
![Aws console 42](/assets/2020-06-19-aws-for-beginners/42.png)

After that if you head straight to https://depguru.ml/about you should still see my app.

## Step 7: Deploy & test
Now our infrastructure is ready to serve content… but we have no content yet! If you visit your domain now you will probably see an Access Denied error from S3.

We need to deploy our app to the S3 bucket now. For the sake of simplicity I'll just go to S3 in the AWS console and upload files to the www directory in my bucket using the GUI.

1. From the Services menu select *S3*
2. Select a bucket you've created before (depguru in my case)
3. Click **Upload**
![Aws console 30](/assets/2020-06-19-aws-for-beginners/30.png)
4. Drag and drop your **www** directory with app contents (in case of React: run yarn build, rename the build dir to www and drag into the bucket in your browser).
![Aws console 31](/assets/2020-06-19-aws-for-beginners/31.png)
5. Click **Next**
![Aws console 32](/assets/2020-06-19-aws-for-beginners/32.png)
6. On the **Set permissions** screen leave default options and click **Next**
![Aws console 33](/assets/2020-06-19-aws-for-beginners/33.png)
7. On the **Set properties** screen leave default options (Standard) and click **Next**
![Aws console 34](/assets/2020-06-19-aws-for-beginners/34.png)
8. Click **Upload**
![Aws console 35](/assets/2020-06-19-aws-for-beginners/35.png)

Now you should see the www directory and its contents.
![Aws console 36](/assets/2020-06-19-aws-for-beginners/36.png)

![Aws console 37](/assets/2020-06-19-aws-for-beginners/37.png)

After that, my app is available at depguru.ml

Note: Propagation of changes in DNS may take time (up to 24 hours) so don’t panic if you can’t see your site right away.

## Summary

So now we have a fully functional production environment on AWS! That wasn’t that hard, was it? It took me about 30 minutes to set up everything on my first attempt. The AWS console seems scarier than it in fact is.

Of course it is hard to contain all DevOps and AWS knowledge in one article, so I had to simplify things here and there and chose a simple use case to show the basic principles. But I still think it's a good starting point if you are interested in the subject. In general it is better to create infrastructure with code (see: Terraform) but if you are taking your first steps with AWS, I think using the GUI (the way shown in this article) is a good starting point.

Note: Be careful with credentials and selecting options. AWS bills you for usage, so doing something stupid may result in a huge bill at the end of the month. Also if your credentials leak somewhere, someone can use your account (and credit card) for things like mining cryptocurrencies.

---

_Cover photo by [Taylor Vick](https://unsplash.com/@tvick?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)_
![](https://macap.ct8.pl/image.php?url={{site.url}}{{page.url}})