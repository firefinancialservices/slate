---
title: Fire Business Account API Reference

language_tabs:
  - shell: cURL
  - java: Java
  - php: PHP

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:

search: true
---

# Integrating to the Pay with Fire Business Account API

The Pay with Fire API allows you to deeply integrate our account features into your application.

# Authentication

```shell
# cat logindetails.json
{
	"businessClientId": "<BUSINESSID>",
	"emailAddress": "<EMAIL>",
	"password": "<PASSWORD>"
}

# Post that to the API
curl https://business.realexfire.com/api/login \
  -X POST \
  -D -
  -d @logindetails.json

# Grab the Authentication Token from the headers (at)

HTTP/1.1 200 OK
...
at: aeb5fdf3-eee2-4e80-b605-7e2fac2add36
...
Content-Length: 2112
Date: Wed, 29 Apr 2015 21:27:17 GMT
```
```json
{
   "userProfile": {
      "firstName": "Brian",
      "lastName": "Johnson",
      "userEmail": "brian@website.com",
      "mobileNumber": "+353871111111",
      "cvl": "FULL",
      "permissions": [],
      "lastLogin": "2015-04-29T21:28:13.233Z"
   },
   "businessProfile": {
      "businessName": "Brian's Company",
      "businessClientId": "brianscompany",
      "businessWebsite": "https://www.website.com",
      "businessAddress": {
         "address1": "7-11 Sir John Rogerson's Quay",
         "city": "Dublin 2",
         "country": {
            "code": "IE",
            "description": "Ireland"
         }
      },
      "businessDetailsSubmitted": true,
      "businessType": "OTHER",
      "apiTokens": [
         {
            "privateToken": "<PRIVATE-TOKEN>",
            "dateCreated": 1423749242837,
            "publicToken": "<PUBLIC-TOKEN>",
            "active": true,
            "tokenId": 2
         }
      ]
   }
}
```

```BETA```
In the BETA period, the authentication process piggybacks the Business Fire Account web application login. This
will change to a dedicated API token once the API is implemented.

Get an authentication token by passing your business id, email and password.
Replace `privateToken` in the code samples with your private token from your Business Account Profile page.


# Fire Accounts 

```shell
# JSON representation of a Fire Account
{
	"cban": 64783,
	"alias": "Main Account",
	"currency": "EUR",
	"balance": 434050,
	"bic": "CPAYID2D",
	"iban": "IE73CPAY99119982718273",
	"nameOnAccount": "Tim's Pen Shop"
}
```    

```java
import com.paywithfire.api.business.FireAccount;

// Java Object representing a Fire Account
FireAccount account = new FireAccount();

int id               = account.getCBAN();
String alias         = account.getAlias();
String currency      = account.getCurrency();
long balance         = account.getBalance();
String bic           = account.getBIC();
String iban          = account.getIBAN();
String accountNumber = account.getAccountNumber();
String sortCode      = account.getSortCode();
String nameOnAccount = account.getNameOnAccount();
```

```php
<?php
# PHP Object representing a Fire Account
$account = new PayWithFire_FireAccount();

$id            = $account->cban;
$alias         = $account->alias;
$currency      = $account->currency;
$balance       = $account->balance;
$bic           = $account->bic;
$iban          = $account->iban;
$accountNumber = $account->accountNumber;
$sortCode      = $account->sortCode;
$nameOnAccount = $account->nameOnAccount;
?>
```

Fire Accounts are the Pay with Fire equivalent of a bank account from bank, but with extra features you won't find anywhere else. 

The resource has the following attributes: 

Field | Description
--------- | -----------
`cban` | identifier for the Fire account _(assigned by Pay with Fire)_ 
`alias` | the name the user gives to the account to help them identify it. 
`currency` | the currency of the account - either `EUR` or `GBP`.
`balance` | the balance of the account (in minor currency units - pence, cent etc. `434050` == `4,340.50 GBP` for a GBP account).
`bic` | the BIC of the account if currency is `EUR`. 
`iban` | the IBAN of the account if currency is `EUR`. 
`sortCode` | the Sort Code of the account if currency is `GBP`. 
`accountNumber` | the Account Number of the account if currency is `GBP`. 
`nameOnAccount` | the name on the account - will be set as your business name. 

## List all Fire Accounts

