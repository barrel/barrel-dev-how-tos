### Barrel Development How-Tos

CDN Tutorials
------------------
- [CloudFront](#cloudfront)
- [Provisioning](#provisioning)
- [Integration](#Integration)

### CloudFront
The steps below outlines a basic workflow to set up a CloundFront CDN, using S3 and W3 Total Cache

#### AWS User setup. 
In order to setup access for both S3 and the [S3Tools command line client](https://github.com/s3tools/s3cmd), we need to supply both installations with credentials; an Access Key ID and a Secret Access Key. Rather than using the keys attached to the root user of the AWS account, it's preferable to create a new AIM user with restricted access. To do this, follow the following steps:
1. Login to the [AWS Console](https://console.aws.amazon.com) and [Create a new user](https://console.aws.amazon.com/iam/home).
2. Download the two keys attached to this user and save them in a secure location. Note: This is the only time you will be provided with the Secret Access Key.
3. Attach a user policy to the user under the "Permissions" tab. This allows access to a specified S3 bucket. We use policy statements to specifically set user permissions settings. The code below provides a user access to <bucket> while also allowing the user to view all buckets attached to this account. This second statement is necessary for W3 Total Cache and S3Tools. Once the statements have been added, save all changes.
```javascript 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::<bucket>",
        "arn:aws:s3:::<bucket>/*"
      ]
    }
  ],
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "arn:aws:s3:::*"
    }
  ]
}
```
#### S3 Bucket Setup
1. Navigate to [S3](https://console.aws.amazon.com/s3/home) and create a new bucket.
2. Once complete, click on the "Properties" tab on the right and open the "Permissions" tab.
3. Open the "CORS Configuration Editor". Here, we're going to specify origin permissions to ensure that we allow cross-origin resource sharing between the server URI that hosts the site and the S3 bucket that will act as the origin for static resources. The code below ensures that origin example.com is allowed to access assets from this bucket. This is particularly important for font files as technically, fonts are interpreted as scripts in standards compliant browsers. Once the configuration has been added, save all changes.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedOrigin>http://example.com</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Content-*</AllowedHeader>
        <AllowedHeader>Host</AllowedHeader>
    </CORSRule>
    <CORSRule>
        <AllowedOrigin>http://*.example.com</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Content-*</AllowedHeader>
        <AllowedHeader>Host</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```
#### Cloudfront Setup
1. Navigate to [Cloudfront](https://console.aws.amazon.com/cloudfront/home) and create a new distribution.
2. Click the "Get Started" button under "Web".
3. The settings below may change for different setups. For the purpose of this readme, settings are detailed below. Note: I am only highlighting the settings that need to be altered.
  - Origin Domain Name: <S3 Bucket>
  - Origin Path: Empty (Root directory)
  - Origin ID: Should have already been set when selecting "Origin Domain Name"
  - Forward Headers: Select "Whitelist" and add "Origin"
  - I leave TTL unchanged (with a value of 0) so it will use the origin's Cache-Control headers that we will set later. Note. if the origin's Cache-Control headers are less than 24 hours, CloudFront will cache assets for 24 hours if this value is kept at 0.
4. Click Create.
5. Note the Domain Name of the distrubition for future reference (i.e. d3c2yng6mv3662.cloudfront.net).

#### W3 Total Cache
1. In the Wordpress backend, activate the W3 Total Cache Plugin if not done already.
2. Navigate to Performance > General Settings > CDN.
3. Enable the CDN and select CDN type: Amazon CloudFront.
4. Save all settings.
5. Navigate to the CDN sub-menu item and insert the credentials we downloaded when we created the AIM user.
6. Type the bucket name of the S3 bucket we previously created.
7. Replace the site's hostname with the CloudFront domain name (i.e. d3c2yng6mv3662.cloudfront.net).
8. Go down to "Theme file types to upload and change the "," infront of ".woff" to ";" - this is a typo and stops anything following this comma to upload to S3. Add "*.eot" to the list.
9. Save all settings.
10. Go to the top of this CDN settings page and click "Upload Attachments" and "Theme Files". This will migrate ```wp-content/uploads``` and ```wp-content/theme``` static media to S3. This can be updated and purged at anytime.

#### S3Tools
This is a command line tool that allows us to recursively set headers for S3 content (among other actions). We will use it here to set Cache-Control:max-age headers for all content uploaded. Cloudfront does not override the response headers for content stored on S3 (or other origins) so it's necessary to set these directly on S3. This tool allows use to do this efficiently.
  1. Download a ZIP of the [S3Tools Repo](https://github.com/s3tools/s3cmd).
  2. cd to the unzipped root directory of the download.
  3. Install the tool with ```sudo python setup.py install```.
  4. run s3cmd --configure with the following settings (You may choose to change these for you own setup. Note: AWS credentials can be set as environmental variables which is a more adaptable way of setting these details but it does require a number of extra steps)
  - Encryption password: Set your own password
  - Path to GPG program: Run "which gpg" in a different terminal window as paste the result*
  - Use HTTPS protocol: I selected "No" but this is up for debate
  - Proxy settings are left blank

With the S3 tool now set up, you can use it to alter header values for selected S3 buckets. The following command adds or alters the Cache-Control:max-age with a value of 1 day, or 86400 seconds. This is how long CloudFront will cache the asset before contacting the origin again for a new version of the asset. 86400 is used here as an example. We will need to decide on an appropriate value internally. Note: WebPageTest provides a warning for any values under 2592000, or 30 days.

*if command ```which gpg``` returns nothing, you will need to download the gpg program at (gpgtools.org)[https://gpgtools.org]


### Provisioning

### Integration
