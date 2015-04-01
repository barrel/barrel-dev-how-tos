### Barrel Development How-Tos

Auto Deployment Tutorials
------------------
- [Wes' Script](#wes-script)

### Wes' Script
The steps below outline automated deployment using Wes' php based, GitHub Webhook script.

#### Installing deploy script
1. Download script from Wes' Gists library: https://gist.github.com/wturnerharris/5ca78f5814085fbc08b0
2. Install deploy.php on web accessible directory
3. Make sure the apache user has permission to execute the script (discussed further in the 'Setting up the Web Hook' step)

#### Retrieving Hased Value
1. Replace the value inside the md5 function with a string specific to the project you’ll be deploying
2. Copy the value you placed inside the md5 function and run it through an md5 hash generator: http://www.miraclesalad.com/webtools/md5.php


#### Settup up the Web Hook
1. Depending on your server environment apache may be running under one of a number of usernames. You can use the following php to determine the name of the user that will be executing the auto deployment script:
```php
<?php echo exec('whoami'); ?>
```
2. If you’re already logged in as this user you can simply generate an ssh key using the command “ssh-keygen”.
3. Otherwise, log in as the user first: sudo su <apache username> ... ssh-keygen
4. Click the settings tab on the repo you’ll be adding the web hook to, and enter the user’s ssh key in the section labeled “Deploy Keys"
5. In the section labeled “Webhooks & Services” paste the url to the deploy.php file you recently added to the staging environment in the field labeled “Payload URL”.
6. Select “application/x-www-form-urlencoded” as the “Content Type"
7. Paste the hashed value you retrieved in the previous section in the field labeled “Secret”.

#### Accommodating a Password Protected Site
1. Prevent tracking of a new directory you’re going to make with your .gitignore file
2. Create this directory for the deploy script
3. Add the deploy script to the directory
4. Modify the path in the deploy script to point to its parent directory
5. Add the following to an .htaccess file to override the basic auth you’ve established in your web root:
```
Satisfy Any
Order Allow,Deny
Allow from all
```

#### Machine Users
If you require multiple auto-deployments on the same server you’ll have to create a machine user so you can assign it to projects with a normal github ssh key instead of a deploy key: https://developer.github.com/guides/managing-deploy-keys/#machine-users