```shell
curl https://paywithfire.com/business/v1/me/accounts
  -X GET
  -d "page=2"
  -d "count=20"
  -H "Authorization: privateToken"


{
	"total": 1,
	"accounts": [
	    {
		"cban": 64783,
		"alias": "Main Account",
		"currency": "EUR",
		"balance": 434050,
		"bic": "CPAYID2D",
		"iban": "IE73CPAY99119982718273",
		"nameOnAccount": "Tim's Pen Shop"
	    }
	]
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

List<FireAccount> accounts 
	= businessAccount.accounts().list();

// you can also configure parameters...
Config params = new Config()
	.setCount(20)
	.setPage(2);

List<FireAccount> accounts 
	= businessAccount.accounts().list(params);

// Now you can retrieve values from the accounts
String iban = accounts.get(0).getIBAN();
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

$accounts = $businessAccount->accounts()->list();

# you can also configure parameters...
$params = array(
	"count" => 20,
	"page" => 2
);

$accounts = $businessAccount->accounts()->list($params);

// Now you can retrieve values from the accounts
$iban = accounts[0]->iban;
?>
```

Returns all your Fire Accounts. Ordered by Alias ascending. Can be paginated. 

### HTTP Request

`GET https://paywithfire.com/business/v1/me/accounts`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
`page` | 1 | The page offset to return if paginating the results.
`count` | 20 | The count of requests to return per page if paginating the results.


## Create a new Fire Account

```shell
# cat newaccount.json
{
	"alias": "UK Invoicing Account",
	"currency": "GBP"
}

# Post that to the API
curl https://paywithfire.com/business/v1/me/accounts
  -X POST
  -d @newaccount.json
  -H "Authorization: privateToken"

{
	"cban": 924733, 
	"currency": "GBP",
	"balance": 0,
	"sortCode": "232221",
	"accountNumber": "34658388",
	"nameOnAccount": "Tim's Pen Shop"
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

FireAccount ukinvoicing = new FireAccount()
	.setAlias("UK Invoicing Account")
	.setCurrency("GBP");

GenericResult res = businessAccount.accounts().add(ukinvoicing);

// Should be a positive result
if (Result.CREATED_OK == res.getResult()) {
	int sortCode = res.getSortCode();
	long balance = res.getBalance(); // 0
	// ....
}
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

$ukinvoicing = new PayWithFire_FireAccount(array(
	"alias" => "UK Invoicing Account",
	"currency" => "GBP"
));

$res = $businessAccount->accounts()->add($ukinvoicing);

# Should be a positive result
if (res->result == "CREATED_OK") {
	$sortCode = res->sortCode;
	$balance = res->balance; # 0
	# ....
}
?>
```

To add a new Fire Account you just need a name and a currency. The details of the new account will be returned to you.

### HTTP Request

`POST https://paywithfire.com/business/v1/me/accounts`

## Retrieve the details of a Fire Account 

```shell
curl https://paywithfire.com/business/v1/me/accounts/924733
  -X GET
  -H "Authorization: privateToken"

{
	"cban": 924733, 
	"currency": "GBP",
	"sortCode": "232221",
	"accountNumber": "34658388",
	"nameOnAccount": "Tim's Pen Shop"
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

try {
	FireAccount account 
		= businessAccount.accounts().get(924733);
} catch (NoSuchFireAccountException nsfae) {
	// dang
}

// Now you can retrieve values from the account
String iban = account.getIBAN();
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

try {
	$account 
		= $businessAccount->accounts()->get(924733);
} catch (PayWithFire_Exception_NoSuchFireAccount $nsfae) {
	# dang
}

# Now you can retrieve values from the account
$iban = $account->iban;
?>
```
You can retrieve the details of a Fire Account by its `cban`. 

### HTTP Request

`GET https://paywithfire.com/business/v1/me/accounts/{cban}`

Parameter | Description
--------- | -----------
`cban` | This is the CBAN of the Fire Account to be returned.

# External Bank Accounts 

```shell
# JSON representation of an External Account
{
	"externalAccountId": 23492,
	"alias": "BoI Current Account",
	"currency": "EUR",
	"balance": 0,
	"bic": "BOFIIE2D",
	"iban": "IE23BOFI90001782718273",
	"nameOnAccount": "Tim's Pen Shop"
}
```    

```java
import com.paywithfire.api.business.ExternalAccount;

// Java Object representing an External Account
ExternalAccount account = new ExternalAccount();

int id               = account.getId();
String alias         = account.getAlias();
String currency      = account.getCurrency();
String bic           = account.getBIC();
String iban          = account.getIBAN();
String accountNumber = account.getAccountNumber();
String sortCode      = account.getSortCode();
String nameOnAccount = account.getNameOnAccount();
```

