---
title: Fire Business Account API Reference

language_tabs:
  - shell: cURL

toc_footers:

includes:

search: true
---

# Integrating to the Fire Business Account API

The Fire API allows you to deeply integrate Fire Account features into your application.

<aside class="notice">
**These docs are in BETA and subject to change at any moment. Please don't build production apps on the API yet.**
</aside>

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

<aside class="notice">
In the BETA period, the authentication process uses the Fire Business Account web application login. This
will change to a dedicated API token once the API is implemented.
</aside>

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

# Future Authentication / OAuth2 Bearer Tokens

```shell
# cat refreshtoken.json
```
```json
{
	"grantType": "refresh_token",
	"nonce": "<NONCE>",
	"refreshToken": "<APP_REFRESH_TOKEN>",
	"clientId": "<APP_CLIENT_ID>",
	"clientSecretHash": "sha256(<APP_CLIENT_SECRET><NONCE>)",
	"state": "data that will be returned to you"	
}
```
```shell
# Post that to the API
curl https://business.realexfire.com/api/login \
  -X POST \
  -D - \
  -d @refreshtoken.json
  
# Returns an access token
```
```json
{
	"expiresIn": 3600,
	"scope": "all",
	"tokenType": "bearer",
	"accessToken": "<ACCESS_TOKEN>"
}
```
```shell
# Keepalive example
ACCESS_TOKEN=<ACCESS_TOKEN>

curl https://business.realexfire.com/api/businesses/v1/me \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Access to the API is by temporary App Access Bearer Tokens. 

1. You must first log into the BUPA application and create a new Application in the Profile > API Tokens page. (You will need your PIN digits and 2-Factor Authentication device.) 
2. Give your application a Name and select the scope  you need the application to have (more on Scopes below). 
3. You will be provided with three pieces of information - the `App Refresh Token`, the `App Client ID` and the `App Client Secret`.  You may want to take note of the `App Client Secret` when it is displayed - it will not be displayed again without entering your PIN digits and 2-Factor Authentication code again.

You now use these pieces of data to retrieve a temporary App Access Token which you can use to access the API. The App Access Token expires within a relatively short time, so even if it is compromised, the attacker will not have long to use it. The `App Client Secret` is the most important piece of information to keep secret. This should only ever be stored on a backend server, and never in a front end client or mobile app. 

<aside class="warning">
If you ever accidentally reveal the Client Secret (or accidentally commit it to Github for instance) it is vital that you log into BUPA and rotate the App Tokens as soon as possible. Anyone who has these three pieces of data can access the API and your data, and potentially make payments from your account (depending on the scope of the tokens).  
</aside>

Once you have the authorization token, pass it as a header for every call. Whenever it expires, use the refresh token to get a new one again.
 
`Authorization: Bearer $ACCESS_TOKEN`

### HTTP Request

`POST https://business.realexfire.com/api/login`


### JSON Input

Parameter | Description
--------- | -----------
`grantType` | Always `refresh_token`.
`nonce` | A random alphanumeric string used as a salt for the `clientSecretHash` below.
`refreshToken` | The `App Refresh Token` from the Application Token page in BUPA.
`clientId` | The `App Client ID` from the Application Token page in BUPA.
`clientSecretHash` | The SHA256 hash of the `App Client Secret` from the Application Token page in BUPA concatenated with the `nonce` above.  
`state` | This can be any data that you can use to retrieve the user session on your side. Only really useful for 3-legged OAuth calls. 	

### Returns
A temporary App Access Token which can be used to access the API. 

Field | Description
--------- | -----------
`expiresIn` | The number of seconds this Access Token is valid. 
`scope` | The scope of the Access Token as a comma-separated string. This provides information on what API access it is allowed. See the section on Scope below.
`tokenType` | Always `bearer`.
`accessToken` | The temporary App Access Token.

## Scopes

Scopes are the list of permissions that you assign to an Application Token when you create it. An Application Access Tokens created from these Application Tokens will only have these permissions.

