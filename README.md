# Configuring Windows Server 2012 to Host a Laravel Project

During my time at an internship I had to create an internal web application for onboarding/offloading employees. I chose to use the Laravel framework and was provided to host the project on a Windows Server 2012 R2 using IIS as well as MSSQL Server 2012.

## What you'll need

* [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
* [PHP](https://windows.php.net/download#php-7.0)
* [MSSQL Drivers for PHP](https://www.microsoft.com/en-us/download/details.aspx?id=20098)
* [ODBC Driver](https://www.microsoft.com/en-us/download/details.aspx?id=50420)
* [Visual C++ Redistributable Package ](https://www.microsoft.com/en-us/download/details.aspx?id=48145)
* [Composer](https://getcomposer.org/download/)

## IIS (Internet Information Services) Setup
* Opening the Server Manager we head over to the top right corner.
```
Manage -> Add Roles and Features
```
![](/images/1.png)
![](/images/2.png)

* Skipping the Features Tab
* In Role Services under Application Development check CGI

![](/images/3.png)

* Install

You will need to install the extension [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite). I'll be using x64 architecture for my project but please select the appropriate version for you.


## PHP

[PHP](https://windows.php.net/download#php-7.0) 
We will be using the x64 Non Thread Safe version since we will be running PHP in Fastcgi mode. Unzip this into `C:PHP`.

You will need to copy either the production or developmental .ini file and save it to C:Windows. You may need to run your text editor with admin permissions due to the file location
```
extension_dir = “ext” (enable this if you’re going to use extensions)
fastcgi.impersonate = 1 (enable this)
cgi.fix_pathinfo=1 (enable this)
cgi.force_redirect = 1 (enable this)
```
Extract the [MSSQL Drivers for PHP](https://www.microsoft.com/en-us/download/details.aspx?id=20098). Look for the file name php_pdo_sqlsrv_7_nts_x64.dll and move it to `C:PHP\ext` folder. Rename it to php_pdo_sqlsrv.dll.

Install the [ODBC Driver](https://www.microsoft.com/en-us/download/details.aspx?id=50420) to make your project work with SQLSRV. Please remember to install the correct one depending on your OS architecture.

Return to the php.ini file to enable the last of the extensions necessary:
```
php_mbstring.dll
php_openssl.dll
php_odbc.dll
php_pdo_sqlsrv.dll (This one is added manually)
```

### Troubleshooting

If you encounter the `VCRUNTIME140.dll is missing` error, please download the component from [Visual C++ Redistributable Package ](https://www.microsoft.com/en-us/download/details.aspx?id=48145).

![](//images5.png)

## Configure Handler Mapping

Head over to and open `IIS Manager` .  At this point you can highlight the entire server to apply the following changes to all websites or you can just highlight a specefic one.
Go to `Handler Mappings` and on the side click `Add Module Mappings`.

Here is how it should look

![](/images/6.png)

A prompt will ask if you want to create a FastCGI application:

![](/images/7.png)

Click Yes.
## Composer

Install [Composer](https://getcomposer.org/download/) . Composer is a dependency manager and with it you will create a new laravel project with just a single command line.

## Creating a New Laravel Project

Use composer to create a new project using Laravel.
* `cd` where you’d like to store projects then run the following command in the command line:
```
composer create-project laravel/laravel myproject
```
* Once completed, right-click on the test folder and select Properties. Under the Security tab, add `IIS_IUSRS` and `IUSR` with the following permissions.
```
Read & execute
List folder contents
Read
Write
```
* Run Notepad as administrator and open `C:Windows\System32\drivers\etc\hosts`
* Add this line:
```
127.0.0.1                 myproject.dev
```
* Return to IIS and `Add Website`

![](/images/8.png)

Make sure you point to the public folder.

* Check if the web.config imported the values in .htaccess found in the public folder. If not then manually import from the `.htaccess` file in the public folder.
* Open `Default Document` and add `index.php`. You should now be able to view the default Laravel home page by visiting http://myproject.dev in your browser.

## Connecting to MSSQL Server 2012

* Open `/config/database.php` file. Change the default value from `mysql` to `sqlsrv`. Add the following connection:
```
'sqlsrv' => array(
'driver' => 'sqlsrv',
'host' => env('DB_HOST', 'localhost'),
'database' => env('DB_DATABASE', 'forge'),
'username' => env('DB_USERNAME', 'forge'),
'password' => env('DB_PASSWORD', ''),
'prefix' => '',
),
```
* Open the `.env` file and provide your database credentials and information.
* Create the database in MSSQL server. 

If everything works as they should, you can run the `php artisan migrate` command in Command Prompt and it should create the migrations, users, password_resets tables in the database.

Finished!

## Troubleshooting

If viewing your project through Internet Explorer returns a `Page cannot be displayed`  and you there are no errors being displayed in any logs then be sure that in the internet options `Enhanced Protected Mode` is unchecked!
