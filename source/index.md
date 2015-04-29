---
title: Fire Business Account API Reference

language_tabs:
  - shell: cURL

toc_footers:

includes:

search: true
---

# Integrating to the Fire Business Account API

The Fire API allows you to deeply integrate our account features into your application.

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
  -D - \
  -d @logindetails.json

# Grab the Authorization Token from the headers (at)

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
```shell
# Keepalive example
AUTHORIZATION_TOKEN=<AUTHORIZATION-TOKEN>

curl https://business.realexfire.com/api/businesses/v1/operatingcountries \
  -H "Authorization: $AUTHORIZATION_TOKEN"
```

`[BETA]`
In the BETA period, the authentication process uses the Fire Business Account web application login. This
will change to a dedicated API token once the API is implemented.

Get an authorization token by passing your business id, email and password to the login endpoint

### HTTP Request

`POST https://business.realexfire.com/api/login`

The authorization token is returned as a header (`at`). This token will expire after 5 minutes 
of inactivity, so a regular call (once every minute say) to an inexpensive endpoint (say `Operating Countries`) 
is needed until you sign out.

Once you have the authorization token, pass it as a header for every call. 
`Authorization: $AUTHORIZATION_TOKEN`

### JSON Input

Parameter | Description
--------- | -----------
`businessClientId` | The alpha-numeric business ID you set during sign up, and used to log into the Business Account application.
`emailAddress` | The email address of the authorized user.
`password` | The password for the authorized user.

### Returns
The user and business profiles as returned by the `Me` endpoint (`/api/businesses/v1/me`). 

# Fire Accounts 

```shell
# JSON representation of a Fire Account
{
     "ican": 1951,
     "name": "Main Account",
     "balance": 434050,
     "ciban": "IE54CPAY99119911111111",
     "cbic": "CPAYIE2D",
     "cnsc": "991199",
     "ccan": "11111111",
     "currency": {
        "description": "Euro",
        "code": "EUR"
     },
     "defaultAccount": true,
     "status": {
        "type": "LIVE",
        "description": "Live"
     },
     "colour": {
        "name": "ORANGE"
     }   
}
```    

Fire Accounts are the Fire equivalent of a bank account from bank, but with extra features you won't find anywhere else. 

The resource has the following attributes: 

Field | Description
--------- | -----------
`ican` | identifier for the Fire account _(assigned by Fire)_ 
`name` | the name the user gives to the account to help them identify it. 
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`.
`balance` | the balance of the account (in minor currency units - pence, cent etc. `434050` == `4,340.50 GBP` for a GBP account).
`cbic` | the BIC of the account (provided if currency is `EUR`). 
`ciban` | the IBAN of the account (provided if currency is `EUR`). 
`cnsc` | the Sort Code of the account. 
`ccan` | the Account Number of the account. 
`defaultAccount` | `true` if this is the default account for this currency. This will be the account that general fees are taken from (as opposed to per-transaction fees). 
`status` | _Not used at present_
`color` | _Not used at present_

## List all Fire Accounts

```shell
curl https://business.realexfire.com/api/businesses/v1/accounts \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN"


{
	"accounts": [
		{
		     "ican": 1951,
		     "name": "Main Account",
		     "balance": 434050,
		     "ciban": "IE54CPAY99119911111111",
		     "cbic": "CPAYIE2D",
		     "cnsc": "991199",
		     "ccan": "11111111",
		     "currency": {
		        "description": "Euro",
		        "code": "EUR"
		     },
		     "defaultAccount": true,
		     "status": {
		        "type": "LIVE",
		        "description": "Live"
		     },
		     "colour": {
		        "name": "ORANGE"
		     }   
		}
	]
}
```


Returns all your Fire Accounts. Ordered by Alias ascending. Can be paginated. 

### HTTP Request

`GET https://business.realexfire.com/api/businesses/v1/accounts`

### Returns

An array of account objects.

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
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
	"cban": 924733, 
	"currency": "GBP",
	"balance": 0,
	"sortCode": "232221",
	"accountNumber": "34658388",
	"nameOnAccount": "Tim's Pen Shop"
}
```


To add a new Fire Account you just need a name and a currency. The details of the new account will be returned to you.

### HTTP Request

`POST https://paywithfire.com/business/v1/me/accounts`

## Retrieve the details of a Fire Account 