<aside class="notice">
It is important to only provide the minimum amount of access to an Application Token when you create it. This limits any potential damage done if the Application Token data is compromised.
</aside>

The list of scopes allowed for the Business API is as follows.

Scope | Description
----- | -----------
`get_accounts` | Read the list of Fire accounts in your profile.
`get_accounts:45` | Read access to just the specified account ID (`45` in this exmaple). You can add this multiple times to provide access to a specific list of accounts.  

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
    "accountName": "UK Invoicing Account", 
    "currency": "EUR", 
    "colour": "ORANGE"
}

# Post that to the API
curl https://business.realexfire.com/api/businesses/v1/accounts /
  -X POST /
  -d @newaccount.json /
  -H "Authorization: $AUTHORIZATION_TOKEN"

HTTP 204 No Content
```


To add a new Fire Account you just need a name and a currency. 

### HTTP Request

`POST https://business.realexfire.com/api/businesses/v1/accounts`

### JSON Input

Parameter | Description
--------- | -----------
`accountName` | A name to give to this Fire Account. This is not the same as the Name on the Account - that will always be your offical company name. This is an alias or nickname to help you identify the account. 
`currency` | Either "EUR" or "GBP"
`colour` | not used at present - set to "ORANGE" for now.

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
# First get the PIN Grid required
curl https://business.realexfire.com/api/businesses/v1/me/pingrid \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
    "positions": "245",
    "cap": "Guiness is Good for You!"
}

# Create the JSON object for the new account with the right digits of your PIN.
# cat newaccount-eur.json
{
    "country": "IE",
    "accountName": "Signs'R'Us",
    "currency": "EUR",
    "accountHolderName": "SignsRUs Limited",
    "bic": "AIBKIE2D",
    "iban": "IE41AIBK93338411111111",
    "authenticatorToken": "825806",
    "select0": "1",
    "select1": "2",
    "select2": "3"
}

# cat newaccount-gbp.json
{
    "country": "GB",
    "accountName": "Signs'R'Us",
    "currency": "GBP",
    "accountHolderName": "SignsRUs Limited",
    "nsc": "232211",
    "accountNumber": "12345678",
    "authenticatorToken": "825806",
    "select0": "1",
    "select1": "2",
    "select2": "3"
}

# Post that to the API
curl https://business.realexfire.com/api/businesses/v1/fundingsources \
  -X POST \
  -d @newaccount-gbp.json \
  -H "Authorization: $AUTHORIZATION_TOKEN"

HTTP 204 No Content
```

Adding a new external bank account is a highly sensitive action and is protected by digits of your PIN and the 2FA code from your phone. 
To add a new bank account, first retrieve a PIN Grid object, and then post the details of the bank account as a JSON object. 

### HTTP Request

`POST https://business.realexfire.com/api/businesses/v1/me/pingrid`
`POST https://business.realexfire.com/api/businesses/v1/fundingsources`

### JSON Input 

Parameter | Description
--------- | -----------
`accountName` | A name to give to this external Bank Account. This is not the same as the Name on the Account - this is an alias or nickname to help you identify the account. 
`currency` | Either `EUR` or `GBP`
`nsc` | If a GBP account, provide the Sort Code.
`accountNumber` | If a GBP account, provide the Account Number.
`bic` | If a EUR account, provide the BIC.
`iban` | If a EUR account, provide the IBAN.
`accountHolderName` | The name on the account. 
`country` | Either `IE` or `GB`  
`authenticatorToken` | The 6 digit code from your 2FA app on your phone, or create programmatically. 
`select0`, `select1` and `select2`  | The 3 digits requested from your PIN


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


