# IceHawk installation

This documentation shows you how to basically install the IceHawk component.

<hr class="blockspace">

## Interactive installer

We provide an interactive installer package, that can be issued using [composer](https://getcomposer.org)'s `create project` command.

Assuming you already have [downloaded](https://getcomposer.org/download/) the `composer.phar`, simply run:

```bash
php composer.phar create-project icehawk/installer /path/to/new-project
```

The installer will ask you some basic questions about your namings to set up your new project and lets you optionally choose to install further IceHawk components.

<i class="fa fa-youtube"></i> [**Watch our short video and see how it works:** Install IceHawk framework in less than 2 minutes](https://youtu.be/ns62lw52AOU)

The installation results in a minimal, ready-to-use web application project setup 
with one read (GET) and one write (POST) route.

<i class="fa fa-github"></i> [You can help us improve the interactive installer.](https://github.com/icehawk/installer/blob/master/CONTRIBUTING.md)

**Please note:** For reasons of automation the installer initially installs some thrid-party dependencies. These will be removed at the end of the installation process.

<hr class="blockspace">

## Manual installation

### Step 1 - require icehawk/icehawk

If you already have created a `composer.json` for your project, then just add the dependency by running:

```bash
php composer.phar require icehawk/icehawk:^2.0
```

or adding it manually:

```json
{
    "require": {
        "icehawk/icehawk": "^2.0"
    }
}
```
_— composer.json_

and running:

```bash
php composer.phar update
```

### Step 2 - Create a request handler

Assuming your project namespace is `YourVendor\YourProject`. 

```php
<?php declare(strict_types = 1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Interfaces\HandlesGetRequest;
use IceHawk\IceHawk\Interfaces\ProvidesReadRequestData;

final class SayHelloRequestHandler implements HandlesGetRequest
{
    public function handle( ProvidesReadRequestData $request ) 
    {
        echo "Hello World!";   
    }   
}

```
_— SayHelloRequestHandler.php_

### Step 3 - Create a basic config

All you need is at least one read or write route.

```php
<?php declare(strict_types = 1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\Routing\ReadRoute;
use IceHawk\IceHawk\Routing\Patterns\Literal;
use YourVendor\YourProject\SayHelloRequestHandler;

final class IceHawkConfig extends \IceHawk\IceHawk\Defaults\IceHawkConfig
{
    public function getReadRoutes() 
    {
        return [
            new ReadRoute( new Literal('/'), new SayHelloRequestHandler() ),    
        ];
    }
}

```
_— IceHawkConfig.php_

### Step 4 - Create a bootstrap script

```php
<?php declare(strict_types = 1);

namespace YourVendor\YourProject;

use IceHawk\IceHawk\IceHawk;
use IceHawk\IceHawk\Defaults\IceHawkDelegate;

require('vendor/autoload.php');

$iceHawk = new IceHawk(new IceHawkConfig(), new IceHawkDelegate());
$iceHawk->init();

$iceHawk->handleRequest();

```
_— index.php_


If the `index.php` is the bootstrap script of your domain you should see _Hello World!_ when visiting `http://your.domain/`.

**That's it.**

