+++
tags = ["aws", "cloudfront", "s3"]
Description = "Cloudfront CDN for Amazon S3 Static Web Hosting"
date = "2016-01-09T12:09:37+02:00"
menu = "main"
title = "Cloudfront CDN for Amazon S3 Static Web Hosting"
summary = "In this post, I will be guiding you how to setup an Amazon S3 bucket as a static website, then configure CloudFront CDN to serve in front of Amazon S3."

+++


In this post, I will be guiding you how to setup an Amazon S3 bucket as a static website, then configure CloudFront CDN to serve in front of Amazon S3. Outline is as following:

* Create a S3 Bucket
* Configure Amazon S3 to serve as a static website
* Create Cloudfront distribution.
* Set Route53 records
* Restrict S3 access to Cloudfront

## Amazon S3

Create an S3 bucket and then follow **Amazon S3 bucket properties** -> **Permissions** -> **Edit bucket policy** and paste the following text.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicReadGetObject",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::www.mysite.com/*"
		}
	]
}
```

Amazon S3 bucket properties -> Static Web Hosting, choose **Enable website hosting** and provide Index Document as *index.html*. You will see the site url there: *www.mysite.com.s3-website-eu-west-1.amazonaws.com*. Now upload some file and check if your site is working.

## Cloudfront

Navigate to CloudFront. Click on [Create Distribution](https://console.aws.amazon.com/cloudfront/home?region=us-east-1#create-distribution:) button. Choose **Get Started** below **Web**.

* **Origin Domain Name**: *www.mysite.com.s3-website-eu-west-1.amazonaws.com* (write the path you see in Amazon S3 bucket properties -> Static Web Hosting -> Endpoint: *www.mysite.com.s3-website-eu-west-1.amazonaws.com*). You could choose your S3 Bucket here but if you do so, you won't be able to display subdirectories: *http://www.mysite.com/blog/* which contains a index.html. You will have to call index.html explicitly (*http://www.mysite.com/blog/index.html*).
* **Origin Path:** Empty or */*
* **Origin ID:** Enter a descriptive ID for this origin: S3-mysite.com
* **Compress Objects Automatically:** Yes
* **Price Class:** US and Europe
* **Alternate Domain Names (CNAMEs):** Enter the domain names you would like to redirect to Cloudfront, one at each line. If you don't specify your domains here, you will get an **Access Denied** error.

	    cloudfront.mysite.com
	    www.mysite.com
	    mysite.com

* **Default Root Object:** *index.html* (If you don't put this, you will get access denied unless you provide full path to the html file.)

Then press **Create Distribution**. In the distributon list, you will see Domain Name column. We will use the domain name written here (*abcdefghi.cloudfront.net*) on Route53. It may take some time to see the system has been fully deployed.

## Route53

Add *A record* for your Hosted Zone.

* **Name:** cloudfront.mysite.com.
* **Type:** *A - IPv4 Address*
* **Alias:** *Yes*.
* **Alias Target:** *abcdefghi.cloudfront.net.*

Then press **Create**. It may take some time until cloudfront.mysite.com is propagated to DNS servers. When Cloudfront is deployed and enabled, Route 53 A records are active, you will see your website on *cloudfront.mysite.com*

Check if your site is served through CloudFront:

```bash
curl -svo /dev/null http://www.mysite.com
```

Check if you can access subdirectories like *http://cloudfront.mysite.com/blog/* after placing an index.html in it.

If it works, create a similar A Record for www.mysite.com and mysite.com


## Going Back to S3

If you wish to stop using Cloudfront and go back to S3 serving, update www.mysite.com A record alias. Delete the URL written there, then a list will appear. Choose your S3 bucket (www.mysite.com) and **Save Record Set**.

## Restricting S3 access to Cloudfront

You may want to allow only CloudFront to access your S3 Bucket so that DDoS attackers can't attack on your S3 bucket's web endpoint, causing you thousands of dollars.

### Cloudfront

Click on your distribution, navigate to **Origins** tab, choose origin in the list and press **Edit**. Add a new header on **Origin Custom Headers**.

* **Header Name:** User-Agent
* **Value:** abcdefghi123 (replace this with something random, this will be the password)

Now Amazon Cloudfront will add *User-Agent: abcdefghi123* on each request it is forwarding to Amazon S3.

### Amazon S3

On S3 permissions, click Bucket Policy Editor and fill is as the following. This will limit the resources to AWS network and allow only requests from User Agent "abcdefghi123". Only the ones who provide this password through User-Agent will be able to access the bucket.


```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicReadGetObject",
			"Effect": "Allow",
			"Principal": {
				"AWS": "*"
			},
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::www.mysite.com/*",
			"Condition": {
				"StringEqualsIgnoreCase": {
					"aws:UserAgent": "abcdefghi123"
				}
			}
		}
	]
}
```
