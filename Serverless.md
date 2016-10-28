# Serverless Web Hosting with AWS

This is a set of instructions for deploying a web service using nothing but AWS tools.
The one exception is that we'll use GMail (via Google Apps for Work) to manage e-mails.

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
Follow instructions at [Route53](https://console.aws.amazon.com/route53/home).
If you've already registered your domain on another registrar, you'll need to
transfer it to Route53 for the instructions below to work.

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

Once you have a user set up in Google Apps for Work, you should be able to receive emals at user@yourdomain.com.

## Get an SSL Cert
We'll use AWS Certificate Manager to get a free SSL cert. As an alternative, you
can check out [Let's Encrypt](https://letsencrypt.org/)

Sadly, even though we registered with Route 53,
you'll need to verify that we have access to the email address admin@yourdomain.com

To set up the e-mail address admin@yourdomain.com:
* Head back to Google Apps for Work
* Click "Users"
* Click your user
* Click "Account"
* Click "Add an Alias"
* Enter "admin" and save.

Now any e-mails to admin@yourdomain.com should get forwarded to user@yourdomain.com (or whatever the primary email is)

Now let's get the certificate.

Head over to [AWS Certificate Manager](https://console.aws.amazon.com/acm/home)
and click "Request a Certificate" and enter your domain. AWS will send you a confirmation email
at admin@yourdomain.com - click the link in the email to confirm.

## Upload your website to S3
Head over to [S3](https://console.aws.amazon.com/s3/home) and create a new bucket.

Then click "Properties" -> "Permissions" -> "Edit Bucket Policy" and enter:

```
{
	"Version": "2008-10-17",
	"Statement": [
		{
			"Sid": "AllowPublicRead",
			"Effect": "Allow",
			"Principal": {
				"AWS": "*"
			},
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::gitway-website-dev/*"
		}
	]
}
```

Then click "Static Website Hosting" and choose "Enable website hosting". Enter "index.html" as the index document.

You can then either upload your HTML/CSS/JS manually, or use the AWS CLI:

```bash
sudo pip install awscli
export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY
export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_KEY
aws s3 sync path/to/your/assets/ s3://your-bucket-name/
```

You can find your key and secret in [AWS IAM](https://console.aws.amazon.com/iam/home)

You should now be able to view your website at http://your-bucket-name.s3-website-us-east-1.amazonaws.com/

## Create a CloudFront Distribution
CloudFront will route traffic from your domain name to your S3 bucket.

Head over to [CloudFront](https://console.aws.amazon.com/cloudfront/home) and create a new distribution.
Under "Web", click "Get Started".

You should now see a form with a bunch of settings.  All the defaults should be good except:

#### Origin Settings
* Origin Domain Name: choose your s3 bucket

#### Default Cache Behavior Settings
* Viewer Protocol Policy: If your website deals with sensitive information, consider redirecting HTTP to HTTPS

#### Distribution Settings
* SSL Certificate: Choose "Custom SSL Certificate" and select the certificate you just got.
* Default Root Object: set this to "index.html" (or whatever file in your s3 bucket has your homepage)


Once your CloudFront distribution is up (takes ~10 minutes), you should be able to view your website at
the domain provided (something.cloudfront.net).

Now we just need to link your domain to the CloudFront distribution. Head back to [Route53](https://console.aws.amazon.com/route53/home)

#### IPv4 Record:
* Select your domain
* Click "Create Record"
* Chose "A" record
* Set "Alias" to "Yes"
* Choose your CloudFormation dist
* Click Save

#### IPv6 Record
* Click "Create Record"
* Choose "AAAA" record
* Set "Alias" to "Yes"
* Choose your CloudFormation dist
* Click Save

#### Alias www to apex
* Click "Create Record"
* Set "name" to "www"
* Choose "CNAME"
* Enter yourdomain.com
* Click Save

**Congrats!** you should now be able to see your website at yourdomain.com