```shell
curl https://business.realexfire.com/api/businesses/v1/accounts/1951 \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
     "ican": 1951,
     "name": "Main Account",
     "balance": 434050,
     "ciban": "IE54CPAY99119911111111",
     "cbic": "CPAYIE2D",
     "cnsc": "991199",
     "ccan": "11111111",
     "currency": {
        "description": "Euro",
        "code": "EUR"
     },
     "defaultAccount": true,
     "status": {
        "type": "LIVE",
        "description": "Live"
     },
     "colour": {
        "name": "ORANGE"
     }   
}
```


You can retrieve the details of a Fire Account by its `ican`. 

### HTTP Request

`GET https://business.realexfire.com/api/businesses/v1/accounts/{ican}`

Parameter | Description
--------- | -----------
`ican` | This is the internal account ID of the Fire Account to be returned.

# External Bank Accounts 

```shell
# JSON representation of an External Account
{
   "id": 742,
   "accountName": "BoI Current Account",
   "bic": "BOFIIE2DXXX",
   "iban": "IE86BOFI90535211111111",
   "nsc": null,
   "accountNumber": null,
   "accountHolderName": "Brian Johnson",
   "currency": {
      "code": "EUR",
      "description": "Euro"
   },
   "dateCreated" : 1423751574577,
   "status": {
      "description": "Validated",
      "type": "LIVE"
   },
   "country": {
      "description": "Ireland",
      "code": "IE"
   }
}
```    

You can add bank accounts from other banks to your profile and transfer to these by bank transfer. These can either be your accounts in other banks or can be suppliers or customers you need to pay by bank transfer. 

The resource has the following attributes: 

Field | Description
--------- | -----------
`id` | identifier for the Bank account _(assigned by Pay with Fire)_ 
`accountName` | the name the user gives to the bank account to help them identify the account. 
`bic` | the BIC of the account if currency is `EUR`. 
`iban` | the IBAN of the account if currency is `EUR`. 
`nsc` | the Sort Code of the account if currency is `GBP`. 
`accountNumber` | the Account Number of the account if currency is `GBP`. 
`accountHolderName` | the name on the external bank account. 
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`
`dateCreated` | the date/time the external account was added. Milliseconds since the epoch (1970).
`status` | the status of the external account. When the account is first added it is in `PENDING` state until the valided by clicking the link in the email sent to the authorized user's email address. `status.description` is the English text to describe the `status.type`.
`country` | the country of the account. `country.description` is the English version of the 2-letter code in `country.code`.

## List all External Bank Accounts

```shell
curl https://business.realexfire.com/api/businesses/v1/fundingsources \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN" 


{
	"fundingSources": [
		{
		   "id": 742,
		   "accountName": "BoI Current Account",
		   "bic": "BOFIIE2DXXX",
		   "iban": "IE86BOFI90535211111111",
		   "nsc": null,
		   "accountNumber": null,
		   "accountHolderName": "Brian Johnson",
		   "currency": {
		      "code": "EUR",
		      "description": "Euro"
		   },
		   "dateCreated" : 1423751574577,
		   "status": {
		      "description": "Validated",
		      "type": "LIVE"
		   },
		   "country": {
		      "description": "Ireland",
		      "code": "IE"
		   }
		}
	]
}
```


Returns all your external bank accounts. Ordered by Alias ascending. Can be paginated. 

### HTTP Request

`GET https://business.realexfire.com/api/businesses/v1/fundingsources`

### Returns

An array of external Bank accounts (referenced as `fundingSources` for legacy reasons).

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
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
	"externalAccountId": 379487
}
```


To add a new bank account, post the details of the bank account as a JSON object. The `externalAccountId` of the account will be returned as a JSON object for you.

### HTTP Request

`POST https://paywithfire.com/business/v1/me/externalaccounts`

## Retrieve the details of an External Bank Account 

```shell
curl https://business.realexfire.com/api/businesses/v1/fundingsources/742 \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
   "id": 742,
   "accountName": "BoI Current Account",
   "bic": "BOFIIE2DXXX",
   "iban": "IE86BOFI90535211111111",
   "nsc": null,
   "accountNumber": null,
   "accountHolderName": "Brian Johnson",
   "currency": {
      "code": "EUR",
      "description": "Euro"
   },
   "dateCreated" : 1423751574577,
   "status": {
      "description": "Validated",
      "type": "LIVE"
   },
   "country": {
      "description": "Ireland",
      "code": "IE"
   }
}
```


You can retrieve the details of an external account by its `id`. 

### HTTP Request

