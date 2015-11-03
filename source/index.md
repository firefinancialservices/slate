---
title: Fire Business Account API Reference

language_tabs:
  - shell: cURL

toc_footers:

includes:

search: false
---

# Integrating to the Fire Business Account API

The Fire API allows you to deeply integrate Fire Business Account features into your application or back-office systems. 

Currently the API provides read-only access to your profile and accounts. The full transaction API is under development and 
will provide complete control of your Business Account. Every endpoint has its own permission, and the API will integrate 
seamlessly with the Role-based Access controls that will be available in a future release. 

The API exposes 3 main areas of functionality: financial functions, service information and service configuration. 

### Financial Functions
These functions provide access to your account details, payments, beneficiary accounts etc.   

### Service Information
These provide information about the fees and limits applied to your account. 

### Service Configuration
These provide information about your service configs - applications, webhooks, API tokens, etc.

<aside class="notice">
**The API and these docs are in BETA and subject to change at any moment. **
</aside>

# Authentication

```shell
# Set up Environment variables
CLIENT_ID=<APP_CLIENT_ID>
CLIENT_KEY=<APP_CLIENT_KEY>
REFRESH_TOKEN=<APP_REFRESH_TOKEN>
NONCE=`date +%s`
SECRET=( `echo -n $NONCE$CLIENT_KEY | sha256sum` )

# Get an Access Token from the API
curl https://business.paywithfire.com/api/businesses/v1/apps/accesstokens \
    -X POST \ 
    -H "Content-type: application/json" \
    -d "{"\""clientId"\"":"\""$CLIENT_ID"\"", "\""refreshToken"\"":"\""$REFRESH_TOKEN"\"","\""nonce"\"":"\""$NONCE"\"","\""grantType"\"":"\""Access Token"\"","\""clientSecret"\"":"\""${SECRET[0]}"\""}"
      
# Returns an access token
{
 "businessId": 23416,
 "applicationId": 113423,
 "expiry": 1443180240273,
 "permissions": [
      "PERM_BUSINESSES_GET_SERVICES",
      "PERM_BUSINESSES_GET_ACCOUNTS",
      "PERM_BUSINESSES_GET_ACCOUNT",
      "PERM_BUSINESSES_GET_PAYMENT",
      "PERM_BUSINESSES_GET_ACCOUNT_PAYMENTS",
      "PERM_BUSINESSES_GET_ACCOUNT_PAYMENTS_FILTER",
      "PERM_BUSINESSES_GET_FUNDING_SOURCES",
      "PERM_BUSINESSES_GET_FUNDING_SOURCE",
      "PERM_BUSINESSES_GET_FUNDING_SOURCE_PAYMENTS",
      "PERM_BUSINESSES_GET_WEBHOOKS",
      "PERM_BUSINESSES_GET_WEBHOOK_EVENT_TEST",
      "PERM_BUSINESSES_GET_LIMITS",
      "PERM_BUSINESSES_GET_FX_RATE",
      "PERM_BUSINESSES_GET_APPS",
      "PERM_BUSINESSES_GET_APP_PERMISSIONS",
      "PERM_BUSINESSES_GET_APPS_PERMISSIONS",
      "PERM_BUSINESSES_GET_WEBHOOK_TOKENS"
 ],
 "accessToken": "<ACCESS_TOKEN>"
}

# Example Call
ACCESS_TOKEN=<ACCESS_TOKEN>

curl https://business.paywithfire.com/api/businesses/v1/accounts \
  -H "Authorization: BEARER $ACCESS_TOKEN"
```

Access to the API is by Bearer Tokens. The process is somewhat similar to OAuth2.0, but with some changes to improve security. 

