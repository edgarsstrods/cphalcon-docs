---
layout: default
title: 'Security - JWT'
keywords: 'security, hashing, passwords, jwt, rfc7519'
---

# Security - JSON Web Tokens (JWT)
- - -
![](/assets/images/document-status-stable-success.svg) ![](/assets/images/version-{{ pageVersion }}.svg)

## 概要

> **NOTE**: Currently, only symmetric algorithms are supported 
> 
> {: .alert .alert-info }

`Phalcon\Encryption\Security\JWT` is a namespace that contains components that allow you to issue, parse and validate JSON Web Tokens as described in [RFC 7915][rfc-7519]. These components are:

- Builder ([Phalcon\Encryption\Security\JWT\Builder][security-jwt-builder])
- Parser ([Phalcon\Encryption\Security\JWT\Token\Parser][security-jwt-token-parser])
- Validator ([Phalcon\Encryption\Security\JWT\Validator][security-jwt-validator])

> **NOTE**: For the examples below, we have split the output split into different lines for readability 
> 
> {: .alert .alert-warning }

An example of using the component is:

```php
<?php

use Phalcon\Encryption\Security\JWT\Builder;
use Phalcon\Encryption\Security\JWT\Signer\Hmac;
use Phalcon\Encryption\Security\JWT\Token\Parser;
use Phalcon\Encryption\Security\JWT\Validator;

// Defaults to 'sha512'
$signer  = new Hmac();

// Builder object
$builder = new Builder($signer);

$now        = new DateTimeImmutable();
$issued     = $now->getTimestamp();
$notBefore  = $now->modify('-1 minute')->getTimestamp();
$expires    = $now->modify('+1 day')->getTimestamp();
$passphrase = 'QcMpZ&b&mo3TPsPk668J6QH8JA$&U&m2';

// Setup
$builder
    ->setAudience('https://target.phalcon.io')  // aud
    ->setContentType('application/json')        // cty - header
    ->setExpirationTime($expires)               // exp 
    ->setId('abcd123456789')                    // JTI id 
    ->setIssuedAt($issued)                      // iat 
    ->setIssuer('https://phalcon.io')           // iss 
    ->setNotBefore($notBefore)                  // nbf
    ->setSubject('my subject for this claim')   // sub
    ->setPassphrase($passphrase)                // password 
;

// Phalcon\Encryption\Security\JWT\Token\Token
$tokenObject = $builder->getToken();

echo $tokenObject->getToken();

// eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImN0eSI6ImFwcGxpY2F0aW9uXC9qc29uIn0.
// eyJhdWQiOlsiaHR0cHM6XC9cL3RhcmdldC5waGFsY29uLmlvIl0sImV4cCI6MTYxNDE4NTkxN
// ywianRpIjoiYWJjZDEyMzQ1Njc4OSIsImlhdCI6MTYxNDA5OTUxNywiaXNzIjoiaHR0cHM6XC
// 9cL3BoYWxjb24uaW8iLCJuYmYiOjE2MTQwOTk0NTcsInN1YiI6Im15IHN1YmplY3QgZm9yIHR
// oaXMgY2xhaW0ifQ.
// LdYevRZaQDZ2lul4CCQ5DymeP2ubcapTtgeezOZGIq7Meu7rFF1pv32b-AMWOxCS63CQz_jpm
// BPlPyOeEAkMbg
```

```php
// #01
$tokenReceived = getMyTokenFromTheApplication();
$audience      = 'https://target.phalcon.io';
$now           = new DateTimeImmutable();
$issued        = $now->getTimestamp();
$notBefore     = $now->modify('-1 minute')->getTimestamp();
$expires       = $now->getTimestamp();
$id            = 'abcd123456789';
$issuer        = 'https://phalcon.io';

// #02
$signer     = new Hmac();
$passphrase = 'QcMpZ&b&mo3TPsPk668J6QH8JA$&U&m2';

// #03
$parser      = new Parser();

// Phalcon\Encryption\Security\JWT\Token\Token
$tokenObject = $parser->parse($tokenReceived);

// Phalcon\Encryption\Security\JWT\Validator
$validator = new Validator($tokenObject, 100); // #04

# 05
$validator
    ->validateAudience($audience)
    ->validateExpiration($expires)
    ->validateId($id)
    ->validateIssuedAt($issued)
    ->validateIssuer($issuer)
    ->validateNotBefore($notBefore)
    ->validateSignature($signer, $passphrase)
;

# 06
var_dump($validator->getErrors())
```