```php
<?php
# PHP Object representing an External Account
$account = new PayWithFire_ExternalAccount();

$id            = $account->cban;
$alias         = $account->alias;
$currency      = $account->currency;
$bic           = $account->bic;
$iban          = $account->iban;
$accountNumber = $account->accountNumber;
$sortCode      = $account->sortCode;
$nameOnAccount = $account->nameOnAccount;
?>
```

You can add bank accounts from other banks to your profile and transfer to these by bank transfer. These can either be your accounts in other banks or can be suppliers or customers you need to pay by bank transfer. 

The resource has the following attributes: 

Field | Description
--------- | -----------
`externalAccountId` | identifier for the Bank account _(assigned by Pay with Fire)_ 
`alias` | the name the user gives to the bank account to help them identify the account. 
`currency` | the currency of the bank account - either `EUR` or `GBP`.
`bic` | the BIC of the account if currency is `EUR`. 
`iban` | the IBAN of the account if currency is `EUR`. 
`sortCode` | the Sort Code of the account if currency is `GBP`. 
`accountNumber` | the Account Number of the account if currency is `GBP`. 
`nameOnAccount` | the name on the external bank account. 



## List all External Bank Accounts

```shell
curl https://paywithfire.com/business/v1/me/externalAccounts
  -X GET
  -d "page=2"
  -d "count=20"
  -H "Authorization: privateToken"


{
	"total": 1,
	"externalaccounts": [
	    {
		"externalAccountId": 23492,
		"alias": "Bank of Ireland Account",
		"currency": "EUR",
		"bic": "BOFIIE2D",
		"iban": "IE23BOFI90001782718273",
		"nameOnAccount": "Tim's Pen Shop"
	    }
	]
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

List<ExternalAccount> accounts 
	= businessAccount.externalAccounts().list();

// you can also configure parameters...
Config params = new Config()
	.setCount(20)
	.setPage(2);

List<ExternalAccount> accounts 
	= businessAccount.externalAccounts().list(params);

// Now you can retrieve values from the accounts
String iban = accounts.get(0).getIBAN();
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

$accounts = $businessAccount->externalAccounts()->list();

# you can also configure parameters...
$params = array(
	"count" => 20,
	"page" => 2
);

$accounts = $businessAccount->externalAccounts()->list($params);

// Now you can retrieve values from the accounts
$iban = accounts[0]->iban;
?>
```

Returns all your external bank accounts. Ordered by Alias ascending. Can be paginated. 

### HTTP Request

`GET https://paywithfire.com/business/v1/me/externalaccounts`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
`page` | 1 | The page offset to return if paginating the results.
`count` | 20 | The count of requests to return per page if paginating the results.


## Create a new External Bank Account

```shell
# cat newaccount.json
{
	"alias": "Signs'R'Us",
	"currency": "GBP",
	"sortCode": "201922",
	"accountNumber": "37928374",
	"nameOnAccount": "SignsRUs"
}

# Post that to the API
curl https://paywithfire.com/business/v1/me/externalaccounts
  -X POST
  -d @newaccount.json
  -H "Authorization: privateToken"

{
	"externalAccountId": 379487
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

ExternalAccount signsrus = new ExternalAccount()
	.setAlias("Signs'R'Us")
	.setCurrency("GBP")
	.setSortCode("201922")
	.setAccountNumber("37928374")
	.setNameOnAccount("SignsRUs");

GenericResult res = businessAccount.externalAccounts().add(signsrus);

// Should be a positive result
if (Result.CREATED_OK == res.getResult()) {
	int accountId = res.getId();
	// ....
}
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

$signsrus = new PayWithFire_ExternalAccount(array(
	"alias" => "Signs'R'Us",
	"currency" => "GBP",
	"sortCode" => "201922",
	"accountNumber" => "37928374",
	"nameOnAccount" => "SignsRUs"
));

$res = $businessAccount->externalAccounts()->add($signsrus);

# Should be a positive result
if (res->result == "CREATED_OK") {
	$d = res->id;
	# ....
}
?>
```

To add a new bank account, post the details of the bank account as a JSON object. The `externalAccountId` of the account will be returned as a JSON object for you.

### HTTP Request

`POST https://paywithfire.com/business/v1/me/externalaccounts`

## Retrieve the details of an External Bank Account 

