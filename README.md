# Adonis Sink

Adonis sink contains fake implementations for commonly used AdonisJs providers. You can use this package to write tests for your own providers.

[![NPM Version][npm-image]][npm-url]
[![Build Status][travis-image]][travis-url]
[![Appveyor][appveyor-image]][appveyor-url]

<img src="http://res.cloudinary.com/adonisjs/image/upload/q_100/v1497112678/adonis-purple_pzkmzt.svg" width="200px" align="right" hspace="30px" vspace="30px">

It contains fake implmentations for

1. Config provider
2. Helpers provider
3. Logger provider
4. Env provider
5. Setup `adonis-fold` resolver.

## Config provider
If your provider has a dependency on **Config provider**, you must use a fake implementation to write tests.


```js
const { Config } = require('adonis-sink')
const test = require('japa')

test.group('My Package', (group) => {
  group.beforeEach(() => {
    this.config = new Config()
  })


  test('test', () => {
    this.config.set('services.redis', {
      host: '',
      port: ''
    })

    const redis = new Redis(this.config)
  })

})
```

Then inside your `Redis` class you can make use of the `Config.get` method.

```js
class Redis {
  constructor (config) {
    const redisConfig = config.get('services.redis')
  }
}
```

## Helpers Provider
Helpers provider is used to get path to certain application directories. All the methods from the actual provider are available in the fake implementation.

```js
const { Helpers } = require('adonis-sink')
const test = require('japa')
const path = require('path')

test.group('My Package', (group) => {
  group.beforeEach(() => {
    this.helpers = new Helpers(path.join(__dirname, './'))
  })
})
```

The `Helpers` provider needs the `appRoot` as the constructor argument.

## Logger Provider
Logger provider is used to log messages. The fake implementation also let you verify whether a message for a given level was logged or not.

```js
const { Logger } = require('adonis-sink')
const test = require('japa')
const path = require('path')

test.group('My Package', (group) => {
  group.beforeEach(() => {
    this.logger = new Logger()
  })

  test('my test', () => {
    const someModule = new SomeModule(this.logger)
    someModule.connect()

    assert.isTrue(this.logger.has('warn', 'Consider passing 127.0.0.1 over localhost'))
  })
})
```

And use it like this

```js
class SomeModule {
  constructor (logger) {
    this.logger = logger
  }

  connect () {
    if (this.config.host = 'localhost') {
      this.logger.warn('Consider passing 127.0.0.1 over localhost')
    }
  }
}
```

For complex message, you may use `logger.contains` over `logger.has`, since `logger.has` checks the equality of 2 strings and `logger.contains` does a sub string check.

```js
this.logger.debug('user profile %j', { name: 'virk' })
this.logger.contains('user profile')
```

## Env Provider

Also env provider can be used to read environment variables. The fake implementation doesn't load any `.env` file, whereas you can set values manually before writing tests.

```js
const { Env } = require('adonis-sink')

const env = new Env()

env.set('HOST', '127.0.0.1')
```

## Resolver Setup

The `resolver` is an object to make namespaces and resolve values for a given namespace based upon the directory structure and autoloaded namespace.

Setup of `resolver` is done before AdonisJs application boots, but at the time of writing tests, there is no application and hence you can **setup the resolver** using a pre-configured method.

```js
const { setupResolver } = require('adonis-sink')

test.group('My Package', (group) => {
  group.before(() => {
    setupResolver()
  })
})
```

Now your application code under the test can make use of the `adonis-fold` resolver to resolve dependencies.

```js
const { resolver } = require('adonis-fold')
resolver.for('httpControllers').resolveFunc('HomeController.render')

// returns { instance: HomeController, isClosure: false, method: render }
```

[appveyor-image]: https://img.shields.io/appveyor/ci/thetutlage/adonis-sink/master.svg?style=flat-square

[appveyor-url]: https://ci.appveyor.com/project/thetutlage/adonis-sink

[npm-image]: https://img.shields.io/npm/v/adonis-sink.svg?style=flat-square
[npm-url]: https://npmjs.org/package/adonis-sink

[travis-image]: https://img.shields.io/travis/adonisjs/adonis-sink/master.svg?style=flat-square
[travis-url]: https://travis-ci.org/adonisjs/adonis-sink