> **LEGEND**
> 
> 1. $tokenReceived is what we received
> 
> 2. Defaults to 'sha512'
> 
> 3. Parse the token
> 
> 4. Allow for a time shift of 100
> 
> 5. Run the validators
> 
> 6. Errors printed out (if any) 
>     
>     {: .alert .alert-info }

The above example gives a general view on how the component can be used to generate, parse and validate JSON Web Tokens.

## Objects

There are several utility components that live in the `Phalcon\Encryption\Security\JWT\Token` namespace, that help with the issuing, parsing and validating JWT tokens

### Enum

[Phalcon\Encryption\Security\JWT\Token\Enum][security-jwt-token-enum] is a class that contains several constants. These constants are the strings defined in [RFC 7915][rfc-7519]. You can use them if you wish or instead use their string equivalents.

```php
<?php

class Enum
{
    /**
     * Headers
     */
    const TYPE         = "typ";
    const ALGO         = "alg";
    const CONTENT_TYPE = "cty";

    /**
     * Claims
     */
    const AUDIENCE        = "aud";
    const EXPIRATION_TIME = "exp";
    const ID              = "jti";
    const ISSUED_AT       = "iat";
    const ISSUER          = "iss";
    const NOT_BEFORE      = "nbf";
    const SUBJECT         = "sub";
}
```

### Item

[Phalcon\Encryption\Security\JWT\Token\Item][security-jwt-token-item] is used internally to store a payload as well as its encoded state. Such payload can be the claims data or the headers' data. By using this component, we can easily extract the necessary information for each Token.

### Signature

[Phalcon\Encryption\Security\JWT\Token\Signature][security-jwt-token-signature] is similar to the [Phalcon\Encryption\Security\JWT\Token\Item][security-jwt-token-item], but it only holds the signature hash as well as its encoded value.

### Token

[Phalcon\Encryption\Security\JWT\Token\Token][security-jwt-token-token] is the component responsible for storing and calculating the JWT token. It accepts the headers, claims (as [Phalcon\Encryption\Security\JWT\Token\Item][security-jwt-token-item] objects) and signature objects in its constructor and exposes:

```php
public function getClaims(): Item
```
Return the claims collection

```php
public function getHeaders(): Item
```
Return the headers collection

```php
public function getPayload(): string
```
Return the payload. For a token `abcd.efgh.ijkl`, it will return `abcd.efgh`

```php
public function getSignature(): Signature
```
Return the signature

```php
public function getToken(): string
```
Return the token as a string. For a token `abcd.efgh.ijkl` it will return `abcd.efgh.ijkl`.

```php
public function validate(Validator $validator): array
```
Run all validators against the token data. Return the errors array from the validator

```php
public function verify(SignerInterface $signer, string $key): bool
```
Verify the signature of the token



### Signer

In order to create a JWT token, we need to supply a Signing algorithm. By default, the builder uses "none" ([Phalcon\Encryption\Security\JWT\Signer\None][security-jwt-signer-none]). You can however use the HMAC signer ([Phalcon\Encryption\Security\JWT\Signer\Hmac][security-jwt-signer-hmac]). Also, for further customization, you can utilize the supplied [Phalcon\Encryption\Security\JWT\Signer\SignerInterface][security-jwt-signer-signerinterface] interface.

```php
<?php

use Phalcon\Encryption\Security\JWT\Signer\Hmac;

$signer  = new Hmac();
```

**None**

This signer is provided mostly for development purposes. You should always sign your JWT tokens.

**HMAC**

The HMAC signer supports the `sha512`, `sha384`, and `sha256` algorithms. If none is supplied, the `sha512` is automatically selected. If you supply a different algorithm, a [Phalcon\Encryption\Security\JWT\Exceptions\UnsupportedAlgorithmException][security-jwt-exceptions-unsupportedalgorithmexception] will be raised. The algorithm is set in the constructor.


```php
<?php

use Phalcon\Encryption\Security\JWT\Signer\Hmac;

$signer  = new Hmac();
$signer  = new Hmac('sha512');
$signer  = new Hmac('sha384');
$signer  = new Hmac('sha256');
$signer  = new Hmac('sha111'); // exception
```

