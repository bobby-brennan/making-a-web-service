# Serverless Web Hosting with AWS

This is a set of instructions for deploying a web service using AWS tools.

For the frontend, we will:
* Register a domain with Route 53
* Set up e-mail for the new domain using Google Apps for Work
* Get a free SSL certificate from AWS Certificate Manager
* Host static content (HTML, JS, CSS) on Amazon S3
* Create a CloudFront formation that associates the SSL cert with the S3 bucket

For the backend, we will:
* Create a MySql or DynamoDB database on AWS 
* Create a [serverless](/serverless/serverless) project to contain backend logic
* Use serverless to deploy to AWS Lambda and AWS API Gateway

## Register a domain
https://console.aws.amazon.com/route53/home?region=us-east-1

## Set up e-mail
We'll use gmail to manage email.

Sign up for Google Apps for Work:
https://gsuite.google.com/index.html

Google will ask you to verify that you own the domain you registered on Route 53.
Choose "Other" from the list of providers, and they'll give you a TXT record to add.

To do this:
* Go back to Route 53
* click your new domain
* click "Add Record"
* select TXT
* paste in the value Google gave you
* Go back to Google and click "verify"

Once that's done, you'll need to add MX records so that Route 53 forwards your mail to GMail:
* Go back to Route 53
* click your new domain
* click "Add Record"
* select MX
* paste in the value below

```
1 ASPMX.L.GOOGLE.COM.
5 ALT1.ASPMX.L.GOOGLE.COM.
5 ALT2.ASPMX.L.GOOGLE.COM.
10 ALT3.ASPMX.L.GOOGLE.COM.
10 ALT4.ASPMX.L.GOOGLE.COM.
```
