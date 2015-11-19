# So you wanna make a Website

## Setting up AWS
Here we'll register a domain, allocate machines to run the site, and set up SSL

### ELB
* In AWS, create a new VPC
* Create one or more new ec2 machines (Ubuntu preferred) in that VPC
** note: reserved instances are cheaper, but require a 1yr commitment
* Under ec2 page, create a loadbalancer
* Add your machine to the loadbalancer
* Set up a listener that forwards port 80 to port 3000
* Add a healthcheck that hits port 3000

### Domain
* In AWS, head to Route53 and register a domain
* Set up an A Record (Alias) that points the root domain to your loadbalancer
* Set up a CNAME that points www to root

### SSL
* Head over to godaddy.com
* Purchase an SSL cert
* Generate a CSR
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout yourdomain.key -out yourdomain.csr
```
* Upload .csr file to GoDaddy, save .key
* Use Route53 to add a TXT record with the code GoDaddy sends you
* Once confirmed, click "Download" from GoDaddy, use "Apache"
* Open your loadbalancer in AWS, click "edit listeners"
* Add a rule to forward HTTPS (443) to 3000
* Add certificates using copy/paste:
** Private key is yourdomain.key generated above
** Public key is $randomID.crt from GoDaddy download
** Certificate Chain is gd_bundle-g2-g1.crt
* (optional) redirect HTTP to HTTPS:
```js
if (!process.env.DEVELOPMENT) {
  App.use(function(req, res, next) {
    var isSecure = req.secure ||
        req.headers["x-forwarded-proto"] === "https" ||
        req.headers['user-agent'].indexOf('ELB-HealthChecker') !== -1;
    if (isSecure) return next();
    var redirectUrl = 'https://yourdomain.com' + req.originalUrl;
    return res.redirect(redirectUrl);
  })
}
```
## MongoDB

## NodeJS
### Passport

## Angular

## 