The component utilizes the \[hmac_equals\]\[hmac_equals\] and \[hash_hmac\]\[hash_hmac\] PHP methods internally to verify and sign the payload. It exposes the following methods:

```php
public function getAlgHeader(): string
```

Returns a string identifying the algorithm. For the HMAC algorithms it will return:

| Algorithm | `getAlgHeader` |
|:---------:|:--------------:|
| `sha512`  |    `HS512`     |
| `sha384`  |    `HS384`     |
| `sha256`  |    `HS256`     |

```php
public function sign(string $payload, string $passphrase): string
```

Returns the hash of the payload using the passphrase

```php
public function verify(string $source, string $payload, string $passphrase): bool
```

Verifies that the hashed source string is the same as the hash of the payload with the passphrase.

## Issuing Tokens

A Builder component ([Phalcon\Encryption\Security\JWT\Builder][security-jwt-builder]) is available, utilizing chained methods, and ready to be used to create JWT tokens. All you have to do is instantiate the Builder object, configure your token and call `getToken()`. This will return a [Phalcon\Encryption\Security\Token\Token][security-jwt-token-token] object which contains all the necessary information for your token. When instantiating the builder component, you have to supply the signer class. In the example below we use the [Phalcon\Encryption\Security\JWT\Signer\Hmac][security-jwt-signer-hmac] signer.

All setters in this component are chainable.

```php
<?php

use Phalcon\Encryption\Security\JWT\Builder;
use Phalcon\Encryption\Security\JWT\Signer\Hmac;

// Defaults to 'sha512'
$signer  = new Hmac();

// Builder object
$builder = new Builder($signer);
```

### メソッド

```php
public function __construct(SignerInterface $signer): Builder
```
Constructor

```php
public function init(): Builder
```
Initializes the object - useful when you want to reuse the same builder

```php
public function addClaim(string $name, mixed $value): Builder
```
Adds a custom claim in the claims collection

```php
public function getAudience(): array|string
```
Returns the `aud` contents

```php
public function getClaims(): array
```
Returns the claims as an array

```php
public function getContentType(): ?string
```
Returns the content type (`cty` - headers)

```php
public function getExpirationTime(): ?int
```
Returns the `exp` contents

```php
public function getHeaders(): array
```
Returns the headers as an array

```php
public function getId(): ?string
```
Returns the `jti` contents (ID of this JWT)

```php
public function getIssuedAt(): ?int
```
Returns the `iat` contents

```php
public function getIssuer(): ?string
```
Returns the `iss` contents

```php
public function getNotBefore(): ?int
```
Returns the `nbf` contents

```php
public function getSubject(): ?string
```
Returns the `sub` contents

```php
public function getToken(): Token
```
Returns the token

```php
public function getPassphrase(): string
```
Returns the supplied passphrase

```php
public function setAudience(array|string $audience): Builder
```
Sets the audience (`aud`). If the parameter passed is not an array or a string, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function setContentType(string $contentType): Builder
```
Sets the content type (`cty` - headers)

```php
public function setExpirationTime(int $timestamp): Builder
```
Sets the audience (`exp`). If the `$timestamp` is less than the current time, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function setId(string $id): Builder
```
Sets the id (`jti`).

```php
public function setIssuedAt(int $timestamp): Builder
```
Sets the issued at time (`iat`).

```php
public function setIssuer(string $issuer): Builder
```
Sets the issuer (`iss`).

```php
public function setNotBefore(int $timestamp): Builder
```
Sets the not before time (`nbf`). If the `$timestamp` is greater than the current time, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function setSubject(string $subject): Builder
```
Sets the subject (`sub`).

```php
public function setPassphrase(string $passphrase): Builder
```
Sets the passphrase. If the `$passphrase` is weak, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
private function setClaim(string $name, $value): Builder
```
Sets a claim value in the internal collection.


### 例

