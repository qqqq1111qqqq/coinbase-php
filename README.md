# Coinbase PHP Client Library

An easy way to buy, send, and accept [bitcoin](http://en.wikipedia.org/wiki/Bitcoin) through the [Coinbase API](https://coinbase.com/docs/api/overview).

This library only supports the [API key authentication method](https://coinbase.com/docs/api/overview). OAuth2 support is not yet implemented.

## Installation

Obtain the latest version of the Coinbase PHP library with:

    git clone https://github.com/coinbase/coinbase-php

Then, add the following to your PHP script:

    require_once("/path/to/coinbase-php/lib/Coinbase.php");

## Usage

Start by [enabling an API Key on your account](https://coinbase.com/account/integrations).

Next, create an instance of the client and pass it your API Key as the first (and only) parameter.

```php
$coinbase = new Coinbase($_ENV['COINBASE_API_KEY'])
```

Notice here that we did not hard code the API key into our codebase, but set it in an environment variable instead.  This is just one example, but keeping your credentials separate from your code base is a good [security practice](https://coinbase.com/docs/api/overview#security).

Now you can call methods on `$coinbase` similar to the ones described in the [API reference](https://coinbase.com/api/doc).  For example:

```php
$balance = $coinbase->getBalance();
echo "Balance is " . $balance . " BTC";
```
  
Currency amounts are returned as Strings. To avoid precision errors, use the [PHP arbitrary precision math functions ](http://www.php.net/manual/en/ref.bc.php) to work with money amounts.

## Examples

### Check your balance

```php
echo $coinbase->getBalance() . " BTC";
// '200.123 BTC'
```

### Send bitcoin

`public function sendMoney($to, $amount, $notes=null, $userFee=null, $amountCurrency=null)`

```php
$response = $coinbase->sendMoney("user@example.com", "2");
echo $response->success ? 'true' : 'false';
// 'true'
echo $response->transaction->status;
// 'pending'
echo $response->transaction->id;
// '518d8567ed3ddcd4fd000034'
```

The first parameter can also be a bitcoin address and the third parameter can be a note or description of the transaction.  Descriptions are only visible on Coinbase (not on the general bitcoin network).

```php
$response = $coinbase->sendMoney("mpJKwdmJKYjiyfNo26eRp4j6qGwuUUnw9x", "0.1", "thanks for the coffee!");
echo $response->transaction->notes;
// 'thanks for the coffee!'
```

You can also send money in [a number of currencies](https://github.com/coinbase/coinbase-ruby/blob/master/supported_currencies.json) using the fifth parameter.  The amount will be automatically converted to the correct BTC amount using the current exchange rate.

```php
$response = $coinbase->sendMoney("user@example.com", "2", null, null, "CAD");
echo $response->transaction->amount->amount;
// '0.0169'
```

### Request bitcoin

This will send an email to the recipient, requesting payment, and give them an easy way to pay.

```php
$response = $coinbase->requestMoney('client@example.com', 50, "contractor hours in January (website redesign for 50 BTC)");
echo $response->transaction->request ? 'true' : 'false';
// 'true'
echo $response->transaction->id;
// '501a3554f8182b2754000003'

$response = $coinbase->resendRequest('501a3554f8182b2754000003');
echo $response->success ? 'true' : 'false';
// 'true'

$response = $coinbase->cancelRequest('501a3554f8182b2754000003');
echo $response->success ? 'true' : 'false';
// 'true'

// From the other account:
$response = $coinbase->completeRequest('501a3554f8182b2754000003');
echo $response->success ? 'true' : 'false';
// 'true'
```

### Create a payment button

This will create the code for a payment button (and modal window) that you can use to accept bitcoin on your website.  You can read [more about payment buttons here and try a demo](https://coinbase.com/docs/merchant_tools/payment_buttons).

The method signature is `public function createButton($name, $price, $currency, $custom=null, $options=array())`.  The `custom` param will get passed through in [callbacks](https://coinbase.com/docs/merchant_tools/callbacks) to your site.  The list of valid `options` [are described here](https://coinbase.com/api/doc/buttons/create.html).

```php
$response = $coinbase->createButton("Your Order #1234", "42.95", "EUR", "my custom tracking code for this order", array(
            "description" => "1 widget at �42.95"
        ));
echo $response->button->code;
// '93865b9cae83706ae59220c013bc0afd'
echo $response->embedHtml;
// '<div class=\"coinbase-button\" data-code=\"93865b9cae83706ae59220c013bc0afd\"></div><script src=\"https://coinbase.com/assets/button.js\" type=\"text/javascript\"></script>'
```

## Adding new methods

You can see a [list of method calls here](https://github.com/coinbase/coinbase-php/blob/master/lib/Coinbase/Coinbase.php) and how they are implemented.  They are a wrapper around the [Coinbase JSON API](https://coinbase.com/api/doc).

If there are any methods listed in the [API Reference](https://coinbase.com/api/doc) that don't have an explicit function name in the library, you can also call `get`, `post`, `put`, or `delete` with a `$path` and optional `$params` array for a quick implementation.  The raw JSON object will be returned. For example:

```php
var_dump($coinbase->get('/account/balance'));
// object(stdClass)#4 (2) {
//   ["amount"]=>
//   string(10) "0.56902981"
//   ["currency"]=>
//   string(3) "BTC"
// }
```

Or feel free to add a new wrapper method and submit a pull request.

## Security notes

If someone gains access to your API Key they will have complete control of your Coinbase account.  This includes the abillity to send all of your bitcoins elsewhere.

For this reason, API access is disabled on all Coinbase accounts by default.  If you decide to enable API key access you should take precautions to store your API key securely in your application.  How to do this is application specific, but it's something you should [research](http://programmers.stackexchange.com/questions/65601/is-it-smart-to-store-application-keys-ids-etc-directly-inside-an-application) if you have never done this before.

## Testing

If you'd like to contribute code or modify this library, you can run the test suite by executing `/path/to/coinbase-php/test/Coinbase.php` in a web browser or on the command line with `php`.
