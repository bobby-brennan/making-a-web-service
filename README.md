# So you wanna make a Website

## Getting a Domain
* Use AWS Route53

## Setting up AWS
### ELB
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
