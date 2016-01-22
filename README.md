# School
Learn Till Now...
#Larave Setup

##Installation
###Command
```
composer create-project --prefer-dist laravel/laravel blog
```
###Directory Permissions
```
<YoutProjecName>sudo chmod -R 777 storage

```
**Create Virtual Host Files and set DocumentRoot value public folder of your project(<YoutProjecName>)**

###Reload apache service and run your site at browser

#Authentication Scaffolding

```
php artisan make:auth
```
***This command will generate plain, Bootstrap compatible views for user login, registration, and password reset. The command will also update your routes file with the appropriate routes.***
###Note: This feature is only meant to be used on new applications, not during application upgrades.

##Migration of Auth
Run below command to create user table(users) and password reset table(password_resets) structure in the database.
```
php artisan migrate 
````
Now your basic app setup.This ist the time to check your site at browser.