```php
<?php

use Phalcon\Encryption\Security\JWT\Builder;
use Phalcon\Encryption\Security\JWT\Signer\Hmac;
use Phalcon\Encryption\Security\JWT\Token\Parser;
use Phalcon\Encryption\Security\JWT\Validator;

// 'sha512'
$signer  = new Hmac();

$builder = new Builder($signer);

$now        = new DateTimeImmutable();
$issued     = $now->getTimestamp();
$notBefore  = $now->modify('-1 minute')->getTimestamp();
$expires    = $now->modify('+1 day')->getTimestamp();
$passphrase = 'QcMpZ&b&mo3TPsPk668J6QH8JA$&U&m2';

$builder
    ->setAudience('https://target.phalcon.io')  // aud
    ->setContentType('application/json')        // cty - header
    ->setExpirationTime($expires)               // exp 
    ->setId('abcd123456789')                    // JTI id 
    ->setIssuedAt($issued)                      // iat 
    ->setIssuer('https://phalcon.io')           // iss 
    ->setNotBefore($notBefore)                  // nbf
    ->setSubject('my subject for this claim')   // sub
    ->setPassphrase($passphrase)                // password 
;

// Phalcon\Encryption\Security\JWT\Token\Token 
$tokenObject = $builder->getToken();

echo $tokenObject->getToken();

// eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImN0eSI6ImFwcGxpY2F0aW9uXC9qc29uIn0.
// eyJhdWQiOlsiaHR0cHM6XC9cL3RhcmdldC5waGFsY29uLmlvIl0sImV4cCI6MTYxNDE4NTkxN
// ywianRpIjoiYWJjZDEyMzQ1Njc4OSIsImlhdCI6MTYxNDA5OTUxNywiaXNzIjoiaHR0cHM6XC
// 9cL3BoYWxjb24uaW8iLCJuYmYiOjE2MTQwOTk0NTcsInN1YiI6Im15IHN1YmplY3QgZm9yIHR
// oaXMgY2xhaW0ifQ.
// LdYevRZaQDZ2lul4CCQ5DymeP2ubcapTtgeezOZGIq7Meu7rFF1pv32b-AMWOxCS63CQz_jpm
// BPlPyOeEAkMbg
```

## Validating Tokens

In order to validate a token you will need to create a new [Phalcon\Encryption\Security\JWT\Validator][security-jwt-validator] object. The object can be constructed using a [Phalcon\Encryption\Security\JWT\Token\Token][security-jwt-token-token] object and an offset in time to handle time/clock shifts of the sending and receiving computers.

In order to parse the JWT received and convert it to a [Phalcon\Encryption\Security\JWT\Token\Token][security-jwt-token-token] object, you will need to use a [Phalcon\Encryption\Security\JWT\Token\Parser][security-jwt-token-parser] object and parse it.

### Validator
```php
$parser = new Parser();

$tokenObject = $parser->parse($tokenReceived);

$validator = new Validator($tokenObject, 100); // allow for a time shift of 100
```
You can use the [Phalcon\Encryption\Security\JWT\Validator][security-jwt-validator] object to validate each claim by calling the `validate*` methods with the necessary parameters (taken from the [Phalcon\Encryption\Security\Token\Token][security-jwt-token-token]). The internal `errors` array in the [Phalcon\Encryption\Security\JWT\Validator][security-jwt-validator] will be populated accordingly, returning the results with the `getErrors()` method.

#### メソッド

```php
public function __construct(Token $token, int $timeShift = 0)
```
Constructor

```php
public function get(string $claim): mixed | null
```
Returns a claim's value - `null` if the claim does not exist

```php
public function set(string $claim, mixed $value): Validator
```
Sets a claim and its value

```php
public function setToken(Token $token): Validator
```
Sets the token object.

```php
public function validateAudience(array|string $audience): Validator
```
Validates the audience. If it is not included in the token's `aud`, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateExpiration(int $timestamp): Validator
```
Validates the expiration time. If the `exp` value stored in the token is less than now, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateId(string $id): Validator
```
Validates the id. If it is not the same as the `jti` value stored in the token, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateIssuedAt(int $timestamp): Validator
```
Validates the issued at time. If the `iat` value stored in the token is greater than now, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateIssuer(string $issuer): Validator
```
Validates the issuer. If it is not the same as the `iss` value stored in the token, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateNotBefore(int $timestamp): Validator
```
Validates the not before time. If the `nbf` value stored in the token is greater than now, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

```php
public function validateSignature(SignerInterface $signer, string $passphrase): Validator
```
Validates the signature of the token. If the signature is not valid, a [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception] will be thrown.

#### 例

```php
<?php

use Phalcon\Encryption\Security\JWT\Signer\Hmac;
use Phalcon\Encryption\Security\JWT\Token\Parser;
use Phalcon\Encryption\Security\JWT\Validator;

// eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImN0eSI6ImFwcGxpY2F0aW9uXC9qc29uIn0.
// eyJhdWQiOlsiaHR0cHM6XC9cL3RhcmdldC5waGFsY29uLmlvIl0sImV4cCI6MTYxNDE4NTkxN
// ywianRpIjoiYWJjZDEyMzQ1Njc4OSIsImlhdCI6MTYxNDA5OTUxNywiaXNzIjoiaHR0cHM6XC
// 9cL3BoYWxjb24uaW8iLCJuYmYiOjE2MTQwOTk0NTcsInN1YiI6Im15IHN1YmplY3QgZm9yIHR
// oaXMgY2xhaW0ifQ.
// LdYevRZaQDZ2lul4CCQ5DymeP2ubcapTtgeezOZGIq7Meu7rFF1pv32b-AMWOxCS63CQz_jpm
// BPlPyOeEAkMbg

$tokenReceived = getMyTokenFromTheApplication();
$audience      = 'https://target.phalcon.io';
$now           = new DateTimeImmutable();
$issued        = $now->getTimestamp();
$notBefore     = $now->modify('-1 minute')->getTimestamp();
$expires       = $now->getTimestamp();
$id            = 'abcd123456789';
$issuer        = 'https://phalcon.io';

// 'sha512'
$signer     = new Hmac();
$passphrase = 'QcMpZ&b&mo3TPsPk668J6QH8JA$&U&m2';

$parser      = new Parser();

// Phalcon\Encryption\Security\JWT\Token\Token 
$tokenObject = $parser->parse($tokenReceived);

// Phalcon\Encryption\Security\JWT\Validator 
$validator = new Validator($tokenObject, 100); // allow for a time shift of 100

$validator
    ->validateAudience($audience)
    ->validateExpiration($expires)
    ->validateId($id)
    ->validateIssuedAt($issued)
    ->validateIssuer($issuer)
    ->validateNotBefore($notBefore)
    ->validateSignature($signer, $passphrase)
;

var_dump($validator->getErrors());
```

### Token
As an alternative, you can `verify()` and `validate()` your token using the relevant methods in the [Phalcon\Encryption\Security\Token\Token][security-jwt-token-token] object.

#### メソッド

```php
public function validate(Validator $validator): array
```
Validate the token claims. The validators that are executed are:

- `validateAudience()`
- `validateExpiration()`
- `validateId()`
- `validateIssuedAt()`
- `validateIssuer()`
- `validateNotBefore()`

You can extend the [Phalcon\Encryption\Security\JWT\Validator][security-jwt-validator] and [Phalcon\Encryption\Security\Token\Token][security-jwt-token-token] objects to include more validators and execute them (as seen below).

```php
public function verify(SignerInterface $signer, string $key): bool
```
Verify the signature of the token

#### 例

```php
<?php

use Phalcon\Encryption\Security\JWT\Enum;
use Phalcon\Encryption\Security\JWT\Signer\Hmac;
use Phalcon\Encryption\Security\JWT\Token\Parser;
use Phalcon\Encryption\Security\JWT\Validator;

// eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImN0eSI6ImFwcGxpY2F0aW9uXC9qc29uIn0.
// eyJhdWQiOlsiaHR0cHM6XC9cL3RhcmdldC5waGFsY29uLmlvIl0sImV4cCI6MTYxNDE4NTkxN
// ywianRpIjoiYWJjZDEyMzQ1Njc4OSIsImlhdCI6MTYxNDA5OTUxNywiaXNzIjoiaHR0cHM6XC
// 9cL3BoYWxjb24uaW8iLCJuYmYiOjE2MTQwOTk0NTcsInN1YiI6Im15IHN1YmplY3QgZm9yIHR
// oaXMgY2xhaW0ifQ.
// LdYevRZaQDZ2lul4CCQ5DymeP2ubcapTtgeezOZGIq7Meu7rFF1pv32b-AMWOxCS63CQz_jpm
// BPlPyOeEAkMbg

$tokenReceived = getMyTokenFromTheApplication();
$subject       = 'Mary had a little lamb';
$audience      = 'https://target.phalcon.io';
$now           = new DateTimeImmutable();
$issued        = $now->getTimestamp();
$notBefore     = $now->modify('-1 minute')->getTimestamp();
$expires       = $now->getTimestamp();
$id            = 'abcd123456789';
$issuer        = 'https://phalcon.io';

