# Module 1: Static Web Hosting with Amazon S3

In this module you'll configure Amazon Simple Storage Service (S3) to host the static resources for your web application. In subsequent modules you'll add dynamic functionality to these pages using JavaScript to call remote RESTful APIs built with AWS Lambda and Amazon API Gateway.


## Architecture Overview

The architecture for this module is very straightforward. All of your static web content including HTML, CSS, JavaScript, images and other files will be stored in Amazon S3. Your end users will then access your site using the public website URL exposed by Amazon S3. You don't need to run any web servers or use other services in order to make your site available.

![Static website architecture](../images/static-website-architecture.png)

For the purposes of this module you'll use the Amazon S3 website endpoint URL that we supply. It takes the form `http://{your-bucket-name}.s3-website-{region}.amazonaws.com` or `bucket-name.s3-website.region.amazonaws.com` depending on the region you use. For most real applications you'll want to use a custom domain to host your site. After the workshop if you're interested in using a your own domain, follow the instructions for [setting up a static website using a custom domain](http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) in the Amazon S3 documentation.

## Implementation Instructions

Each of the following sections provides a brief implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### Region Selection

This workshop can be deployed in any AWS region that supports the following services:

- Amazon Cognito
- AWS Lambda
- Amazon API Gateway
- Amazon S3
- Amazon DynamoDB

You can refer to the [region table](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) in the AWS documentation to see which regions have the supported services. 

**Use the Ireland region please**.

Once you've chosen a region, you should deploy all of the resources for this workshop there. Make sure you select your region from the dropdown in the upper right corner of the AWS Console before getting started.

![Region selection screenshot](../images/region-selection.png)

### 1. Create an S3 Bucket

Amazon S3 can be used to host static websites without having to configure or manage any web servers. In this step you'll create a new S3 bucket that will be used to host all of the static assets (e.g. HTML, CSS, JavaScript, and image files) for your web application.

#### High-Level Instructions

Use the console or AWS CLI to create an Amazon S3 bucket. Keep in mind that your bucket's name must be globally unique across all regions and customers. We recommend using a name like `wildrydes-firstname-lastname`. If you get an error that your bucket name already exists, try adding additional numbers or characters until you find an unused name.


### 2. Upload Content

Upload the website assets for this module to your S3 bucket. Use the AWS CLI to complete this step.

<details>
<summary><strong>CLI step-by-step instructions (expand for details)</strong></summary><p>

If you already have the CLI installed and configured, you can use it to copy the necessary web assets from `s3://SOURCE_BUCKET` to your bucket.

**Tip**: `s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website s3://YOUR_BUCKET_NAME --recursive`

You should see a list of objects that were copied to your bucket.
</p></details>


### 3. Add a Bucket Policy to Allow Public Reads

You can define who can access the content in your S3 buckets using a bucket policy. Bucket policies are JSON documents that specify what principals are allowed to execute various actions against the objects in your bucket.

#### High-Level Instructions

You will need to add a bucket policy to your new Amazon S3 bucket to let anonymous users view your site. By default your bucket will only be accessible by authenticated users with access to your AWS account.

See [this example](http://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) of a policy that will grant read only access to anonymous users. This example policy allows anyone on the Internet to view your content. 

The easiest way to update a bucket policy is to use the console. Select the bucket, choose the permission tab and then select Bucket Policy. As a reminder, by default, all buckets are now created with additional guard-rails to prevent security misconfigurations and you need to relax the public bucket policies. 


### 4. Enable Website Hosting

By default objects in an S3 bucket are available via URLs with the structure `http://<Regional-S3-prefix>.amazonaws.com/<bucket-name>/<object-key>`. In order to serve assets from the root URL (e.g. /index.html), you'll need to enable website hosting on the bucket. This will make your objects available at the AWS Region-specific website endpoint of the bucket. 


#### High-Level Instructions

Using the console, enable static website hosting. You can do this on the Properties tab after you've selected the bucket. Set `index.html` as the index document, and leave the error document blank.

Note the **Endpoint** URL at the top of the dialog before choosing **Save**. You will use this URL throughout the rest of the workshop to view your web application. From here on this URL will be referred to as your website's base URL.


## Implementation Validation

After completing these implementation steps you should be able to access your static website by visiting the website endpoint URL for your S3 bucket.

Visit your website's base URL (this is the URL you noted in section 4) in the browser of your choice. You should see the Wild Rydes home page displayed. If you need to lookup the base URL, visit the S3 console, select your bucket and then click the **Static Web Hosting** card on the **Properties** tab.

If the page renders correctly (see below for an example screenshot), you can move on to the next module, [User Management](../2_UserManagement).

![Wild Rydes homepage screenshot](../images/wildrydes-homepage.png)
