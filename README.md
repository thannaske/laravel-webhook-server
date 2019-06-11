**WORK IN PROGRESS: DO NOT USE (YET)**

# Send webhooks from Laravel apps

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/laravel-webhook-server.svg?style=flat-square)](https://packagist.org/packages/spatie/:package_name)
[![Build Status](https://img.shields.io/travis/spatie/laravel-webhook-server/master.svg?style=flat-square)](https://travis-ci.org/spatie/:package_name)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/laravel-webhook-server.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/:package_name)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/laravel-webhook-server.svg?style=flat-square)](https://packagist.org/packages/spatie/:package_name)

A webhook is a way for an app to provide information to another app about a certain event. The way the two apps communicate is with a simple HTTP request. 

This package allows you to easily configure and send webhooks in a Laravel app. It has support for signing calls, retrying calls and backoff strategies.

If you need to receive and process webhooks take a look at our [laravel-webhook-client] package.

## Installation

You can install the package via composer:

```bash
composer require spatie/laravel-webhook-server
```

You can publish the config file with:
```bash
php artisan vendor:publish --provider="Spatie\WebhookServer\WebhookServerServiceProvider"
```

This is the contents of the file that will be published at `config/webhook-server.php`:

```php
return [

    /*
     *  The default queue that should be used to send webhook requests.
     */
    'queue' => 'default',

    /*
     * The default http verb to use.
     */
    'http_verb' => 'post',

    /*
     * This class is responsible for calculating the signature that will be added to
     * the headers of the webhook request. A webhook client can use the signature
     * to verify the request hasn't been tampered with.
     */
    'signer' => \Spatie\WebhookServer\Signer\DefaultSigner::class,

    /*
     * This is the name of the header where the signature will be added.
     */
    'signature_header_name' => 'Signature',

    /*
     * These are the headers that will be added to all webhook requests.
     */
    'headers' => [],

    /*
     * If a call to a webhook takes longer that this amount of seconds
     * the attempt will be considered failed.
     */
    'timeout_in_seconds' => 3,

    /*
     * The amount of times the webhook should be called before we give up.
     */
    'tries' => 3,

    /*
     * This class determines how many seconds there should be between attempts.
     */
    'backoff_strategy' => \Spatie\WebhookServer\BackoffStrategy\ExponentialBackoffStrategy::class,

    /*
     * By default we will verify that the ssl certificate of the destination
     * of the webhook is valid.
     */
    'verify_ssl' => true,
];
```

By default the package uses queues to retry failed webhook requests. Be sure to setup a real queue other that `sync` in non-local environments.

## Usage

This is the simplest way to call a webhook:

```php
Webhook::create()
   ->url('https://other-app.com/webhooks')
   ->payload(['key' => 'value'])
   ->signUsingSecret('sign-using-this-secret')
   ->call();
```

This will send a post request to `https://other-app.com/webhooks`. The body of the request will be json encoded version of the array passed to `payload`. The request will have a header called `Signature` that will contain a signature the receiving app can use [to verify](TODO: add link) the payload hasn't been tampered with. 

If the receiving app doesn't respond with a response code starting with `2`, the package will retry calling the webhook after 10 seconds. If that second attempt fails, the package will attempt to call the webhook a final time after 100 seconds. Should that attempt fail the `FinalWebhookCallFailedEvent` will be raised.

### Signing the webhook call

When setting up it's common to generate, store and share a secret secret between your app and the app that wants to receive webhooks. Generating the secret could be done with `Illuminate\Support\Str::random()`, but it's really entirely up to you. The package will use the secret to sign a webhook call.

By default the package will add a header called `Signature` that will contain a signature the receiving app can use the payload hasn't been tampered with. This is how that signature is calculated.

```php
// payload is the array passed to the `payload` method of the webhook
// secret is the string given to the `signUsingSecret` method on the webhook.

$payloadJson = json_encode($payload); 

$signature = hash_hmac('sha256', $payloadJson, $secret);
```

### Customizing signing requests



### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)

## Support us

Spatie is a webdesign agency based in Antwerp, Belgium. You'll find an overview of all our open source projects [on our website](https://spatie.be/opensource).

Does your business depend on our contributions? Reach out and support us on [Patreon](https://www.patreon.com/spatie). 
All pledges will be dedicated to allocating workforce on maintenance and new awesome stuff.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