# Payments
```shell
# Full details of an individual payment.
{
	"txnId": 30157,
	"refId": 26774,
	"txnType": {
		"type": "FX_INTERNAL_TRANSFER_FROM",
		"description": "Fx Internal Transfer From"
	},
	"from": {
		"type": "FIRE_ACCOUNT",
		"account": {
			"id": 1979,
			"alias": "Second EUR",
			"nsc": "991199",
			"accountNumber": "80502876",
			"bic": "CPAYIE2D",
			"iban": "IE57CPAY99119980502876"
		}
	},
	"to": {
		"type": "FIRE_ACCOUNT",
		"account": {
			"id": 1954,
			"alias": "GBP"
		}
	},
	"currency": {
		"code": "EUR",
		"description": "Euro"
	},
	"amountBeforeFee": 500,
	"feeAmount": 125,
	"amountAfterFee": 625,
	"balance": 35,
	"date": 1429695798917,
	"fxTradeDetails": { 
		"buyCurrency": "GBP",
		"sellCurrency": "EUR",
		"fixedSide": "SELL",
		"buyAmount": 359,
		"sellAmount": 500,
		"rate4d": 7180
	},
	"feeDetails": [ 
		{ 
			"percentage4d": 12500,
			"fixed": 0,
			"minimum": 125,
			"amountCharged": 125
		}
	]
}

# Condensed payment details when part of a list.
{
	"txnId": 30260,
	"refId": 26834,
	"ican": 1979,
	"txnType": {
		"type": "INTERNAL_TRANSFER_TO",
		"description": "Transfer"
	},
	"relatedParty": {
		"alias": "Main Account"
	},
	"currency": { 
		"code": "EUR",
		"description": "Euro"
	},
	"amountBeforeFee": 5000,
	"feeAmount": 0,
	"amountAfterFee": 5000,
	"balance": 8500,
	"myRef": "Transfer to main account",
	"date": "2015-04-29T22:56:48.867Z"
}
```
While there are many types of payments, they are all represented by the same JSON object with a different `txnType`.

The payment resource has the following attributes: 

Field | Description
--------- | -----------
`txnId` | The id of this side of the payment (each payment has two sides - a to and a from). This is used to get the details of the payment.
`refId` | The id of the payment.
`ican` | identifier for the Fire account _(assigned by Fire)_ _This field is only used in the condensed version._
`txnType` | The type of payment. `txnType.type` is the code, `txnType.description` is an English version. Use this to determine the "side" of the payment - e.g. `INTERNAL_TRANSFER_FROM` would be a positive payment from another account, `INTERNAL_TRANSFER_TO` is the negative side.
`relatedParty` | `relatedParty.alias` is the name of the account on the other side of the payment. _This field is only used in the condensed version._
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`.
`amountBeforeFee` | Amount of the payment before the fee was applied.
`feeAmount` | The amount of the fee.
`amountAfterFee` | Net amount lodged or taken from the account after fees applied.
`balance` | the balance of the account (in minor currency units - pence, cent etc. `434050` == `4,340.50 GBP` for a GBP account).
`myRef` | The comment/reference on the payment 
`date` | Date of the payment _(epoch date in full version, ISO date in condensed - will be fixed in a future release)_
`from` | The "from" side of the payment. `from.type` is the type of the account, and `from.account` is the details of that account. _This field is only present in the full version._
`to` | The "to" side of the payment. `to.type` is the type of the account, and `to.account` is the details of that account. _This field is only present in the full version._
`fxTradeDetails` | If this is a currency conversion, this will contain the FX rate and converted amount. _This field is only present in the full version._
`feeDetails` | The details of any fees applied. _This field is only present in the full version._


## List payments for an account
```shell
curl https://business.realexfire.com/api/businesses/v1/accounts/1979/payments \
  -X GET \
  -d "limit=25" \
  -d "offset=0" \
  -H "Authorization: $AUTHORIZATION_TOKEN"

{
	"total": 1,
	"dateRangeTo": 1430511042924,
	"payments": [ 
		{
			"txnId": 30260,
			"refId": 26834,
			"ican": 1979,
			"txnType": {
				"type": "INTERNAL_TRANSFER_TO",
				"description": "Transfer"
			},
			"relatedParty": {
				"alias": "Main Account"
			},
			"currency": { 
				"code": "EUR",
				"description": "Euro"
			},
			"amountBeforeFee": 5000,
			"feeAmount": 0,
			"amountAfterFee": 5000,
			"balance": 8500,
			"myRef": "Transfer to main account",
			"date": "2015-04-29T22:56:48.867Z"
		}
	]
}
```

Retrieve a list of payments against an account.

### HTTP Request

`GET  https://business.realexfire.com/api/businesses/v1/accounts/{accountId}/payments`

