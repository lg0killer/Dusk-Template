# Browser Testing

This repository contains the structure to create and execute Automated Browser tests using [Laravel Dusk](https://laravel.com/docs/master/dusk)

# Table of Contents
1. [Getting started](#Getting-Started)

    a. [Prechecks](#Prechecks)

    b. [Installing Dependencies](#Installing-Dependencies)

    c. [Updating the .env file](#Updating-the-.env-file)

2. [Test Cases](#Test-Cases)

    a. [Create a new Test Case](#Create-a-new-Test-Case)

    b. [Filling Form Data](#Filling-Form-Data)

    c. [Reusing Tests](#Reusing-Tests)

3. [Pages](#Pages)

    a. [Create a new page](#Create-a-new-page)

    b. [Importing Pages to Test Cases](#Importing-Pages-to-Test-Cases)

4. [Running Tests](#Running-Tests)

    a. [Run all Tests](#Run-all-Tests)

    b. [Run a group of tests](#Run-a-group-of-tests)

    c. [Running tests in phpunit](#Running-tests-in-phpunit)

    d. [Screenshots on failure](#Screenshots-on-failure)

    e. [Reports using Laravel](#Reports-using-Laravel)

5. [References](#References)

# Getting started

Tested with php 7.4.10

## Prechecks

In order to run the tests locally, ensure the following:

* Ensure you have a chrome/chromium instance installed(Version > 83.5)
* Edit Tests/.env to your environment-specific information

## Installing Dependencies
Install dependencies
```console
composer install                        # This will install the tools for artisan
php artisan dusk:install                # This will install dusk and relevant dependencies
php artisan dusk:chrome-driver --detect # This will install the correct driver for your chrome instance
```

Composer will ensure all the relevant items are available in the vendor/ directory.
Artisan is the Laravel runtime engine and will execute the appropriate commands for the Laravel suite (in our situation Dusk).

## Updating the .env file
The .env file contains all of the variables that might change swapping from different environments. Typically you would set it up to your environment and have the settings ready for another environment.

Variables to take note of:
* APP_URL           : This variable is used to define the path to the Loadbalancer application
* Balancer_Address  : This variable is used to define the path to the Loadbalancer backend/frontend services (HAProxy/Nginx)
* webserver1_ip     : The IP of the testing server used during the connection tests
* webserver1_port   : The Port of the testing server used during the connection tests
* LicenseUsername   : By default not included. Needs to be added by the user for security purposes
* LicensePassword   : By default not included. Needs to be added by the user for security purposes

# Test Cases
The test cases are stored in tests/Browser/ with a suffix of Test.php. These items are executed in alphabetical order or numerical (whichever provided first)

## Create a new Test Case
A new test case can be executed using the following command:
```console
php artisan dusk:make newDuskTest
```

This will create a new test case: tests/Browser/newDuskTest

The new file provides a frame to build out your test cases into.

## Filling Form Data

Dusk provides a variety of wrapper methods around webdriver to easily interact with form elements. In this article we will review these methods.

### Clicking objects

To click buttons, you can use the click method. This requires the selector of the clickable object
```php
$browser->click('#balancer_section > table > tbody > tr:nth-child(5) > td > button:nth-child(1)');
```

Click can also be extended to go to xpath locations or links:

* [clickAtXpath](https://laravel.com/docs/master/dusk#using-the-mouse)()  --> allows a xpath location to be specified

* [clickLink](https://laravel.com/docs/master/dusk#clicking-links)()     --> clicks buttons going to a specific link (Needs to be the sub-path and not full link)

* [clickAtPoint](https://laravel.com/docs/master/dusk#using-the-mouse)()  --> Clicks at spcific locations on the page.


### Filling Text Fields
To type value in the text field, you can make use of type method.

```php
$browser->type('firstname' , 'John');
```

This method will simulate the action of user typing value John in the input field named firstname with the keyboard.

If you are looking to append value in the field, you can make use append method.

```php
$browser->append('firstname' , 'Mayor');
```

If you are looking to clear out any already filled value from the fields, make use of clear method

```php
$browser->clear('email');
```

### Selecting Dropdowns
Dusk provides a straightforward method select to work with select boxes.

To select any random value from the dropdown

```php
$browser->select('state');
```

This will select any random state value from the state dropdown.

If you want to select any specific value from the dropdown, you can pass in a second parameter in the select method.

```php
$browser->select('state', 'NC');
```

### Selecting Checkboxes
To select the checkbox you can make use of check method

```php
$browser->check('terms');
$browser->uncheck('terms');
```

### Selecting Radio Buttons
To select the radio button you can make use of radio method, radio method does not give you can option to select radm radio button from a group.

Thus you have to pass the value of which button you are looking to select

```php
$browser->radio('gender', 'Male);
```

## Reusing Tests
The easiest way to repeatedly use a previously created test is with the help of functions. A function can be created within a Test Case (Limited to only that file) or a Page (Sharable between different Test Cases)

# Pages
Laravel Dusk has a concept called pages, which helps you organize your browser automation tests. When you are working with a browser test and if the functionality / feature to be tested is complex that spans over multiple pages, then usually the test code becomes cumbersome and messy.

This is where the concept of pages in Laravel Dusk comes in. Dusk pages helps you organize the automation code of pages in your application in different page classes. And then different methods of the page can be called with a single line of code in your main test class.

## Create a new page
You can generate a new page using the following:

```console
php artisan dusk:page Checkout
```

This will create a new Page: tests/Browser/Pages/Checkout

Pages by default will import all it's required tools from the page.php file in the same directory. If you want to create a grouping of pages you will need to generate your own page.php file and amend the import paths.

## Importing Pages to Test Cases
Pages can then be imported to Test Cases using the ``use `` keyword:
```php
use Tests\Browser\Pages\Login;
```

This will enable you to start using the login page and all the functions included.

To start using a page's function you need to create the page object, either by a visit, on or new keyword.

The easies method will be to use the following:

```php
use Tests\Browser\Pages\Login;

...

public function testLogin()
    {
        $this->browse(function (Browser $browser) {            
            $browser->visit(new Login())
                    ->Login_User(env('Correct_User'), env('Correct_Password'));
        });
    }
```

This will ensures the URL you intend to visit/use is correct and unlocks the functions for that page grouping


# Running Tests
Tests can be executed multiple ways

## Run all Tests
To run all tests in chronological order:
```console
php artisan dusk
```

This will run all the tests and continue even if an error/failure has occured

## Run a group of tests
To run specific tests you need to add a decorator to the collection of tests you want to group:

```php
    /**
     * @group accelerator
     * @group setuptests
     * @group cleanup
     */
    public function testLogin()
    {
        $this->browse(function (Browser $browser) {            
            $browser->visit(new Login())
                    ->Login_User(env('Correct_User'), env('Correct_Password'));
        });
    }
```

This test is included in 3 different tests and, due to being the first test in the php file, will always execute first

To run a group:
```console
php artisan dusk --group accelerator
```

This will execute all the items within the accelerator group

## Running tests in phpunit
PHPUnit can be used to execute tests.

To run tests using phpunit it is important to remember to set the configuration file:
```console
./vendor/bin/phpunit --configuration ./phpunit.dusk.xml
```

This will execute all the tests in chronological order

## Screenshots on failure
Dusk will automatically take screenshots on any failed tests.

These items are stored in tests/Browser/screenshots.

## Reports using Laravel
Generating a simple html report using the testdox flag. This creates a very basic checklist on failures.

```console
php artisan dusk --testdox-html report.html
```

# References

[Official Laravel Dusk documentation](https://laravel.com/docs/master/dusk)

[5 Balloons](https://www.5balloons.info/introduction-to-laravel-dusk/)

[Laracast](https://laracasts.com/browse/testing)