```shell
curl https://paywithfire.com/business/v1/me/externalaccounts/379487
  -X GET
  -H "Authorization: privateToken"

{
	"externalAccountId": 379487
	"alias": "Signs'R'Us",
	"currency": "GBP",
	"sortCode": "201922",
	"accountNumber": "37928374",
	"nameOnAccount": "SignsRUs"
}
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

try {
	ExternalAccount account 
		= businessAccount.externalAccounts().get(379487);
} catch (NoSuchExternalAccountException nseae) {
	// dang
}

// Now you can retrieve values from the account
String iban = account.getIBAN();
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

try {
	$account 
		= $businessAccount->externalAccounts()->get(379487);
} catch (PayWithFire_Exception_NoSuchFireAccount $nsfae) {
	# dang
}

# Now you can retrieve values from the account
$iban = $account->iban;
?>
```

You can retrieve the details of an external account by its `externalAccountId`. 

### HTTP Request

`GET https://paywithfire.com/business/v1/me/externalaccounts/{externalAccountId}`

Parameter | Description
--------- | -----------
`externalAccountId` | This is the ID of the external account to be returned.

## Delete an External Bank Account

```shell
curl https://paywithfire.com/business/v1/me/externalaccounts/379487
  -X DELETE
  -H "Authorization: privateToken"
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

try {
	GenericResult res 
		= businessAccount.externalAccounts().delete(379487);
} catch (NoSuchExternalAccountException nseae) {
	// dang
}
```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

try {
	$res = $businessAccount->externalAccounts()->delete(379487);
} catch (PayWithFire_Exception_NoSuchFireAccount $nseae) {
	# dang
}
?>
```

Delete an external account from your profile.

### HTTP Request

`DELETE https://paywithfire.com/business/v1/me/externalaccounts/{externalAccountId}`

Parameter | Description
--------- | -----------
`externalAccountId` | This is the ID of the external account to be deleted.

# Payments and Transfers

## Pay by Bank Transfer

```shell
# TODO: More detail needed
# cat paymentdetails.json
{
    "amount": 3748,
    "currency": "EUR",
    "narrative": "REF 2347839",
    "comment": "Paid for March Bill"
}

# Post that to the API
curl https://paywithfire.com/business/v1/me/makePayment?fromAccount=379487&toAccount=84654
  -X POST
  -d @paymentdetails.json
  -H "Authorization: privateToken"

```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

// Set up the payment
Payment payment = new Payment()
	.setAmount(3748)
	.setCurrency("EUR")
	.setNarrative("REF 2347839")
	.setComment("Paid for March Bill");

// Set the account to pay from - a Fire Account in this case
Account fromAccount 
	= new FireAccount().setId(84654);

// Set the account to pay to - an External Account in this case
Account toAccount 
	= new ExternalAccount().setId(379487);

try {
	GenericResult res 
		= businessAccount.makePayment(payment, fromAccount, toAccount);
} catch (PaymentExceedsLimitException pele) {
	// Check the status for reason
	pele.getStatus();
} catch (NoSuchExternalAccountException nseae) {
	// dang
}

```

```php
<?php
$businessAccount = new PayWithFireSDK(array(
	"environment" => "SANDBOX",
	"privateToken" => "privateToken"
));

# Set up the payment
$payment = new PayWithFire_Payment(array(
	"amount" => 3748,
	"currency" => "EUR",
	"narative" => "REF 2347839",
	"comment" => "Paid for March Bill"
));

# Set the account to pay from - a Fire Account in this case
$fromAccount = new PayWithFire_FireAccount(array("id" => 84654));

# Set the account to pay to - an External Account in this case
$toAccount = new PayWithFire_ExternalAccount(array("id" => 379487));

try {
	$res = $businessAccount->makePayment(array(
		"payment" => $payment, 
		"fromAccount" => $fromAccount, 
		"toAccount" => $toAccount
	));
} catch (PayWithFire_Exception_PaymentExceedsLimit $e) {
	# Check the status for reason
	$e->getMessage();

} catch (PayWithFire_Exception_NoSuchExternalAccount $nseae) {
	# dang
}
?>
```

You can pay from any of your Fire Accounts to an existing External Bank Account. 

Depending on the authorisation rules configured by your account administrator, the API call may only set up the transfer rather than actually execute it. Check the status of the response to know what happened. 

Some administrators may not allow payments via the API at all, and others can place restrictions on the total value of payments over a period of time. If you receive an error 403, check the status of the response for more information.