### Returns

An array of payments for `accountId` with a count (`total`)



# Transfers
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
    -X GET \
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

*Work in progress!*
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

*Work in progress!*
You can pay from any of your Fire Accounts to an existing External Bank Account. 


# Webhooks
Webhooks allow you to be notified of events as they happen on your Realex Fire accounts. This is useful if you have systems that need to know when things happen on your account, such as payments or withdrawals. 

A webhook is a URL that you set up on your backend. We can then send the details of various events to you at this URL as they happen. You can have many webhooks, and can configure each one to listen for different events in Fire. 

## Configuring your webhook settings
You can set up webhooks in the Business Account web application or use the API to configure them programmatically. 

## View Webhooks
```shell
curl https://business.realexfire.com/api/businesses/v1/webhooks \
  -X GET \
  -H "Authorization: $AUTHORIZATION_TOKEN" 

{
    "webhookEvents": [
        {
            "webhook": {
                "id": 7,
                "businessId": 2,
                "webhookUrl": "https://mysite.com/webhook/7384"
            },
            "events": [ "LODGEMENT_RECEIVED" ] 
        }
    ]
}
```

Retrieve a list of your existing webhooks 

`GET https://business.realexfire.com/api/businesses/v1/webhooks`

### Returns
An array of webhook event configuration objects.

Parameter | Description
--------- | -----------
`webhook` | The details of the webhook. `webhook.id` and `webhook.businessId` are identifiers, and `webhook.webhookUrl` is the actual URL of the webhook.
`events` | An array of Fire Account events that will trigger a call to this webhook. 

## Receiving a webhook at your server.
```shell
# This is the payload of the message you will receive for a lodgement. 
{ 
    "txnId": 1234,
    "refId": 13001,
    "txnType": { 
        "type": "LODGEMENT",
        "description": "Lodgement"
    },
    "from": { 
        "type": "WITHDRAWAL_ACCOUNT",
        "account": { 
            "id": 123,
            "alias": "Smyth and Co.",
            "bic": "DABAIE2D",
            "iban": "IE29AIBK93115212345654"
        }
    },
    "to": { 
        "type": "FIRE_ACCOUNT",
        "account": { 
            "id": 789,
            "alias": "Ticket Sales",
            "nsc": "991199",        
            "accountNumber": "00000000",
            "bic": "CPAYIE2D",
            "iban": "IE76CPAY99119900000000" 
        }
    },
    "currency": { 
        "code": "EUR", 
        "description": "Euro" 
    },
    "amountBeforeFee": 300,
    "feeAmount": 3,
    "amountAfterFee": 297,
    "balance": 397,
    "myRef": "Money for concert",
    "date": 1339511599000,
    "feeDetails": [       
        {       
          "amountCharged": 3,      
          "minimumAmount": 1,       
          "maximumAmount": 49,      
          "fixedPercentage4d": 10000        
        }
    ] 
}
```

When the data is sent to your webhook it will be signed and encoded using JWT (JSON Web Token). JWT is a compact URL-safe means of representing data to be transferred between two parties (see [JWT.io](http://jwt.io) for more details and to get a code library for your programming environment). While the data is the message is 
visibile to anyone, the signature is created using a shared secret that only you and Fire have access to, so you can be sure that it came from us. 

A JWT looks like this:

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM`
`0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV`
`9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

This needs to be decoded using the library from JWT.io. You should ensure that the signature is valid. There are a set of Webhook API Tokens in the Profile / Webhooks section of the Business Fire Account application. The Key ID (`kid`) in the JWT header will be the `Webhooks` public token, and you should use the corresponding private token as the secret to verify the signature on the JWT.  