1. You must first log into the [BUPA application](https://business.paywithfire.com) and create a new Application in the *Profile* > *API* page. (You will need your PIN digits and 2-Factor Authentication device.) 
2. Give your application a Name and select the scope/permissions you need the application to have (more on Scopes below). 
3. You will be provided with three pieces of information - the App `Refresh Token`, `Client ID` and `Client Key`.  You need to take note of the `Client Key` when it is displayed - it will not be shown again.

You now use these pieces of data to retrieve a short-term Access Token which you can use to access the API. The Access Token expires within a relatively short time, so even if it is compromised, the attacker will not have long to use it. The `Client Key` is the most important piece of information to keep secret. This should only ever be stored on a backend server, and never in a front end client or mobile app. 

<aside class="warning">
If you ever accidentally reveal the Client Key (or accidentally commit it to Github for instance) it is vital that you log into BUPA and delete/recreate the App Tokens as soon as possible. Anyone who has these three pieces of data can access the API and your data, and in future potentially make payments from your account (depending on the scope of the tokens).  
</aside>

Once you have the access token, pass it as a header for every call, like so:  
 
`Authorization: BEARER $ACCESS_TOKEN`

Whenever it expires, create a new nonce and get a new one again.

### HTTP Request

`POST https://business.paywithfire.com/api/businesses/v1/apps/accesstokens`


### JSON Input

Parameter | Description
--------- | -----------
`grantType` | Always `Access Token`. (This will change to `refresh_token` in a future release.)
`nonce` | A random non-repeating number used as a salt for the `clientSecret` below. The simplest nonce is a unix time. 
`refreshToken` | The app's `Refresh Token` from the API page in BUPA.
`clientId` | The app's `Client ID` from the API page in BUPA.
`clientSecret` | The SHA256 hash of the `nonce` above and the app's `Client Key` from the API page in BUPA.

### Returns
A temporary App Access Token which can be used to access the API. 

Field | Description
--------- | -----------
`accessToken` | The temporary App Access Token you can use in further API calls.
`expiry` | The unix time (in milliseconds) that the access token will expire. Based on the server time.  
`permissions` | The permissions assigned to the Access Token as an array of strings. This provides information on what API access it is allowed. See the section on Scope below.
`applicationId` | The ID of the application you are using. 


## Permissions/Scopes

Scopes are the list of permissions that you assign to an Application when you create it. An Application Access Tokens created from these Applications will only have these permissions.

<aside class="notice">
It is important to only provide the minimum amount of access to an Application when you create it. This limits any potential damage done if the Application Token data is compromised.
</aside>

The list of scopes allowed for the Business API is as follows.

Scope | Description
----- | -----------
`PERM_BUSINESSES_GET_SERVICES` | Get Service Fees and Info
`PERM_BUSINESSES_GET_ACCOUNTS` | Read the list of Fire accounts in your profile. 
`PERM_BUSINESSES_GET_ACCOUNT` | Get the details of a single Fire account in your profile.
`PERM_BUSINESSES_GET_PAYMENT` | View details of a payment
`PERM_BUSINESSES_GET_ACCOUNT_PAYMENTS` | List payments on an Account
`PERM_BUSINESSES_GET_ACCOUNT_PAYMENTS_FILTER` | Filter payments on an Account
`PERM_BUSINESSES_GET_FUNDING_SOURCES` | List Withdrawal Accounts
`PERM_BUSINESSES_GET_FUNDING_SOURCE` | View details of a Withdrawal Account
`PERM_BUSINESSES_GET_FUNDING_SOURCE_PAYMENTS` | List payments on a Withdrawal Account
`PERM_BUSINESSES_GET_WEBHOOKS` | List all Webhooks
`PERM_BUSINESSES_GET_WEBHOOK_EVENT_TEST` | Send a test Webhook
`PERM_BUSINESSES_GET_LIMITS` | List all Limits
`PERM_BUSINESSES_GET_FX_RATE` | Check FX Rates
`PERM_BUSINESSES_GET_APPS` | List all Applications
`PERM_BUSINESSES_GET_APP_PERMISSIONS` | List all permissions for an Application
`PERM_BUSINESSES_GET_APPS_PERMISSIONS` | List all permissions available for Applications
`PERM_BUSINESSES_GET_WEBHOOK_TOKENS` | Get Webhook Tokens


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
curl https://business.paywithfire.com/api/businesses/v1/accounts \
  -X GET \
  -H "Authorization: BEARER $ACCESS_TOKEN"


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

`GET https://business.paywithfire.com/api/businesses/v1/accounts`

### Returns

An array of account objects.


## Retrieve the details of a Fire Account 

```shell
curl https://business.paywithfire.com/api/businesses/v1/accounts/1951 \
  -X GET \
  -H "Authorization: BEARER $ACCESS_TOKEN"

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

`GET https://business.paywithfire.com/api/businesses/v1/accounts/{ican}`

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
curl https://business.paywithfire.com/api/businesses/v1/fundingsources \
  -X GET \
  -H "Authorization: BEARER $ACCESS_TOKEN"


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

`GET https://business.paywithfire.com/api/businesses/v1/fundingsources`

### Returns

An array of external Bank accounts (referenced as `fundingSources` for legacy reasons).



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
curl https://business.paywithfire.com/api/businesses/v1/accounts/1979/payments \
  -X GET \
  -d "limit=25" \
  -d "offset=0" \
  -H "Authorization:  BEARER $ACCESS_TOKEN"

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

`GET  https://business.paywithfire.com/api/businesses/v1/accounts/{accountId}/payments`

### Returns

An array of payments for `accountId` with a count (`total`)



# Webhooks
Webhooks allow you to be notified of events as they happen on your Fire accounts. This is useful if you have systems that need to know when things happen on your account, such as payments or withdrawals. 

A webhook is a URL that you set up on your backend. We can then send the details of various events to you at this URL as they happen. You can have many webhooks, and can configure each one to listen for different events in Fire. 

## Configuring your webhook settings
You can set up webhooks in the Business Account web application or use the API to configure them programmatically. 

## View Webhooks
```shell
curl https://business.paywithfire.com/api/businesses/v1/webhooks \
  -X GET \
  -H "Authorization: BEARER $ACCESS_TOKEN"

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

`GET https://business.paywithfire.com/api/businesses/v1/webhooks`

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
   "generationTime":1339511604000,
   "status":"LIVE",
   "events":[{
     "type":"lodgement.received",
     "user": "fire-system",
     "data":{  
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
    }]
}
```

You will recieve an array of events as they occur. In general there will be only one event per message, but as your volume increases, we will gather all events in a short time-window into one call to your webhook. This reduces the load on your server. 

When the data is sent to your webhook it will be signed and encoded using JWT (JSON Web Token). JWT is a compact URL-safe means of representing data to be transferred between two parties (see [JWT.io](http://jwt.io) for more details and to get a code library for your programming environment). While the data is the message is 
visibile to anyone, the signature is created using a shared secret that only you and Fire have access to, so you can be sure that it came from us. 

A JWT looks like this:

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM`
`0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV`
`9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

This needs to be decoded using the library from JWT.io. You should ensure that the signature is valid. There are a set of Webhook API Tokens in the Profile / Webhooks section of the Business Fire Account application. The Key ID (`kid`) in the JWT header will be the `Webhooks` public token, and you should use the corresponding private token as the secret to verify the signature on the JWT.  