// 'sha512'
$signer     = new Hmac();
$passphrase = 'QcMpZ&b&mo3TPsPk668J6QH8JA$&U&m2';

$parser      = new Parser();

// Phalcon\Encryption\Security\JWT\Token\Token 
$tokenObject = $parser->parse($tokenReceived);

// Phalcon\Encryption\Security\JWT\Validator 
$validator = new Validator($tokenObject, 100); // allow for a time shift of 100

$validator
    ->set(Enum::AUDIENCE, $audience)
    ->set(Enum::EXPIRATION_TIME, $expiry)
    ->set(Enum::ISSUER, $issuer)
    ->set(Enum::ISSUED_AT, $issued)
    ->set(Enum::ID, $id)
    ->set(Enum::NOT_BEFORE, $notBefore)
    ->set(Enum::SUBJECT, $subject)
;

$tokenObject->verify($signer, $passphrase);
$errors = $tokenObject->validate($validator);

var_dump($errors);
```

## Exceptions

Any exceptions thrown in the Security component will be of the namespace `Phalcon\Encryption\Security\JWT\*`. You can use this exception to selectively catch exceptions thrown only from this component. There are two exceptions raised. First if you supply the wrong algoritm string when instantiating the [Phalcon\Encryption\Security\JWT\Signer\Hmac][security-jwt-signer-hmac] component. This exception is [Phalcon\Encryption\Security\JWT\Exceptions\UnsupportedAlgorithmException][security-jwt-exceptions-unsupportedalgorithmexception].

The second exception is thrown when validating a JWT. This exception is [Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException][security-jwt-exceptions-validatorexception].

```php
<?php

use Phalcon\Mvc\Controller;
use Phalcon\Encryption\Security\JWT\Builder;
use Phalcon\Encryption\Security\JWT\Exceptions\ValidatorException;
use Phalcon\Encryption\Security\JWT\Signer\Hmac;
use Phalcon\Encryption\Security\JWT\Validator;

class IndexController extends Controller
{
    public function index()
    {
        try {
            $signer     = new Hmac();
            $builder    = new Builder($signer);
            $expiry     = strtotime('+1 day');
            $issued     = strtotime('now') + 100;
            $notBefore  = strtotime('-1 day');
            $passphrase = '&vsJBETaizP3A3VX&TPMJUqi48fJEgN7';

            return $builder
                ->setAudience('my-audience')
                ->setExpirationTime($expiry)
                ->setIssuer('Phalcon JWT')
                ->setIssuedAt($issued)
                ->setId('PH-JWT')
                ->setNotBefore($notBefore)
                ->setSubject('Mary had a little lamb')
                ->setPassphrase($passphrase)
                ->getToken()
            ;

            $validator = new Validator($token);
            $validator->validateAudience("unknown");
        } catch (Exception $ex) {
            echo $ex->getMessage(); // Validation: audience not allowed
        }
    }
}
```

[rfc-7519]: https://datatracker.ietf.org/doc/html/rfc7519
[security-jwt-builder]: api/phalcon_encryption#encryption-security-jwt-builder
[security-jwt-exceptions-unsupportedalgorithmexception]: api/phalcon_encryption#encryption-security-jwt-exceptions-unsupportedalgorithmexception
[security-jwt-exceptions-validatorexception]: api/phalcon_encryption#encryption-security-jwt-exceptions-validatorexception
[security-jwt-signer-hmac]: api/phalcon_encryption#encryption-security-jwt-signer-hmac
[security-jwt-signer-none]: api/phalcon_encryption#encryption-security-jwt-signer-none
[security-jwt-signer-signerinterface]: api/phalcon_encryption#encryption-security-jwt-signer-signerinterface
[security-jwt-token-enum]: api/phalcon_encryption#encryption-security-jwt-token-enum
[security-jwt-token-item]: api/phalcon_encryption#encryption-security-jwt-token-item
[security-jwt-token-parser]: api/phalcon_encryption#encryption-security-jwt-token-parser
[security-jwt-token-signature]: api/phalcon_encryption#encryption-security-jwt-token-signature
[security-jwt-token-token]: api/phalcon_encryption#encryption-security-jwt-token-token
[security-jwt-token-token]: api/phalcon_encryption#encryption-security-jwt-token-token
[security-jwt-validator]: api/phalcon_encryption#encryption-security-jwt-validator