`GET https://business.realexfire.com/api/businesses/v1/fundingsources/{id}`

Parameter | Description
--------- | -----------
`id` | This is the ID of the external account to be returned.


# Payments and Transfers
You can transfer instantly between any two of your Fire Accounts in either currency, or perform a bank transfer to 
a pre-existing External Bank account. 

## Transfer between Fire Accounts in the same currency

```shell
# cat transferdetails.json
{
    "icanFrom": 1951, 
    "icanTo": 1979, 
    "currency": "EUR", 
    "amount": 50, 
    "ref": "Cover the bills for April 2015"
}

# Post that to the API
curl https://business.realexfire.com/api/businesses/v1/accounts/transfer \
  -X POST \
  -d @transferdetails.json \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
    "refId": 32424
}
```

To transfer between two of your Fire Accounts in the same currency, post the details of the transfer as a JSON object. The `refId` of the transfer will be returned to you.

### HTTP Request

`POST https://business.realexfire.com/api/businesses/v1/accounts/transfer`

### Returns
The `refId` of the resulting transfer.

## Transfer between Fire Accounts in different currencies

```shell
# First, get the fee details object for FX transfers from this account.
curl https://business.realexfire.com/api/businesses/v1/services/FX_INTERNAL_TRANSFER?ican=1954 \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
   "feeRule": {
      "feeRuleId": 19,
      "fixed": 0,
      "percentage4d": 12500,
      "minimum": 125
   },
   "currency": {
      "code": "GBP",
      "description": "Sterling"
   }
}

# ------- This seems all wrong????? ------
# To get an estimate of the currency conversion rate that will be used:
curl https://business.realexfire.com/api/businesses/v1/fx/rate?buyCurrency=GBP&sellCurrency=EUR&fixedSide=BUY&amount=10000 \
    -x GET \
    -H "Authorization: $AUTHORIZATION_TOKEN"
    
{
    "buyCurrency":"GBP",
    "sellCurrency":"EUR",
    "fixedSide":"BUY",
    "buyAmount":10000,
    "sellAmount":13883,
    "rate4d": 13883
}

# Then use the feeRuleId in the transfer object to agree to the fees.
# cat transferdetails.json
{
    "icanFrom": 1954, 
    "icanTo": 1951, 
    "amount": "500000", 
    "amountCurrency": "GBP", 
    "ref": "Transfer invoice payments back to Euro", 
    "feeRuleId": 19
}

# Post that to the API
curl https://business.realexfire.com/api/businesses/v1/fx/transfer \
  -X POST \
  -d @transferdetails.json \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
   "refId":26835,
   "txnId":30261,
   "txnType":{
      "type":"FX_INTERNAL_TRANSFER_FROM",
      "description":"Fx Internal Transfer From"
   },
   "from":{
      "type":"FIRE_ACCOUNT",
      "account":{
         "id":1954,
         "alias":"GBP",
         "nsc":"232221",
         "accountNumber":"05379385"
      }
   },
   "to":{
      "type":"FIRE_ACCOUNT",
      "account":{
         "id":1951,
         "alias":"EUR"
      }
   },
   "currency":{
      "code":"GBP",
      "description":"Sterling"
   },
   "amountBeforeFee":500000,
   "feeAmount":,
   "amountAfterFee":506250,
   "balance":31000,
   "myRef":"Transfer invoice payments back to Euro",
   "date":1430348793650,
   "fxTradeDetails":{
      "buyCurrency":"EUR",
      "sellCurrency":"GBP",
      "fixedSide":"SELL",
      "buyAmount":694000,
      "sellAmount":500000,
      "rate4d":13880
   },
   "feeDetails":[
      {
         "percentage4d":12500,
         "fixed":0,
         "minimum":125,
         "amountCharged":6250
      }
   ]
}
```

To transfer between two of your Fire Accounts in different currencies, you must first know what fee applies. You can get this by requesting the `FX_INTERNAL_TRANSFER` service for the source account.

`GET https://business.realexfire.com/api/businesses/v1/services/FX_INTERNAL_TRANSFER?ican={ican}`

This returns a fee details object which you can use to determine the fees you will be charged. 

Include this `feeRuleId` in the POST request to explicitly agree to the fees you will be charged. 

### HTTP Request

`POST https://business.realexfire.com/api/businesses/v1/fx/transfer`

### Returns
The payment object for the resulting transfer.


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
  -H "Authorization: $AUTHORIZATION_TOKEN"

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


