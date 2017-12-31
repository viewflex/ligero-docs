
# Tests

## Setup

All tests extend the `TestCase` class installed with Laravel. When using Laravel 5.4 or greater, this line is necessary in each of the tests. With prior Laravel versions, this line must be commented out before trying to run the tests:


```php
    use Tests\TestCase;
```

Before running tests, add database connection in config/database.php:

    'sqlite_testing' => [
         'driver' => 'sqlite',
         'database' => database_path('testing/testing.sqlite'),
         'prefix' => '',
     ],


On first time or if you deleted the sqlite database, create it:

	touch database/testing/testing.sqlite


## Running Tests

Run the `phpunit` executable from the `ligero/tests` directory. You may have installed phpunit elsewhere, but to run the copy installed with Laravel, execute this command:

	./../../../../vendor/bin/phpunit

## Running Individual Test Suites

The `testsuite` phpunit option allows tests suites, as defined in the `phpunit.xml` file, to be run individually.

	--testsuite="Environment"
	--testsuite="Unit"
	--testsuite="Integration"
	--testsuite="Functional"
	--testsuite="Context"