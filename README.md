# Pimcore Codeception Config Files

**Pimcore Files For Codeception Test Environment Configuration**

## Install
Run this command
```
composer require sl0wlydeadly/pimcore-codeception-config-files
```

## DB Configuration

- Log into your MariaDB client as root

```
mysql -u [root username] -p[root password]
```

- Create a different DB for tests to run i.e. pimcore_test

```
CREATE DATABASE pimcore_test;
```

- Give your pimcore user priviledges to that DB

```
GRANT ALL PRIVILEGES ON pimcore_test.* TO 'pimcore'@'%';
```

## Setup test config in your Pimcore project

- [Example] we have a docker DB container at port 3306 called db and a test DB called pimcore_test then the doctrine dbal part of config under **config/packages/test/config.yaml** should look like this:

```
doctrine:
    dbal:
        connections:
            default:
                host: db
                port: '3306'
                dbname: pimcore_test
                user: pimcore
                password: pimcore
```
## Setup your bootstrap.php according to Pimcore documentation for codeception tests
- Mine looks something like this:
```
<?php

// tests/_bootstrap.php

use App\Kernel;
use Pimcore\Bootstrap;

// define project root which will be used throughout the bootstrapping process
define('PIMCORE_PROJECT_ROOT', realpath(__DIR__ . '/..'));

const PROJECT_ROOT = PIMCORE_PROJECT_ROOT;

$kernel = Kernel::class;

// set the used pimcore/symfony environment
foreach (['APP_ENV' => 'test', 'PIMCORE_SKIP_DOTENV_FILE' => true] as $name => $value) {
    putenv("{$name}={$value}");
    $_ENV[$name] = $_SERVER[$name] = $value;
}

putenv("KERNEL_CLASS={$kernel}");
$_ENV["KERNEL_CLASS"] = $kernel;

require_once PIMCORE_PROJECT_ROOT . '/vendor/autoload.php';

Bootstrap::setProjectRoot();
Bootstrap::bootstrap();
Bootstrap::kernel();
```

## Setup your codeception configurations

- In the root path of your Pimcore project you should have a **tests** directory.
- In **tests** directory make sure you have a **codeception.dist.yml** configuration file. You may modify and adjust the content to your preferences but as default it should look something like this:
```
# tests/codeception.dist.yml

namespace: Tests
support_namespace: Support
actor_suffix: Tester
paths:
    tests: .
    output: ./_output
    data: ./Support/Data
    support: ./Support
    envs: ./_envs
settings:
    bootstrap: _bootstrap.php
    colors: true
params:
    - env
extensions:
    enabled:
        - Codeception\Extension\RunFailed
```
- Set up your Test suite config to use the package data to recreate the test database after a test run. Below example for Unit test suite configuring **Unit.suite.yml** should look something like that:
```
suite_namespace: \App\Tests\Unit
actor: UnitTester
modules:
    enabled:
        - Asserts
        - \Sl0wlydeadly\PimcoreCodeceptionConfigBundle\Support\Helper\Pimcore:
            # CAUTION: the following config means the test runner
            # will drop and re-create the Pimcore DB and purge var/classes
            # use only in a test setup (e.g. during CI)!
            connect_db: true
            initialize_db: true
            purge_class_directory: false
            # If true, it will create database structures for all definitions
            setup_objects: false
```
For more config info read the official Pimcore documentation under Codeception tests

## Build your codeception config

Run following command:
```
php vendor/bin/codecept build
```

## Run tests of your configured test suite using test as environment

```
php vendor/bin/codecept run -c . Unit --env=test
```