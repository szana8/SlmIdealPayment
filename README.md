SlmIdealPayment
===============
Created by Jurian Sluiman

Introduction
---
SlmIdealPayment is a Zend Framework 2 module to provide payments via the iDEAL
system. iDEAL is a payment service for the Dutch market and allows integration
of payments for almost all Dutch banks.

The module provides integration for the so-called iDEAL professional / iDEAL
advanced method. Integration via iDEAL basic / iDEAL lite is not included.

> **Standalone usage**
> This module is developed for Zend Framework 2, but can be used without the
> framework in your own application. There is no need to understand Zend
> Framework 2 to use this module.

Requirements
---
The module only requires a HTTP client where it utilizes the `Zend\Http`
component. Furthermore php 5.3+ is required to work with.

Installation
---
The module can be loaded via composer. Require `slm/ideal-payment` in your
composer.json file. If you do not have a composer.json file in the root of your
project, copy the contents below and put that into a file called composer.json
and save it in the root of your project:

```
{
    "require": {
        "slm/ideal-payment": "@dev"
    }
}
```

Then execute the following commands in a CLI:

```
curl -s http://getcomposer.org/installer | php
php composer.phar install
```

Now you should have a vendor directory, including a slm/ideal-payment. In your
bootstrap code, make sure you include the vendor/autoload.php file to properly
load the SlmIdealPayment module.

Configuration
---
The file `slmidealpayment.local.php.dist` eases the iDEAL configuration. Copy
the file from `vendor/slm/ideal-payment/config/slmidealpayment.local.php.dist`
to your autoload folder and remove the `.dist` extension.

Open the file and update the values to your needs. If you do not have a SSL
certificate, you can use a self-signed certificate and upload that one to the
iDEAL dashboard of your acquirer. Generate a key with 2048 bits encryption with
the following command:

```
openssl genrsa –aes128 –out priv.pem –passout pass:[privateKeyPass] 2048
```

Then create a certificate valid for 5 years:

```
openssl req –x509 –sha256 –new –key priv.pem –passin pass:[privateKeyPass]
-days 1825 –out cert.cer
```

Then use the `priv.pem` and `cert.cer` files for the iDEAL signatures.

Supported acquirers
---
SlmIdealPayment works with the following banks (or acquirers in iDEAL terms):

 1. Rabobank
 2. ING Bank
 3. ABN Amro

Usage
---
SlmIdealPayments will setup the client with the correct properties. Use the
following names to request an instance from the service manager:

 1. Rabobank: `SlmIdealPayment\Client\Standard\Rabobank`
 2. ING Bank: `SlmIdealPayment\Client\Standard\Ing`
 3. ABN Amro: `SlmIdealPayment\Client\Standard\AbnAmro`

### Directory request
Then use the client to perform the request. A directory request gives a list of
supported issuers. The result is a `SlmIdealPayment\Response\DirectoryResponse`:

```php
$client = $sl->get('SlmIdealPayment\Client\Standard\Rabobank');

$request  = new DirectoryRequest;
$response = $client->send($request);

foreach ($response as $issuer) {
    echo sprintf("%s: %s\n", $issuer->getId(), $issuer->getName());
}
```

### Transaction request
A transaction request needs a Transaction object. The result is a
`SlmIdealPayment\Response\TransactionResponse` object:

```php
use SlmIdealPayment\Model;

$client = $sl->get('SlmIdealPayment\Client\Standard\Rabobank');

// Set up the issuer
$issuer = new Model\Issuer;
$issuer->setId($issuer); // set selected issuer here

// Set up the transaction
$transaction = new Model\Transaction;
$transaction->setPurchaseId($purchaseId);
$transaction->setAmount($amount);
$transaction->setDescription($description);
$transaction->setEntranceCode($ec);

$request  = new TransactionRequest;
$request->setIssuer($issuer);
$request->setTransaction($transaction);

$response = $client->send($request);

echo $response->getTransaction()->getTransactionId();

// Then perform redirect:
// Redirect to $response->getAuthenticationUrl();
```

### Status request
A status request also needs a transaction object, but then for its transaction
id. The result is a `SlmIdealPayment\Response\StatusRequest`:

```php
use SlmIdealPayment\Model;

$client = $sl->get('SlmIdealPayment\Client\Standard\Rabobank');

$transaction = new Model\Transaction;
$transaction->setTransactionId($transactionId);

$request  = new StatusRequest;
$request->setTransaction($transaction);

$response = $client->send($request);

echo $response->getTransaction()->getStatus();
```