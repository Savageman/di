# Orno\Di

[![Build Status](https://travis-ci.org/orno/di.png?branch=master)](https://travis-ci.org/orno/di) [![Latest Stable Version](https://poser.pugx.org/orno/di/v/stable.png)](https://packagist.org/packages/orno/di) [![Total Downloads](https://poser.pugx.org/orno/di/downloads.png)](https://packagist.org/packages/orno/di)

Orno\Di is a small but powerful dependency injection container that allows you to decouple components in your application in order to write clean and testable code. The container can automatically resolve dependencies of objects resolved through it.

## Installation

Add `orno/di` to your `composer.json`.

```
{
    "require": {
        "orno/di": "2.*"
    }
}
```

Allow Composer to autoload the container.

```
<?php

include 'vendor/autoload.php';
```

## Usage

### Constructor Injection

The container can be used to register objects and inject constructor arguments such as dependencies or config items.


For example, if we have a `Session` object that depends on an implementation of a `StorageInterface` and also requires a session key string. We could do the following:

```
class Session
{
    protected $storage;
    protected $sessionKey;
    public function __construct(StorageInterface $storage, $sessionKey)
    {
        $this->storage    = $storage;
        $this->sessionKey = $sessionKey;
    }
}

interface StorageInterface
{
    // ..
}

class Storage implements StorageInterface
{
    // ..
}

$container = new \Orno\Di\Container;

$container->add('Storage');

$container->add('session', 'Session')
          ->withArgument('Storage')
          ->withArgument('my_super_secret_session_key');

$session = $container->get('session');
```

### Setter Injection

If you prefer setter injection to constructor injection, a few minor alterations can be made to accommodate this.

```
class Session
{
    protected $storage;
    protected $sessionKey;
    public function setStorage(StorageInterface $storage)
    {
        $this->storage = $storage;
    }
    public function setSessionKey($sessionKey)
    {
        $this->sessionKey = $sessionKey;
    }
}

interface StorageInterface
{
    // ..
}

class Storage implements StorageInterface
{
    // ..
}

$container = new Orno\Di\Container;

$container->add('session', 'Session')
          ->withMethodCall('setStorage', ['Storage'])
          ->withMethodCall('setSessionKey', ['my_super_secret_session_key']);

$session = $container->get('session');
```

This has the added benefit of being able to manipulate the behaviour of the object with optional setters. Only call the methods you need for this instance of the object.

### Automatic Dependency Resolution

Orno\Di has the power to automatically resolve your objects and all of their dependencies recursively by inspecting the type hints of your constructor arguments. Unfortunately, this method of resolution has a few small limitations but is great for smaller apps. First of all, you are limited to constructor injection and secondly, all injections must be objects.

```
class Foo
{
    public $bar;
    public $baz;
    public function __construct(Bar $bar, Baz $baz)
    {
        $this->bar = $bar;
        $this->baz = $baz;
    }
}

class Bar
{
    public $bam;
    public function __construct(Bam $bam)
    {
        $this->bam = $bam;
    }
}

class Baz
{
    // ..
}

class Bam
{
    // ..
}
```

In the above code, `Foo` has 2 dependencies `Bar` and `Baz`, `Bar` has a further dependency of `Bam`. Normally you would have to do the following to return a fully configured instance of `Foo`.

```
$bam = new Bam;
$baz = new Baz;
$bar = new Bar($bam);
$foo = new Foo($bar, $baz);
```

With nested dependencies, this can become quite cumbersome and hard to keep track of. With the container, to return a fully configured instance of `Foo` it is as simple as requesting `Foo` from the container.

```
$container = new \Orno\Di\Container;

$foo = $container->get('Foo');
```
