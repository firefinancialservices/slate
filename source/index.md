---
title: fire.com Business Account API Reference

language_tabs:
  - shell: cURL

toc_footers:

includes:

search: false
---

# Integrating to the fire.com Business Account API

The fire.com API allows you to deeply integrate Business Account features into your application or back-office systems. 

The API provides read access to your profile, accounts and transactions, event-driven notifications of activity on the 
account and payment initiation via batches. Each feature has it's own HTTP endpoint and every endpoint has its own permission.

The API exposes 3 main areas of functionality: financial functions, service information and service configuration. 

### Financial Functions
These functions provide access to your account details, transactions, payee accounts, payment initiation etc.   

### Service Information
These provide information about the fees and limits applied to your account. 

### Service Configuration
These provide information about your service configs - applications, webhooks, API tokens, etc.

### Service Endpoint
The Business API is available at 

`https://api.fire.com/business/v1/`

<aside class="notice">
**The API and these docs are in BETA and subject to change.**
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
curl https://api.fire.com/business/v1/apps/accesstokens \
    -X POST \ 
    -H "Content-type: application/json" \
    -d "{"\""clientId"\"":"\""$CLIENT_ID"\"", "\""refreshToken"\"":"\""$REFRESH_TOKEN"\"","\""nonce"\"":"\""$NONCE"\"","\""grantType"\"":"\""AccessToken"\"","\""clientSecret"\"":"\""${SECRET[0]}"\""}"
      
# Returns an access token
{
 "businessId": 23416,
 "applicationId": 113423,
 "expiry": 1443180240273,
 "permissions": [
      "PERM_BUSINESSES_GET_SERVICES",
      "PERM_BUSINESSES_GET_ACCOUNTS",
      "PERM_BUSINESSES_GET_ACCOUNT",
      "PERM_BUSINESSES_GET_ACCOUNT_TRANSACTIONS",
      "PERM_BUSINESSES_GET_ACCOUNT_TRANSACTIONS_FILTER",
      "PERM_BUSINESSES_GET_FUNDING_SOURCES",
      "PERM_BUSINESSES_GET_FUNDING_SOURCE",
      "PERM_BUSINESSES_GET_FUNDING_SOURCE_TRANSACTIONS",
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

curl https://api.fire.com/business/v1/accounts \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Access to the API is by Bearer Tokens. The process is somewhat similar to OAuth2.0, but with some changes to improve security. 

1. You must first log into the [firework online application](https://business.fire.com) and create a new Application in the *Profile* > *API* page. (You will need your PIN digits and 2-Factor Authentication device.) 
2. Give your application a Name and select the scope/permissions you need the application to have (more on Scopes below). 
3. You will be provided with three pieces of information - the App `Refresh Token`, `Client ID` and `Client Key`.  You need to take note of the `Client Key` when it is displayed - it will not be shown again.

You now use these pieces of data to retrieve a short-term Access Token which you can use to access the API. The Access Token expires within a relatively short time, so even if it is compromised, the attacker will not have long to use it. The `Client Key` is the most important piece of information to keep secret. This should only ever be stored on a backend server, and never in a front end client or mobile app. 

<aside class="warning">
If you ever accidentally reveal the Client Key (or accidentally commit it to Github for instance) it is vital that you log into firework online and delete/recreate the App Tokens as soon as possible. Anyone who has these three pieces of data can access the API to view your data and set up payments from your account (depending on the scope of the tokens).  
</aside>

Once you have the access token, pass it as a header for every call, like so:  
 
`Authorization: Bearer $ACCESS_TOKEN`

Whenever it expires, create a new nonce and get a new access token again.

### HTTP Request

`POST https://api.fire.com/business/v1/apps/accesstokens`



### JSON Input

Parameter | Description
--------- | -----------
`grantType` | Always `AccessToken`. (This will change to `refresh_token` in a future release.)
`nonce` | A random non-repeating number used as a salt for the `clientSecret` below. The simplest nonce is a unix time. 
`refreshToken` | The app's `Refresh Token` from the API page in firework online.
`clientId` | The app's `Client ID` from the API page in firework online.
`clientSecret` | The SHA256 hash of the `nonce` above and the app's `Client Key`. The Client Key will only be shown to you when you create the app, so don't forget to save it somewhere safe.

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
`PERM_BUSINESS_GET_SERVICES` | Get Service Fees and Info
`PERM_BUSINESS_GET_ACCOUNTS` | Read the list of fire.com accounts in your profile. 
`PERM_BUSINESS_GET_ACCOUNT` | Get the details of a single fire.com account in your profile.
`PERM_BUSINESS_GET_ACCOUNT_TRANSACTIONS` | List transactions on an Account
`PERM_BUSINESS_GET_ACCOUNT_TRANSACTIONS_FILTER` | Filter transactions on an Account
`PERM_BUSINESS_POST_PAYMENT_REQUEST` | Create a Payment Request
`PERM_BUSINESS_GET_PAYMENT_REQUESTS` | List all sent Payment Requests and their details
`PERM_BUSINESS_PUT_PAYMENT_REQUEST_STATUS` | Update a Payment Request status
`PERM_BUSINESS_GET_PUBLIC_PAYMENT_REQUEST` | Get a public payment request
`PERM_BUSINESS_GET_PAYMENT_REQUEST_REPORTS` | Get a report of the total amount paid to a payment request
`PERM_BUSINESS_GET_PAYMENT_REQUEST_TRANSACTIONS` | Get a paged list of all payments to a payment request
`PERM_BUSINESS_GET_FUNDING_SOURCES` | List Payee Accounts
`PERM_BUSINESS_GET_FUNDING_SOURCE` | View details of a Payee Account
`PERM_BUSINESS_GET_FUNDING_SOURCE_TRANSACTIONS` | List transactions to and from Payee Accounts
`PERM_BUSINESS_GET_WEBHOOKS` | List all Webhooks
`PERM_BUSINESS_GET_WEBHOOK_EVENT_TEST` | Send a test Webhook
`PERM_BUSINESS_GET_WEBHOOK_TOKENS` | Get Webhook Tokens
`PERM_BUSINESS_GET_LIMITS` | List all Limits
`PERM_BUSINESS_GET_FX_RATE` | Check FX Rates
`PERM_BUSINESS_GET_APPS` | List all Applications
`PERM_BUSINESS_GET_APP_PERMISSIONS` | List all permissions for an Application
`PERM_BUSINESS_GET_APPS_PERMISSIONS` | List all permissions available for Applications
`PERM_BUSINESS_GET_USERS` |  List all Users
`PERM_BUSINESS_GET_USER` | Get the details of a User
`PERM_BUSINESS_GET_USER_ADDRESS` | Get the address of a User
`PERM_BUSINESS_GET_CARDS` | List Debit Cards
`PERM_BUSINESS_GET_MY_CARD_TRANSACTIONS` | Get a list of Debit Card transactions
`PERM_BUSINESS_GET_MY_CARD_TRANSACTIONS_FILTERED` | Get a filtered list of Debit Card Transactions
`PERM_BUSINESS_GET_ACTIVITIES` | Get a list of Account Activities
`PERM_BUSINESS_GET_BATCHES` | List all Batches
`PERM_BUSINESS_POST_BATCHES` | Create a new Batch
`PERM_BUSINESS_GET_BATCH` | Get the details of a Batch
`PERM_BUSINESS_POST_BATCH_INTERNALTRANSFERS` | Add an Internal Transfer to a Batch
`PERM_BUSINESS_POST_BATCH_BANKTRANSFERS` | Add a Bank Transfer to a Batch
`PERM_BUSINESS_DELETE_BATCH_INTERNALTRANSFERS` | Remove an internal transfer from a batch
`PERM_BUSINESS_DELETE_BATCH_BANKTRANSFERS` | Remove a bank transfer from a batch
`PERM_BUSINESS_GET_BATCH_INTERNALTRANSFERS` | List items for an Internal Transfer Batch
`PERM_BUSINESS_GET_BATCH_BANKTRANSFERS` | List items for a Bank Transfer Batch
`PERM_BUSINESS_GET_BATCH_NEWPAYEES` | List items for a New Payee Batch
`PERM_BUSINESS_GET_BATCH_APPROVALS` | List approvals for a Batch
`PERM_BUSINESS_DELETE_BATCH` | Cancel a Batch
`PERM_BUSINESS_PUT_BATCH` | Submit a batch
`PERM_BUSINESS_POST_ACCOUNTS` | Add a new Account
`PERM_BUSINESS_POST_CARDS` | Create a new card
`PERM_BUSINESS_POST_MY_CARD_BLOCK` | Blocks a card
`PERM_BUSINESS_POST_MY_CARD_UNBLOCK` | Unblocks a card





# Pagination and Response Objects

```shell
# JSON representation of a list object
{
  "total": 2,
  "{itemTypes}": [
    { },  
    { } 
  ]
}
```

Lists are returned in a specific format (a *fire.com list*). An object containing some metadata about the list and the list itself is provided. 

fire.com List Meta Data | Desc
------------------- | ----
`total` | The total number of items in the list. 
`{itemTypes}` | Varies depending on the type of items in the list, but contains an array of `{itemType}`s.

## Pagination
```shell
curl https://api.fire.com/business/v1/{itemTypes} \
  -X GET -G \
  -d "offset=0" \
  -d "limit=10" \
  -d "orderBy=DATE" \
  -d "order=DESC" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Pagination applies to GET request endpoints. 

Parameter | Description
--------- | -----------
`orderBy` | Currently defaults to `DATE`. No other options at this time. 
`order` | Either `ASC` or `DESC`
`limit` | The number of records to return. Defaults to `10` - max is `200`.
`offset` | The page offset. Defaults to `0`. This is the record number that the returned list will start at. E.g. `offset` = `40` and `limit` = `20` will return records 40 to 59.



# fire.com Accounts 

```shell
# JSON representation of a fire.com Account
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
     "status": "LIVE",
     "colour":"ORANGE"
}
```    

fire.com Accounts are the equivalent of a bank account from bank. 

The resource has the following attributes: 

Field | Description
--------- | -----------
`ican` | identifier for the fire.com account _(assigned by fire.com)_ 
`name` | the name the user gives to the account to help them identify it. 
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`.
`balance` | the balance of the account (in minor currency units - pence, cent etc. `434050` == `4,340.50 GBP` for a GBP account).
`cbic` | the BIC of the account (provided if currency is `EUR`). 
`ciban` | the IBAN of the account (provided if currency is `EUR`). 
`cnsc` | the Sort Code of the account. 
`ccan` | the Account Number of the account. 
`defaultAccount` | `true` if this is the default account for this currency. This will be the account that general fees are taken from (as opposed to per-transaction fees). 
`status` | Either _LIVE_ or _MIGRATED_
`color` | _Internal Use_

## List all fire.com Accounts

```shell
curl https://api.fire.com/business/v1/accounts \
  -X GET \
  -H "Authorization: Bearer $ACCESS_TOKEN"


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
		     "status": "LIVE",
             "colour":"ORANGE"
		}
	]
}
```


Returns all your fire.com Accounts. Ordered by Alias ascending. Can be paginated. 

### HTTP Request

`GET https://api.fire.com/business/v1/accounts`

### Returns

An array of account objects.


## Retrieve the details of a fire.com Account 

```shell
curl https://api.fire.com/business/v1/accounts/1951 \
  -X GET \
  -H "Authorization: Bearer $ACCESS_TOKEN"

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
     "status": "LIVE",
     "colour":"ORANGE"
}
```

You can retrieve the details of a fire.com Account by its `ican`. 

### HTTP Request

`GET https://api.fire.com/business/v1/accounts/{ican}`

Parameter | Description
--------- | -----------
`ican` | This is the internal account ID of the fire.com Account to be returned.


## Add New Account

```shell

{ 
    "accountName": "Test",
    "currency": "EUR",
    "acceptFeesAndCharges": true|false 
} 

# Response Body 
204 no content

```

Creates a new fire.com account.

<aside class="warning">
Please note there is a charge associated with creating a new account. 
</aside>


### HTTP Request

`POST https://api.fire.com/business/v1/accounts`



# Users

The fire.com users are the business users you have setup on your account. 

```shell
# Full details of an individual user.

{
    "id": 30157,
    "emailAddress": "user@user.com",
    "firstName": "User",
    "lastName": "User",
    "mobileNumber": "+353876543829",
    "role": {
        "code": "ADMIN",
        "description": "Administrator"
    },
    "status": {
        "code": "LIVE",
        "description": "User has full access to fire.com applications."
    },
    "lastLogin": "2017-02-11T19:56:40.977Z",
    "userCvl": {
        "code": "FULL",
        "description": "Full"
    },
    "mobileApplicationDetails": {
        "mobileApplicationId": 100801,
        "clientId": 7492,
        "status": "LIVE",
        "businessUserId": 30157,
        "deviceName": "iPhone",
        "os": "iOS",
        "deviceOSVersion": "12.4"
    }
}

```  

The resource has the following attributes:

Field | Description
--------- | -----------
`id` | the id of the user.
`emailAddress` | the email address of the user.
`firstName` | the user’s first name.
`lastName` | the user’s last name.
`mobileNumber` | the user’s mobile number.
`role` | role assigned to the user.
`status` | user’s status type id.
`lastLogin` | user’s last log-in date.
`userCvl` | user’s CVL type ID.
`mobileApplicationDetails` | business mobile application details.

The user status can be one of the following:

Field | Description
--------- | -----------
`INVITE_SENT` | initial invitation sent.
`SMS_CODE_SENT` | user has been sent an SMS verification message during the registration flow.
`LIVE` | user is LIVE.
`CLOSED` | user has been closed.

The user CVL can be one of the following:

Field | Description
--------- | -----------
`BASIC` | not fully verified.
`FULL` | fully verified.

## List all fire.com Users

```shell
# JSON representation of list all users

{
  "users": [
    {
      "id", 1001,
      "emailAddress":"terry@example.com",
      "firstName": "Terry",
      "lastName": "Example",
      "mobileNumber":"+353855555555",
      "status": "LIVE",
      "lastLogin": "2012-01-20T11:21:35.000Z"
    }
  ]
}

```   

Returns list of all users on your fire.com account.

### HTTP REQUEST

`GET https://api.fire.com/business/v1/users`

### RETURNS

An array of user objects.

## List User Details

```shell
# JSON representation of list all users

{
   "id", 1001,
   "emailAddress":"terry@example.com",
   "firstName": "Terry",
   "lastName": "Example",
   "mobileNumber":"+353855555555",
   "role": "FULL_USER",
   "status": "LIVE",
   "lastLogin": "2012-01-20T11:21:35.000Z"
}


```   

Returns details of a specific fire.com user. 

### HTTP REQUEST

`GET https://api.fire.com/business/v1/users/{userId}`

### RETURNS

A user object.


# Cards
```shell
# JSON representation of a card object
{
      "cardId": 97002,
      "expiryDate": "2019-01-22T00:00:00.000Z",
      "maskedPan": "4319***********1234",
      "status": "LIVE",
      "userId": 1056,
      "firstName": "chfn",
      "lastName": "chln",
      "emailAddress": "test@test.com"
}

```   

You can create multiple debit cards which can be linked to your fire.com accounts. 

The resource has the following attributes:

Field | Description
--------- | -----------
`cardId` | card id assigned by fire.com
`expiryDate` | card expiry dare
`maskedPan` |card number (masked)
`status`| card status
`userId` | card user id assigned by fire.com
`firstName` | card user first name
`lastName`| card user last name
`emailAddress`| card user email address

## View List of Cards

```shell
#List of card objects

{
  "cards": [
    {
      "cardId": 97002,
      "expiryDate": "2019-01-22T00:00:00.000Z",
      "maskedPan": "4319***********1234",
      "status": "LIVE",
      "userId": 1056,
      "firstName": "chfn",
      "lastName": "chln",
      "emailAddress": "test@test.com"
    }
  ]
}
```
Returns a list of cards related to your fire.com account.

### HTTP REQUEST
`GET https://api.fire.com/business/v1/cards`

### RETURNS
An array of card objects.

## Create Card

```shell
#Create new card

{ 
  "eurIcan":2150, 
  "gbpIcan":2152, 
  "userId":3138, 
  "cardPin":"1111", 
  "acceptFeesAndCharges": true|false, 
  "addressType":"BUSINESS|HOME" 
} 

Return: 
{ 
  "cardId":2645, 
  "expiryDate":"2020-06-30T00:00:00.000Z", 
  "maskedPan":"537455******0386", 
  "status": CREATED_ACTIVE
} 
```
Creates a new card.

<aside class="warning">
Please note there is a charge associated with creating a new card. 
</aside>

### HTTP REQUEST
`POST https://api.fire.com/business/v1/cards`

### RETURNS
New card object.

## List Card Transactions

```shell
#List Card Transactions

{
  "total": 1,
  "dateRangeTo": 1547744156603,
  "transactions": [
    {
      "txnId": 97904,
      "refId": 62887,
      "ican": 1607,
      "type": "CARD_ECOMMERCE_CREDIT",
      "relatedParty": {
        "type": "CARD_MERCHANT",
        "cardMerchant": {
          "acquirerIdDe32": "00000004619",
          "additionalAmtDe54": null,
          "authCodeDe38": null,
          "billAmt": 1510,
          "billCcy": "826",
          "expiryDate": "0",
          "mccCode": "5999",
          "merchIdDe42": "273141000156182",
          "merchNameDe43": "AMZ*Aisence online5 rue PlaetisAMAZON.FRL2338     LUXLUX",
          "posDataDe61": null,
          "posTermnlDe41": null,
          "posDataDe22": "690550S99001",
          "procCode": "200000",
          "respCodeDe39": null,
          "retRefNoDe37": null,
          "statusCode": "43",
          "token": "431524366",
          "txnAmt4d": 169900,
          "txnCcy": "978",
          "txnCtry": "LUX",
          "txnDesc": "AMZ*Aisence online5 rue PlaetisAMAZON.FRL2338     LUXLUX",
          "txnStatCode": "S",
          "txnType": "P",
          "additionalDataDe48": "ABCDEF",
          "authorisedByGps": "N",
          "avsResult": null,
          "mtId": "1240",
          "recordDataDe120": null,
          "additionalDataDe124": "0.00"
        }
      },
      "card": {
        "cardId": 146,
        "provider": "MASTERCARD",
        "alias": "MasterCard-2995",
        "maskedPan": "537455******2995",
        "embossCardName": "DD DD",
        "embossBusinessName": "XXXX",
        "expiryDate": "2019-09-30T00:00:00.000Z"
      },
      "currency": {
        "code": "EUR",
        "description": "Euro"
      },
      "amountBeforeCharges": 1699,
      "feeAmount": 0,
      "taxAmount": 0,
      "amountAfterCharges": 1699,
      "balance": 3146,
      "myRef": "AMZAisence online",
      "date": "2017-09-21T10:32:53.773Z"
    }
  ]
}
```
List Card Transactions.


### HTTP REQUEST
`GET: https://api.fire.com/business/v1/me/cards/{cardid}/transactions?limit=25&offset=0`

### RETURNS
List of transactions for a card.

## Block a Card

```shell
# Send a block card request to the API
Curl https://api.fire.com/business/v1/me/cards/{cardId}/block
-X POST
-H “Authorization: Bearer $ACCESS_TOKEN”

# Status 204 No Content
```  

Updates status of an existing card to “Blocked” which prevents any transactions being carried out with that card.

### HTTP Request 

`POST https://api.fire.com/business/v1/me/cards/{cardId}/block`

### RETURNS
No body is returned - "Status 204 No Content" signifies the call was successful.

## Unblock a Card  

```shell
# Send a unblock card request to the API
Curl https://api.fire.com/business/v1/me/cards/{cardId}/unblock
-X POST
-H “Authorization: Bearer $ACCESS_TOKEN”

# Status 204 No Content
```  

Updates status of an existing card to “unblocked” which means that transactions can be carried out with that card.

### HTTP Request 

`POST https://api.fire.com/business/v1/me/cards/{cardId}/unblock`

### RETURNS
No body is returned - "Status 204 No Content" signifies the call was successful.





# Payee Bank Accounts

```shell
# JSON representation of a Payee Account
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
`id` | identifier for the Bank account _(assigned by fire.com)_ 
`accountName` | the name the user gives to the bank account to help them identify the account. 
`bic` | the BIC of the account if currency is `EUR`. 
`iban` | the IBAN of the account if currency is `EUR`. 
`nsc` | the Sort Code of the account if currency is `GBP`. 
`accountNumber` | the Account Number of the account if currency is `GBP`. 
`accountHolderName` | the name on the payee bank account. 
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`
`dateCreated` | the date/time the payee account was added. Milliseconds since the epoch (1970).
`status` | the status of the payee account. When the account is first added it is in `PENDING` state until the valided by clicking the link in the email sent to the authorized user's email address. `status.description` is the English text to describe the `status.type`.
`country` | the country of the account. `country.description` is the English version of the 2-letter code in `country.code`.

## List all Payee Bank Accounts

```shell
curl https://api.fire.com/business/v1/fundingsources \
  -X GET -G \
  -d "offset=0" \
  -d "limit=10" \
  -H "Authorization: Bearer $ACCESS_TOKEN"


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


Returns all your payee bank accounts. Ordered by date added descending. Can be paginated. 

### HTTP Request

`GET https://api.fire.com/business/v1/fundingsources`

### Returns

An array of payee Bank accounts (referenced as `fundingSources` for legacy reasons).



# Transactions
```shell
# Full details of an individual transaction.
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
	"amountBeforeCharges": 500,
	"feeAmount": 125,
	"taxAmount": 0,
	"amountAfterCharges": 625,
	"balance": 35,
	"date": "2017-02-11T19:56:40.977Z",
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

# Condensed transaction details when part of a list.
{
	"txnId": 30260,
	"refId": 26834,
	"ican": 1979,
	"type": "INTERNAL_TRANSFER_TO",
	"relatedParty": {
		"alias": "Main Account"
	},
	"currency": { 
		"code": "EUR",
		"description": "Euro"
	},
	"amountBeforeCharges": 5000,
	"feeAmount": 0,
	"taxAmount": 0,
	"amountAfterCharges": 5000,
	"balance": 8500,
	"myRef": "Transfer to main account",
	"date": "2015-04-29T22:56:48.867Z"
}
```
While there are many types of transactions, they are all represented by the same JSON object with a different `txnType`.

The transaction resource has the following attributes: 

Field | Description
--------- | -----------
`txnId` | The id of this side of the transaction (each transaction has two sides - a to and a from). This is used to get the details of the transaction.
`refId` | The id of the transaction.
`ican` | identifier for the fire.com account _(assigned by fire.com)_ _This field is only used in the condensed version._
`type` | The type of transaction. Use this to determine the "side" of the transaction - e.g. `INTERNAL_TRANSFER_FROM` would be a positive transaction from another account, `INTERNAL_TRANSFER_TO` is the negative side.
`relatedParty` | `relatedParty.alias` is the name of the account on the other side of the transaction. _This field is only used in the condensed version._
`currency` | a JSON entity with the currency code (`currency.code`) and English name (`currency.description`) of the currency for the account - either `EUR` or `GBP`.
`amountBeforeCharges` | Amount of the transaction before the fees and taxes were applied.
`feeAmount` | The amount of the fee, if any.
`taxAmount` | The amount of the tax, if any (e.g. Stamp duty for ATM transactions).
`amountAfterCharges` | Net amount lodged or taken from the account after fees and charges were applied.
`balance` | the balance of the account (in minor currency units - pence, cent etc. `434050` == `4,340.50 GBP` for a GBP account).
`myRef` | The comment/reference on the transaction 
`date` | Date of the transaction _(epoch date in full version, ISO date in condensed - will be fixed in a future release)_
`from` | The "from" side of the transaction. `from.type` is the type of the account, and `from.account` is the details of that account. _This field is only present in the full version._
`to` | The "to" side of the transaction. `to.type` is the type of the account, and `to.account` is the details of that account. _This field is only present in the full version._
`fxTradeDetails` | If this is a currency conversion, this will contain the FX rate and converted amount. _This field is only present in the full version._
`feeDetails` | The details of any fees applied. _This field is only present in the full version._

The transaction type can be one of the following:

Transaction Type | Description
--------- | -----------
`PAYMENT_RECEIVED`|
`LODGEMENT`|
`MANUAL_TRANSFER`|
`WITHDRAWAL`|
`REVERSAL`|
`INTERNAL_TRANSFER_TO`|
`INTERNAL_TRANSFER_FROM`|
`WITHDRAWAL_RETURNED`|
`LODGEMENT_REVERSED`|
`FX_INTERNAL_TRANSFER_FROM`|
`FX_INTERNAL_TRANSFER_TO`|
`CARD_POS_CONTACT_DEBIT`|
`CARD_POS_CONTACT_CREDIT`|
`CARD_POS_CONTACTLESS_DEBIT`|
`CARD_POS_CONTACTLESS_CREDIT`|
`CARD_ECOMMERCE_DEBIT`|
`CARD_ECOMMERCE_CREDIT`|
`CARD_ATM_DEBIT`|
`CARD_ATM_CREDIT`|
`CARD_INTERNATIONAL_POS_CONTACT_DEBIT`|
`CARD_INTERNATIONAL_POS_CONTACT_CREDIT`|
`CARD_INTERNATIONAL_POS_CONTACTLESS_DEBIT`|
`CARD_INTERNATIONAL_POS_CONTACTLESS_CREDIT`|
`CARD_INTERNATIONAL_ECOMMERCE_DEBIT`|
`CARD_INTERNATIONAL_ECOMMERCE_CREDIT`|
`CARD_INTERNATIONAL_ATM_DEBIT`|
`CARD_INTERNATIONAL_ATM_CREDIT`|
`CARD_POS_CONTACT_DEBIT_REVERSAL`|
`CARD_POS_CONTACT_CREDIT_REVERSAL`|
`CARD_POS_CONTACTLESS_DEBIT_REVERSAL`|
`CARD_POS_CONTACTLESS_CREDIT_REVERSAL`|
`CARD_ECOMMERCE_DEBIT_REVERSAL`|
`CARD_ECOMMERCE_CREDIT_REVERSAL`|
`CARD_ATM_DEBIT_REVERSAL`|
`CARD_ATM_CREDIT_REVERSAL`|
`CARD_INTERNATIONAL_POS_CONTACT_DEBIT_REVERSAL`|
`CARD_INTERNATIONAL_POS_CONTACT_CREDIT_REVERSAL`|
`CARD_INTERNATIONAL_POS_CONTACTLESS_DEBIT_REVERSAL`|
`CARD_INTERNATIONAL_POS_CONTACTLESS_CREDIT_REVERSAL`|
`CARD_INTERNATIONAL_ECOMMERCE_DEBIT_REVERSAL`|
`CARD_INTERNATIONAL_ECOMMERCE_CREDIT_REVERSAL`|
`CARD_INTERNATIONAL_ATM_DEBIT_REVERSAL`|
`CARD_INTERNATIONAL_ATM_CREDIT_REVERSAL`|


## List transactions for an account
```shell
curl https://api.fire.com/business/v1/accounts/1979/transactions \
  -X GET -G \
  -d "limit=25" \
  -d "offset=0" \
  -H "Authorization: Bearer $ACCESS_TOKEN"

{
	"total": 1,
	"dateRangeTo": 1430511042924,
	"transactions": [ 
		{
			"txnId": 30260,
			"refId": 26834,
			"ican": 1979,
			"type": "INTERNAL_TRANSFER_TO",
			"relatedParty": {
				"alias": "Main Account"
			},
			"currency": { 
				"code": "EUR",
				"description": "Euro"
			},
			"amountBeforeCharges": 5000,
			"feeAmount": 0,
			"taxAmount": 0,
			"amountAfterCharges": 5000,
			"balance": 8500,
			"myRef": "Transfer to main account",
			"date": "2015-04-29T22:56:48.867Z"
		}
	]
}
```

Retrieve a list of transactions against an account.

### HTTP Request

`GET  https://api.fire.com/business/v1/accounts/{accountId}/transactions`

### Returns

An array of transactions for `accountId` with a count (`total`)


## Filtered list of transactions for an account
```shell
curl https://api.fire.com/business/v1/accounts/1979/transactions/filter \
  -X GET -G \
  -d "dateRangeFrom=1493593200000" \
  -d "dateRangeTo=1496271600000" \
  -d "limit=25" \
  -d "offset=0" \
  -H "Authorization: Bearer $ACCESS_TOKEN"

{
	"total": 1,
	"dateRangeTo": 1496271600000,
	"transactions": [ 
		{
			"txnId": 30260,
			"refId": 26834,
			"ican": 1979,
			"type": "INTERNAL_TRANSFER_TO",
			"relatedParty": {
				"alias": "Main Account"
			},
			"currency": { 
				"code": "EUR",
				"description": "Euro"
			},
			"amountBeforeCharges": 5000,
			"feeAmount": 0,
			"taxAmount": 0,
			"amountAfterCharges": 5000,
			"balance": 8500,
			"myRef": "Transfer to main account",
			"date": "2018-04-29T22:56:48.867Z"
		}
	]
}
```

Retrieve a filtered list of transactions against an account.

### HTTP Request

`GET  https://api.fire.com/business/v1/accounts/{accountId}/transactions/filter`

The following query parameters can be used to filter the list:

Parameter | Description
--------- | -----------
`dateRangeFrom` | A millisecond epoch time specifying the date range start date. 
`dateRangeTo` | A millisecond epoch time specifying the date range end date. 
`searchKeyword` | Search term to filter by from the reference field (`myRef`). 
`transactionTypes` | One or more of the transaction types above. This field can be repeated multiple times to allow for multiple transaction types. 

### Returns

An array of transactions for `accountId` filtered by the specified fields with a count (`total`) filtered by date rage, transaction type or reference. 

# Direct Debits

The fire.com api allows businesses to automate direct debit payment actions on their fire.com business accounts.

You can retrieve details of your direct debit payments, direct debit mandates and also take actions on both your direct debit payments and mandates.

## Get the details of a Direct Debit
```shell
curl https://api.fire.com/business/v1/directdebits/42de0705-e3f1-44fa-8c41-79973eb80eb2 \
  -X GET -G \
  -d "limit=25" \
  -d "offset=0" \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# Response Body 
{
      "directDebitUuid": "42de0705-e3f1-44fa-8c41-79973eb80eb2",
      "currency": {
        "code": "GBP",
        "description": "Sterling"
      },
      "status": "RECEIVED | REJECT_REQUESTED | REJECT_READY_FOR_PROCESSING | REJECT_RECORD_IN_PROGRESS | REJECT_RECORDED | REJECT_FILE_CREATED |
 REJECT_FILE_SENT | COLLECTED | REFUND_REQUESTED | REFUND_RECORD_IN_PROGRESS | REFUND_RECORDED | REFUND_FILE_CREATED | REFUND_FILE_SENT",
      "type": "FIRST_COLLECTION|ONGOING_COLLECTION|REPRESENTED_COLLECTION|FINAL_COLLECTION",
      "mandateUuid": "f171b143-e3eb-47de-85a6-1c1f1108c701",
      "originatorReference": "VODA-123456",
      "originatorName": "Vodafone PLC",
      "directDebitReference": "VODA-ABC453-1",
      "targetIcan ": 1,
      "targetPayeeId": 12,
      "isDDIC": false,
      "amount": 100,
      "schemeRejectReason": "<e.g. 'Instruction cancelled by payer' >",
      "schemeRejectReasonCode": "For BACS (ARUDD): 0|1|2|3|5|6|7|8|9|A|B",
      "lastUpdated": "2016-12-15T22:56:05.937Z",
      "dateCreated": "2016-12-15T22:56:05.937Z"
}

```
Retrieve all details of a single direct debit collection/payment, whether successful or not.

The permision needed to access this endpoint is PERM_BUSINESS_GET_DIRECT_DEBIT



### HTTP Request

`GET https://api.fire.com/business/v1/directdebits/{directDebitUuid}`

Parameter | Description
--------- | -----------
`directDebitUuid` | The UUID for the direct debit payment


## Get all Direct Debit payments associated with a direct debit mandate
```shell
curl https://api.fire.com/business/v1/directdebits \
  -X GET -G \
  -d "limit=25" \
  -d "offset=0" \
  -d "directDebitUuid=42de0705-e3f1-44fa-8c41-79973eb80eb2" \
  -H "Authorization: Bearer $ACCESS_TOKEN"



# Response Body 
{
  "total": 1,
  "directdebits": [
    {
      "directDebitUuid": "42de0705-e3f1-44fa-8c41-79973eb80eb2",
      "currency": {
        "code": "GBP",
        "description": "Sterling"
      },
      "status": "RECEIVED | REJECT_REQUESTED | REJECT_READY_FOR_PROCESSING | REJECT_RECORD_IN_PROGRESS | REJECT_RECORDED | REJECT_FILE_CREATED |
 REJECT_FILE_SENT | COLLECTED | REFUND_REQUESTED | REFUND_RECORD_IN_PROGRESS | REFUND_RECORDED | REFUND_FILE_CREATED | REFUND_FILE_SENT",
      "type": "FIRST_COLLECTION|ONGOING_COLLECTION|REPRESENTED_COLLECTION|FINAL_COLLECTION",
      "mandateUuid": "f171b143-e3eb-47de-85a6-1c1f1108c701",
      "originatorReference": "VODA-123456",
      "originatorName": "Vodafone PLC",
      "originatorAlias": "Vodafone PLC Alias",
      "originatorLogoUrlSmall": "www.originatorLogoSmall.com",
      "originatorLogoUrlLarge": "www.originatorLogoLarge.com",
      "directDebitReference": "VODA-ABC453-1",
      "targetIcan ": 1,
      "targetPayeeId": 12,
      "amount": 100,
      "feeAmount": 100,
      "taxAmount": 100,
      "amountAfterCharges": 100,
      "fireRejectReason": "ACCOUNT_NOT_FOUND | ACCOUNT_NOT_LIVE | ACCOUNT_DOES_NOT_ACCEPT_DIRECT_DEBITS | INVALID_ACCOUNT_CURRENCY | MANDATE_NOT_FOUND | MANDATE_INVALID_STATUS |
       BUSINESS_NOT_LIVE | BUSINESS_NOT_FULL | PERSONAL_USER_NOT_LIVE | PERSONAL_USER_NOT_FULL | INSUFFICIENT_FUNDS | REQUESTED_BY_CUSTOMER_VIA_SUPPORT | MANDATE_CANCELLED | CUSTOMER_DECEASED | ACCOUNT_TRANSFERRED |
       ADVANCE_NOTICE_DISPUTED_REQUESTED_BY_CUSTOMER | AMOUNT_DIFFERS_REQUESTED_BY_CUSTOMER | AMOUNT_NOT_DUE_REQUESTED_BY_CUSTOMER | PRESENTATION_OVERDUE_REQUESTED_BY_CUSTOMER | ORIGINATOR_DIFFERS | CUSTOMER_ACCOUNT_CLOSED | REQUESTED_BY_CUSTOMER |",
      "schemeRejectReason": "<e.g. 'Instruction cancelled by payer' >",
      "schemeRejectReasonCode": "For BACS (ARUDD): 0|1|2|3|5|6|7|8|9|A|B",
      "fireRefundReason": "ADVANCE_NOTICE_DIFFERS_FROM_DIRECT_DEBIT | CUSTOMER_DID_NOT_RECEIVE_ADVANCE_NOTICE | MANDATE_CANCELLED_BY_FIRE | MANDATE_CANCELLED_EXTERNALLY_BY_CUSTOMER | MANDATE_NOT_APPROVED_BY_CUSTOMER | MANDATE_SIGNATURE_IS_NOT_VALID |
       REQUESTED_BY_ORIGINATOR | CUSTOMER_DOES_NOT_RECOGNISE_ORIGINATOR |",
      "schemeRefundReason": "<e.g. 'DDI cancelled by Paying Bank' >",
      "schemeRefundReasonCode": "For BACS (DDIC): 0|1|2|3|5|6|7|8",
      "dateRefunded": "2016-12-15T22:56:05.937Z",
      "lastUpdated": "2016-12-15T22:56:05.937Z",
      "dateCreated": "2016-12-15T22:56:05.937Z"
    }
  ]
}
```
Retrieve all direct debit payments associated with a direct debit mandate.

The permision needed to access this endpoint is PERM_BUSINESS_GET_DIRECT_DEBITS

### HTTP Request

`GET https://api.fire.com/business/v1/directdebits`

Parameter | Description
--------- | -----------
`ican` | Identifier for the fire.com account (assigned by fire.com)
`payeeId` | The ID of the existing or automatically created payee 
`mandateUuid` | The UUID for the mandate
`currency` | The currency of the direct debit mandate
`status` | The statuses of the direct debit payments associated with the mandate (RECEIVED, COLLECTED, REJECT_REQUESTED, REJECT_PAID)
`limit` | The number of records to return. Defaults to `10` - max is `200`.
`offset` | The page offset. Defaults to `0`. This is the record number that the returned list will start at. E.g. `offset` = `40` and `limit` = `20` will return records 40 to 59.

## Get Direct Debit mandate details
```shell
# Response Body 
{

"mandateUuid": "28d627c3-1889-44c8-ae59-6f6b20239260",
  "currency": {
    "code": "GBP",
    "description": "Sterling"
  },
  "status": "CREATED | LIVE | REJECT_REQUESTED | REJECT_RECORD_IN_PROGRESS | REJECT_RECORDED | REJECT_FILE_CREATED | REJECT_FILE_SENT | CANCEL_REQUESTED |
 CANCEL_RECORD_IN_PROGRESS | CANCEL_RECORDED | CANCEL_FILE_CREATED | CANCEL_FILE_SENT | COMPLETE | DORMANT",
  "originatorReference": "VODA-123456",
  "originatorName": "Vodafone PLC",
  "originatorAlias": "Vodafone PLC Alias",
  "originatorLogoUrlSmall": "www.originatorLogoSmall.com",
  "originatorLogoUrlLarge": "www.originatorLogoLarge.com",
  "mandateReference": "VODA-ABC453",
  "alias": "Vodafone",
  "targetIcan": 1,
  "numberOfDirectDebitCollected": 1,
  "valueOfDirectDebitCollected": 2,
  "latestDirectDebitAmount": 3,
  "latestDirectDebitDate": 4,
  "fireRejectReason": "ACCOUNT_DOES_NOT_ACCEPT_DIRECT_DEBITS | DDIC | ACCOUNT_NOT_FOUND | ACCOUNT_NOT_LIVE | CUSTOMER_NOT_FOUND | BUSINESS_NOT_LIVE | BUSINESS_NOT_FULL | 
   PERSONAL_USER_NOT_LIVE | PERSONAL_USER_NOT_FULL | MANDATE_ALREADY_EXISTS | MANDATE_WITH_DIFERENT_ACCOUNT | NULL_MANDATE_REFERENCE | INVALID_ACCOUNT_CURRENCY | INVALID_MANDATE_REFERENCE | REQUESTED_BY_CUSTOMER_VIA_SUPPORT | 
   CUSTOMER_ACCOUNT_CLOSED | CUSTOMER_DECEASED | ACCOUNT_TRANSFERRED | MANDATE_NOT_FOUND | ACCOUNT_TRANSFERRED_TO_DIFFERENT_ACCOUNT | INVALID_ACCOUNT_TYPE | MANDATE_EXPIRED | MANDATE_CANCELLED | REQUESTED_BY_CUSTOMER |",
  "schemeRejectReason": "<e.g. 'Instruction cancelled by payer' >",
  "schemeRejectReasonCode": "For BACS (AUDDIS): 1|2|3|5|6|B|C|F|G|H|O|K",
  "fireCancelReason": "REFRER_TO_CUSTOMER | REQUESTED_BY_CUSTOMER_VIA_SUPPORT | CUSTOMER_DECEASED | CUSTOMER_ACCOUNT_CLOSED | ADVANCE_NOTICE_DISPUTED_VIA_SUPPORT | ACCOUNT_TRANSFERRED | ACCOUNT_TRANSFERRED_TO_DIFFERENT_ACCOUNT | 
   MANDATE_AMENDED | MANDATE_REINSTATED | REQUESTED_BY_CUSTOMER",
  "schemeCancelReason": "<e.g. Instruction cancelled by payer >",
  "schemeCancelReasonCode": "For BACS (ADDACS): 0|1|2|3|B|C|D|E|R",
  "lastUpdated": "2016-12-15T22:56:05.937Z",
  "dateCreated": "2016-12-15T22:56:05.937Z",
  "dateRejected": "2016-12-15T22:56:05.937Z",
  "dateCompleted": "2016-12-15T22:56:05.937Z",
  "dateCancelled": "2016-12-15T22:56:05.937Z"
}
```
Retrieve all details for a direct debit mandate.

The permision needed to access this endpoint is PERM_BUSINESS_GET_MANDATE



### HTTP Request
`GET https://api.fire.com/business/v1/mandates/{mandateUuid}`

Parameter | Description
--------- | -----------
`mandateUuid` | The UUID for the mandate

## List all Direct Debit Mandates
```shell
# Response Body 
{
  "total": 1,
  "mandates": [
    {
      "mandateUuid": "28d627c3-1889-44c8-ae59-6f6b20239260",
      "currency": {
        "code": "GBP",
        "description": "Sterling"
      },
      "status": "CREATED | LIVE | REJECT_REQUESTED | REJECT_RECORD_IN_PROGRESS | REJECT_RECORDED | REJECT_FILE_CREATED | REJECT_FILE_SENT | CANCEL_REQUESTED | 
CANCEL_RECORD_IN_PROGRESS | CANCEL_RECORDED | CANCEL_FILE_CREATED | CANCEL_FILE_SENT | COMPLETE | DORMANT",
      "originatorReference": "VODA-123456",
      "originatorName": "Vodafone PLC",
      "mandateReference": "VODA-ABC453",
      "alias": "Vodafone",
      "targetIcan": 1,
      "numberOfDirectDebitsCollected": 1,
      "valueOfDirectDebitsCollected": 2,
      "latestDirectDebitAmount": 3,
      "latestDirectDebitDate": "2016-12-15T22:56:05.937Z",

      "schemeRejectReason": "<e.g. 'Instruction cancelled by payer' >",
      "schemeRejectReasonCode": "For BACS (AUDDIS): 1|2|3|5|6|B|C|F|G|H|O|K",

      "schemeCancelReason": "<e.g. Instruction cancelled by payer >",
      "schemeCancelReasonCode": "For BACS (ADDACS): 0|1|2|3|B|C|D|E|R",

      "lastUpdated": "2016-12-15T22:56:05.937Z",
      "dateCreated": "2016-12-15T22:56:05.937Z",
      "dateCompleted": "2016-12-15T22:56:05.937Z",
      "dateCancelled": "2016-12-15T22:56:05.937Z"
    }
  ]
}
```
List all direct debit mandates.

The permision needed to access this endpoint is	PERM_BUSINESS_GET_MANDATES

### HTTP Request
`GET https://api.fire.com/business/v1/mandates`

Parameter | Description
--------- | -----------
`ican` | Identifier for the fire.com account (assigned by fire.com)
`payeeId` | The ID of the existing or automatically created payee 
`currency` | The currency of the direct debit mandate
`status` | The statuses of the direct debit payments associated with the mandate (RECEIVED, COLLECTED, REJECT_REQUESTED, REJECT_PAID)
`limit` | The number of records to return. Defaults to `10` - max is `200`.
`offset` | The page offset. Defaults to `0`. This is the record number that the returned list will start at. E.g. `offset` = `40` and `limit` = `20` will return records 40 to 59.


## Reject a Direct Debit
```shell
# Response Body 
204 no content
```
permission name	PERM_BUSINESS_POST_DIRECT_DEBIT_REJECT

This endpoint allows you to reject a direct debit payment where the status is still set to RECEIVED.

### HTTP Request
`POST https://api.fire.com/business/v1/directdebits/{directDebitUuid}/reject`

Parameter | Description
--------- | -----------
`directDebitUuid` | The UUID for the direct debit payment


## Cancel a Direct Debit Mandate
```shell
# Response Body 
204 no content
```

The permision needed to access this endpoint is	PERM_BUSINESS_POST_MANDATE_CANCEL

This endpoint allows you to cancel a direct debit mandate.

### HTTP Request
`POST https://api.fire.com/business/v1/mandates/{mandateUuid}/cancel`


Parameter | Description
--------- | -----------
`mandateUuid` | The UUID for the mandate


## Update Direct Debit Mandate Alias

```shell
# Response Body 
204 no content
```
The permision needed to access this endpoint is	PERM_BUSINESS_PUT_MANDATE

### HTTP Request
`POST https://api.fire.com/business/v1/mandates/{mandateUuid}`

Parameter | Description
--------- | -----------
`mandateUuid` | The UUID for the mandate
`alias` | The name for the mandate


## Activate a Direct Debit Mandate

```shell
# Response Body 
204 no content
```
The permision needed to access this endpoint is	PERM_BUSINESS_POST_MANDATE_ACTIVATE

This endpoint can only be used to activate a direct debit mandate when it is in the status REJECT_REQUESTED (even if the account has direct debits disabled). This action will also enable the account for direct debits if it was previously set to be disabled. 

### HTTP Request
`POST https://api.fire.com/business/v1/mandates/{mandateUuid}/activate`

Parameter | Description
--------- | -----------
`mandateUuid` | The UUID for the mandate







# Payment Initiation

The fire.com API allows businesses to automate payments between their accounts or to third parties across the UK and Europe.

For added security, the API can only set up the payments in batches. These batches must be approved
by an authorised user via the firework mobile app. 

The process is as follows:

1. Create a new batch
2. Add payments to the batch
3. Submit the batch for approval. 


Once the batch is submitted, the authorised users will receive notifications to their firework mobile apps. 
They can review the contents of the batch and then approve or reject it. 
If approved, the batch is then processed. 
You can avail of enhanced security by using Dual Authorisation to verify payments if you wish.
Dual Authorisation can be enabled by you when setting up your API application in firework online.


### Batch Object
```shell
# JSON representation of a batch
{
  "batchUuid": "F2AF3F2B-4406-4199-B249-B354F2CC6019",
  "type": "BANK_TRANSFER",
  "status":"COMPLETE",
  "sourceName": "Payment API",
  "batchName": "January 2018 Payroll",
  "jobNumber": "2018-01-PR",
  "callbackUrl": "https://my.webserver.com/cb/payroll"
  "currency":"EUR", 
  "numberOfItemsSubmitted":1, 
  "valueOfItemsSubmitted":1000, 
  "numberOfItemsFailed":0,
  "valueOfItemsFailed":0,
  "numberOfItemsSucceeded":1,
  "valueOfItemsSucceeded":1000,
  "lastUpdated":"2018-04-04T10:48:53.540Z",
  "dateCreated":"2018-04-04T00:53:21.910Z"
}
```

A Batch Object consists of the following data:

Field | Description
----- | -----------
`batchUuid` | A UUID for this batch.
`type` | The type of the batch - can be `INTERNAL_TRANSFER`, `BANK_TRANSFER`, `NEW_PAYEE`.
`status` | The status of the batch - can be `PENDING_APPROVAL`, `REJECTED`, `COMPLETE`, `OPEN`, `CANCELLED`, `PENDING_PARENT_BATCH_APPROVAL`, `READY_FOR_PROCESSING`, `PROCESSING`.
`sourceName` | A string describing where the batch originated - for instance the name of the API token that was used, or showing that the batch was automatically created by fire.com (in the case of a new payee batch).  
`batchName` | An optional name you give to the batch at creation time. Say `January 2018 Payroll`.
`jobNumber` | An optional job number you can give to the batch to help link it to your own system. 
`callbackUrl` | An optional POST URL that all events for this batch will be sent to. 
`currency` | All payments in the batch must be the same currency - either `EUR` or `GBP`
`numberOfItemsSubmitted` | A count of the number of items in the batch
`valueOfItemsSubmitted` | A sum of the value of items in the batch. Specified in pence or cent.
`numberOfItemsFailed` | Once processed, a count of the number of items that didn't process successfully. 
`valueOfItemsFailed` | Once processed, a sum of the value of items that didn't process successfully. Specified in pence or cent. 
`numberOfItemsSucceeded` | Once processed, a count of the number of items that processed successfully. 
`valueOfItemsSucceeded` | Once processed, a sum of the value of items that processed successfully. Specified in pence or cent.
`lastUpdated` | The datestamp of the last action on this batch - ISO format: e.g. 2018-04-04T10:48:53.540Z
`dateCreated` | The datestamp the batch was created - ISO format: e.g. 2018-04-04T00:53:21.910Z

### Internal Transfer Object
```shell
# Pending
{
  "batchItemUuid": "FBA4A76A-CE51-4FC1-B562-98EC01299E4D",
  "status": "SUBMITTED",
  "dateCreated": "2018-04-04T01:20:38.647Z",
  "lastUpdated": "2018-04-04T01:20:38.647Z",
  "icanFrom": 5532,
  "icanTo": 2150,
  "amount": 1000,
  "ref": "Testing a transfer via batch"
}

# After processing
{
  "batchItemUuid":"FBA4A76A-CE51-4FC1-B562-98EC01299E4D",
  "status":"SUCCEEDED",
  "dateCreated":"2018-04-04T01:20:38.647Z",
  "lastUpdated":"2018-04-04T10:48:53.417Z",
  "icanFrom":5532,
  "icanTo":2150,
  "amount":1000,
  "ref":"Testing a transfer via batch",
  "result": {
    "code":50001,
    "message":"SUCCESS"
  },
  "feeAmount":0,
  "taxAmount":0,
  "amountAfterCharges":1000,
  "refId":523211
}
```

The Internal Transfer Object contains the following data:

Field | Description
----- | -----------
`batchItemUuid` | A UUID for this item.
`status` | The status of the batch - can be `SUBMITTED`, `REMOVED`, `SUCCEEDED`, `FAILED`.
`icanFrom` | The account ID for the fire.com account the funds are taken from. 
`icanTo` | The account ID for the fire.com account the funds are sent to.
`amount` | The amount of the transfer in pence or cent. 
`ref` | The reference on the transaction.
`result` | The outcome of the attempted transaction. 
`feeAmount` | The fee charged by fire.com for the payment. In pence or cent. 
`taxAmount` | Any taxes/duty collected by fire.com for this payments (e.g. stamp duty etc). In pence or cent.
`amountAfterCharges` | The amount of the transfer after fees and taxes. in pence or cent.
`refId` | The ID of the resulting payment in your account. Can be used to retrieve the transaction using the `https://api.fire.com/business/v1/accounts/{accountId}/transactions/{refId}` endpoint. 
`lastUpdated` | The datestamp of the last action on this item - ISO format: e.g. 2018-04-04T10:48:53.540Z
`dateCreated` | The datestamp the item was created - ISO format: e.g. 2018-04-04T00:53:21.910Z

### Bank Transfer Object
```shell
# Pending
{
  "batchItemUuid": "8A868124-61AE-4680-ACF0-19DA117AEC14",
  "status": "SUBMITTED",
  "dateCreated": "2018-04-04T14:03:48.260Z",
  "lastUpdated": "2018-04-04T14:03:48.260Z",
  "icanFrom": 2150,
  "amount": 500,
  "myRef": "Payment to John Smith for Consultancy in Dec.",
  "yourRef": "ACME LTD - INV 2342",
  "payeeType": "ACCOUNT_DETAILS",
  "destIban": "IE00AIBK93123412345123",
  "destAccountHolderName": "John Smith"
}

# After Processing
{
  "batchItemUuid":"8A868124-61AE-4680-ACF0-19DA117AEC14",
  "status":"SUCCEEDED",
  "dateCreated":"2018-04-04T14:03:48.260Z",
  "lastUpdated":"2018-04-04T14:13:33.587Z",
  "icanFrom":2150,
  "amount":500,
  "myRef":"Payment to John Smith for Consultancy in Dec.",
  "yourRef":"ACME LTD - INV 2342",
  "payeeType":"ACCOUNT_DETAILS",
  "destIban":"IE00AIBK93123412345123",
  "destAccountHolderName":"John Smith",
  "result":{
    "code":50001,
    "message":"SUCCESS"
  },
  "feeAmount":49,
  "taxAmount":0,
  "amountAfterCharges":549,
  "refId":523576,
  "payeeId":1304
}
```

The Bank Transfer Object contains the following data:

Field | Description
----- | -----------
`batchItemUuid` | A UUID for this item.
`status` | The status of the batch - can be `SUBMITTED`, `REMOVED`, `SUCCEEDED`, `FAILED`.
`icanFrom` | The account ID for the fire.com account the funds are taken from. 
`amount` | The amount of the transfer in pence or cent. 
`myRef` | The reference on the transaction for your records - not shown to the beneficiary.
`yourRef` | The reference on the transaction - displayed on the beneficiary bank statement.
`payeeType` | The type of payee that was specified - either `PAYEE_ID` or `ACCOUNT_DETAILS`
`payeeId` | The ID of the existing or automatically created payee
`destAccountHolderName` | The name of the payee.
`destIban` | The destination IBAN if a Euro Bank transfer.
`destNsc` | The destination Sort Code if a GBP Bank transfer.
`destAccountNumber` |The destination Account Number if a GBP Bank transfer.
`result` | The outcome of the attempted transaction. 
`feeAmount` | The fee charged by fire.com for the payment. In pence or cent. 
`taxAmount` | Any taxes/duty collected by fire.com for this payments (e.g. stamp duty etc). In pence or cent.
`amountAfterCharges` | The amount of the transfer after fees and taxes. in pence or cent.
`refId` | The ID of the resulting payment in your account. Can be used to retrieve the transaction using the `https://api.fire.com/business/v1/accounts/{accountId}/transactions/{refId}` endpoint. 
`lastUpdated` | The datestamp of the last action on this item - ISO format: e.g. 2018-04-04T10:48:53.540Z
`dateCreated` | The datestamp the item was created - ISO format: e.g. 2018-04-04T00:53:21.910Z


## Batch Life Cycle Events
A batch webhook can be specified to receive details of all the payments as they are processed. 
This webhook receives notifications for every event in the batch lifecycle. 

The following events are triggered during a batch:

Event | Description
----- | -----------
`batch.opened` | Contains the details of the batch opened. Checks that the callback URL exists - unless a HTTP 200 response is returned, the callback URL will not be configured. 
`batch.item-added` | Details of the item added to the batch
`batch.item-removed` | Details of the item removed from the batch
`batch.cancelled` | Notifies that the batch was cancelled. 
`batch.submitted` | Notifes that the batch was submitted
`batch.approved` | Notifies that the batch was approved.
`batch.rejected` | Notifies that the batch was rejected.
`batch.failed` | Notifies that the batch failed - includes the details of the failure (insufficient funds etc)
`batch.completed` | Notifies that the batch completed successfully. Includes a summary. 

Push notifications are sent to the firework mobile app for many of these events too - these can be
configured from within the app. 




## Create a new Batch

```shell
# Create the JSON object for the new Batch
# cat create-batch-request.json
{
  "type": "BANK_TRANSFER",
  "currency": "EUR", 
  "batchName": "January 2018 Payroll",
  "jobNumber": "2018-01-PR",
  "callbackUrl": "https://my.webserver.com/cb/payroll"
}

# Post that to the API
curl https://api.fire.com/business/v1/batches \
  -X POST \
  -d @create-batch-request.json \
  -H "Content-type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN"


{
  "batchUuid":"F2AF3F2B-4406-4199-B249-B354F2CC6019"
}
```

Create a new batch of payments. 

### HTTP Request

`POST  https://api.fire.com/business/v1/batches`

Parameter | Description
--------- | -----------
`type` | This is the type of payments that will be included in the batch - either `INTERNAL_TRANSFER` or `BANK_TRANSFER`. Only one type of transfer can be included in each batch. 
`currency` | The currency of payments in the batch. Either `EUR` or `GBP`. Only one currency can be included in a batch.
`batchName` | An optional name for this batch. Useful in reporting. 
`jobNumber` | An optional job number for the batch. Useful in reporting.
`batchUuid` | Optionally set the UUID for this batch. Leave blank to let fire.com create it for you. Must be a UUID.
`callbackUrl` | An optional callback URL for batch events. 
 
### Returns

The UUID of the newly created batch. This is used to reference the batch in further API calls. 







## Add a Payment to the Batch

```shell
# Create the JSON object for the new Batch
# cat add-internal-transfer-to-batch-request.json
{
  "icanFrom": "2001",
  "icanTo": "2041", 
  "amount": "10000",
  "ref": "Moving funds to Operating Account"
}

# cat add-mode-1-bank-transfer-to-batch-request.json
{
  "icanFrom": "2001",
  "payeeId": "15002", 
  "payeeType": "PAYEE_ID", 
  "amount": "500",
  "myRef": "Payment to John Smith for Consultancy in Dec.",
  "yourRef": "ACME LTD - INV 23434"
}

# cat add-mode-2-euro-bank-transfer-to-batch-request.json
{
  "icanFrom": "2001",
  "payeeType": "ACCOUNT_DETAILS", 
  "destIban": "IE00AIBK93123412341234", 
  "destAccountHolderName" :"John Smith",
  "amount": "500"
  "myRef": "Payment to John Smith for Consultancy in Dec.",
  "yourRef": "ACME LTD - INV 23434"
}

# cat add-mode-2-gbp-bank-transfer-to-batch-request.json
{
  "icanFrom": "2001",
  "payeeType": "ACCOUNT_DETAILS", 
  "destNsc": "123456", 
  "destAccountNumber": "12345678",
  "destAccountHolderName" :"John Smith",
  "amount": "500"
  "myRef": "Payment to John Smith for Consultancy in Dec.",
  "yourRef": "ACME LTD - INV 23434"
}

# Post that to the API (Internal transfers)
curl https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers \
  -X POST \
  -d @add-internal-transfer-to-batch-request.json \
  -H "Content-type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# Post that to the API (Bank transfers)
curl https://api.fire.com/business/v1/batches/{batchUuid}/banktransfers \
  -X POST \
  -d @add-mode-2-eur-bank-transfer-to-batch-request.json \
  -H "Content-type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
  
{
  "batchItemUuid":"fba4a76a-ce51-4fc1-b562-98ec01299e4d"
}
```

This process is slightly different depending on whether it is an `INTERNAL_TRANSFER` or `BANK_TRANSFER` batch type.

### Internal Transfers
Simply specify the source account, destination account, amount and a reference.

### Bank Transfers
There are two ways to process bank transfers - by Payee ID *(Mode 1)* or by Payee Account Details *(Mode 2)*.

Mode | Description
---- | -----------
Mode 1 | Use the payee IDs of existing approved payees set up against your account. These batches can be approved in the normal manner.
Mode 2 | Use the account details of the payee. In the event that these details correspond to an existing approved payee, the batch can be approved as normal. If the account details are new, a batch of New Payees will automatically be created. This batch will need to be approved before the Payment batch can be approved. These payees will then exist as approved payees for future batches.


### HTTP Request

`POST  https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers`

`POST  https://api.fire.com/business/v1/batches/{batchUuid}/banktransfers`

The POST data for an Internal Transfer (between your own fire.com accounts) is:

Parameter | Description
--------- | -----------
`icanFrom` | The Account ID of the source account. 
`icanTo` | The Account ID of the destination account.
`amount` | The amount of the transfer in pence or cent. 
`ref` | The reference to put on the transfer.
 
The POST data for a Bank Transfer (to an external payee account) is:

Parameter | Description
--------- | -----------
`icanFrom` | The Account ID of the source account. 
`payeeType` | Either `PAYEE_ID` or `ACCOUNT_DETAILS`. Use `PAYEE_ID` if you are paying existing approved payees *(Mode 1)*. Specify the payee ID in the `payeeId` field. Use `ACCOUNT_DETAILS` if you are providing account numbers/sort codes/IBANs *(Mode 2)*. Specify the account details in the `destIban`, `destAccountHolderName`, `destNsc` or `destAccountNumber` fields as appropriate.  
`payeeId` | _(Conditional)_ Provide this field if using *Mode 1* and `payeeType` = `PAYEE_ID`. Use the ID of the payee.
`destAccountHolderName` | _(Conditional)_ Provide this field if using *Mode 2*.
`destIban` | _(Conditional)_ Provide this field if using *Mode 2* and the payee account is in EURO.
`destNsc` | _(Conditional)_ Provide this field if using *Mode 2* and the payee account is in GBP.
`destAccountNumber` | _(Conditional)_ Provide this field if using *Mode 2* and the payee account is in GBP.
`amount` | The amount of the transfer in pence or cent. 
`myRef` | The reference to put on the transfer - only you see this reference.
`yourRef` | The reference to put on the transfer for the payee to see on their bank statement.

### Returns

The UUID of the newly created batch item. This can be used to remove the item from the batch as shown below.



## Remove a Payment from the Batch

```shell
# Send a DELETE request to the API
curl https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers/{itemUuid} \
  -X DELETE \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# Returns a HTTP 200 OK.
```

Removes a Payment from the Batch. You can only remove payments before the batch is submitted for approval (while it is in the `OPEN` state.)

### HTTP Request

`DELETE https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers/{itemUuid}`

`DELETE https://api.fire.com/business/v1/batches/{batchUuid}/banktransfers/{itemUuid}`


### Returns

No body is returned - a HTTP 200 OK signifies the call was successful. 





## Cancel a Batch

```shell
# Send a DELETE request to the API
curl https://api.fire.com/business/v1/batches/{batchUuid} \
  -X DELETE \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# Returns a HTTP 200 OK.
```

Cancels the Batch. You can only cancel a batch before it is submitted for approval (while it is in the `OPEN` state.)

### HTTP Request

`DELETE https://api.fire.com/business/v1/batches/{batchUuid}`


### Returns

No body is returned - a HTTP 200 OK signifies the call was successful. 



## Submit a Batch for Approval

```shell
# Send a PUT request to the API
curl https://api.fire.com/business/v1/batches/{batchUuid} \
  -X PUT \
  -H "Authorization: Bearer $ACCESS_TOKEN"

# Returns a HTTP 204 No Content.
```

Submits the Batch (for approval in the case of a `BANK_TRANSFER`). If this is an `INTERNAL_TRANSFER` batch, the transfers are immediately queued for processing. If this is a `BANK_TRANSFER` batch, this will trigger requests for approval to the firework mobile apps of authorised users. Once those users approve the batch, it is queued for processing.

You can only submit a batch while it is in the `OPEN` state.

### HTTP Request

`PUT https://api.fire.com/business/v1/batches`


### Returns

No body is returned - a HTTP 204 No Content response signifies the call was successful. 





## List Batches

```shell
curl https://api.fire.com/business/v1/batches \
  -X GET -G \
  -d "batchStatuses=COMPLETE" \   
  -d "batchTypes=INTERNAL_TRANSFER" \   
  -d "orderBy=DATE" \
  -d "order=DESC" \
  -d "offset=0" \
  -d "limit=10" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
  
{
  "total": 1,
  "batchRequests": [
    {
	   "batchUuid": "F2AF3F2B-4406-4199-B249-B354F2CC6019",
	   "type": "INTERNAL_TRANSFER",
	   "status":"COMPLETE",
	   "sourceName": "Payment API",
	   "batchName": "January 2018 Payroll",
	   "jobNumber": "2018-01-PR",
	   "callbackUrl": "https://my.webserver.com/cb/payroll"
	   "currency":"EUR", 
	   "numberOfItemsSubmitted":1, 
	   "valueOfItemsSubmitted":1000, 
	   "numberOfItemsFailed":0,
	   "valueOfItemsFailed":0,
	   "numberOfItemsSucceeded":1,
	   "valueOfItemsSucceeded":1000,
	   "lastUpdated":"2018-04-04T10:48:53.540Z",
	   "dateCreated":"2018-04-04T00:53:21.910Z"
	 }
  ]
}
```

Returns the list of batch with the specified types and statuses.   

### HTTP Request

`GET https://api.fire.com/business/v1/batches`


### Returns

A fire.com List of Batch objects. 



## Get details of a single Batch

```shell
curl https://api.fire.com/business/v1/batches/{batchUuid} \
  -X GET \
  -H "Authorization: Bearer $ACCESS_TOKEN"
  
{
  "batchUuid": "F2AF3F2B-4406-4199-B249-B354F2CC6019",
  "type": "BANK_TRANSFER",
  "status":"COMPLETE",
  "sourceName": "Payment API",
  "batchName": "January 2018 Payroll",
  "jobNumber": "2018-01-PR",
  "callbackUrl": "https://my.webserver.com/cb/payroll"
  "currency":"EUR", 
  "numberOfItemsSubmitted":1, 
  "valueOfItemsSubmitted":1000, 
  "numberOfItemsFailed":0,
  "valueOfItemsFailed":0,
  "numberOfItemsSucceeded":1,
  "valueOfItemsSucceeded":1000,
  "lastUpdated":"2018-04-04T10:48:53.540Z",
  "dateCreated":"2018-04-04T00:53:21.910Z"
}
```

Returns the details of the batch specified in the API endpoint - `{batchUuid}`.   

### HTTP Request

`GET https://api.fire.com/business/v1/batches/{batchUuid}`


### Returns

A Batch object containing the details of the batch. 




## List Items in a Batch

```shell
curl https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers \
  -X GET -G \
  -d "offset=0" \
  -d "limit=10" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
  
{
  "total":1, 
  "items": [
    {
      "batchItemUuid":"FBA4A76A-CE51-4FC1-B562-98EC01299E4D",
      "status":"SUCCEEDED",
      "result": {
        "code":50001,
        "message":"SUCCESS"
      },
      "dateCreated":"2018-04-04T01:20:38.647Z",
      "lastUpdated":"2018-04-04T10:48:53.417Z",
      "feeAmount":0,
      "taxAmount":0,
      "amountAfterCharges":1000,
      "icanFrom":5532,
      "icanTo":2150,
      "amount":1000,
      "ref":"Testing a transfer via batch",
      "refId":523211
    }
  ]
}
```

Returns a paginated list of items in the specified batch.  

### HTTP Request

`GET https://api.fire.com/business/v1/batches/{batchUuid}/internaltransfers`

`GET https://api.fire.com/business/v1/batches/{batchUuid}/banktransfers`


### Returns

A fire.com list object of Batch Items (Internal transfers or Bank transfers). 


## List Approvers for a Batch

```shell
curl https://api.fire.com/business/v1/batches/{batchUuid}/approvals \
  -X GET -G \
  -H "Authorization: Bearer $ACCESS_TOKEN"
  
{
  "approvals": [
    {
      "userId":3138,
      "emailAddress":"jane.doe@acme.com",
      "firstName":"Jane",
      "lastName":"Doe",
      "mobileNumber":"+353871234567",
      "status":"PENDING",
      "lastUpdated":"2018-04-04T14:07:22.193Z"
    }
  ]
}
```

Returns a list of approvers for this batch.  

### HTTP Request

`GET https://api.fire.com/business/v1/batches/{batchUuid}/approvals`


### Returns

A list of approvers for this batch.  









# Payment Requests
Payment Requests allow you to collect payment in real-time through Twitter, Facebook, text, email and print media.

They are unique URLs that contain all the information needed for anyone to pay you. They are just URLs like this:

`https://paywithfi.re/ez2scsqk`

This means they can be tweeted, posted to Facebook, sent as a text or email - whatever method of communication you already have in place can be used.

If an existing fire.com user receives the Payment Request, they can tap it to view the details of the payment in their Personal Mobile app. They can then enter their PIN to pay securely. The payment is instant and arrives in the business fire.com account immediately.

If a non-fire.com user receives the request, tapping it will bring them to a web site showing the details of the payment. From there they can easily go to the app store to get the Personal Mobile app, top it up with a card, and pay. 


```shell
# Full details of an individual payment request.
{
	"code":"p5u6umne",
	"type":"SHAREABLE",
	"direction":"SENT",
	"status":"ACTIVE",
	"currency":"EUR",
	"amount":500,
	"description":"April Membership fees",
	"myRef":"April membership fees",
	"dateCreated":"2016-04-26T19:03:56.930Z"
}
```


The payment request resource has the following attributes: 

Field | Description
--------- | -----------
`code` | This is the Payment Request Code that is assigned by fire.com to this request. Used in the URL to share this request.
`type` | The type of the request - either `DIRECT` (a one-to-one request to a specific mobile number) or `SHAREABLE` (a request that can be paid by anyone who receives it)
`direction` | either `SENT` or `RECEIVED` depending on whether you sent or received the request.
`status` | This can be `ACTIVE`, `EXPIRED` or `CLOSED`. Only `ACTIVE` requests can be paid.
`currency` | Either `EUR` or `GBP`
`amount` | The amount of the request if provided. This will be in minor units (cents or pence) 
`description` | The description of the request. This will be displayed to the user when they tap or scan the request.  
`myRef` | An internal description of the request if you want a different one to the one that's shown to the public.
`dateCreated` | Date of the payment request. _(ISO formatted)_


## Create a new shared Payment Request
```shell
# Create the JSON object for the new Payment Request
# cat payment-request.json
{
	"icanTo":2150,
	"currency":"EUR",
	"amount":3000,
	"myRef":"Fees for April 2016",
	"description":"April Membership Fees",
	"maxNumberPayments":null,
	"maxNumberCustomerPayments":null
}

# Post that to the API
curl https://api.fire.com/business/v1/paymentrequests \
  -X POST \
  -d @payment-request.json \
  -H "Authorization: Bearer $ACCESS_TOKEN"

{
	type: "SHAREABLE", 
	code: "kmxe4rxz"
}
```

Creates a new Shareable Payment Request. This returns a code that can be shared to your customers as a URL by any channel you wish.  


### HTTP Request

`POST  https://api.fire.com/business/v1/paymentrequests`

### Returns

A JSON object containing the code and the type of the request.


### JSON Input 

Parameter | Description
--------- | -----------
`icanTo` | The ican of the account to collect the funds into. Must be one of your fire.com Accounts. 
`currency` | Either `EUR` or `GBP`, and must correspond the account select in `icanTo`.
`amount` | _(Optional)_ Provide a specific amount to pay. Omit the parameter if you want to allow the user to choose their own amount to pay.
`myRef` | _(Optional)_ An internal description of the request.
`description` | _(Optional)_ A public facing description of the request. Will be shown to the user when they tap or scan the request.
`maxNumberPayments` | _(Optional)_ The max number of people who can pay this request. Omit to let anyone pay.
`maxNumberCustomerPayments` | _(Optional)_ The max number of times each person can pay the request. Omit to allow any number of times.

### JSON Output

Parameter | Description
--------- | -----------
`code` | The code for this request. Create a URL using this code like this: `https://paywithfi.re/<code>` and share to your customers.
`type` | Either `DIRECT` or `SHAREABLE`. `DIRECT` is a one-to-one request to the specified mobile number. `SHAREABLE` is a public request that can be viewed and paid by anyone who receives it. 










# Webhooks
Webhooks allow you to be notified of events as they happen on your fire.com accounts. This is useful if you have systems that need to know when things happen on your account, such as payments or withdrawals. 

A webhook is a URL that you set up on your backend. We can then send the details of various events to you at this URL as they happen. You can have many webhooks, and can configure each one to listen for different events in fire.com. 

## Configuring your webhook settings
You can set up webhooks in the Business Account web application or use the API to configure them programmatically. 

## View Webhooks
```shell
curl https://api.fire.com/business/v1/webhooks \
  -X GET \
  -H "Authorization: Bearer $ACCESS_TOKEN"

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

`GET https://api.fire.com/business/v1/webhooks`

### Returns
An array of webhook event configuration objects.

Parameter | Description
--------- | -----------
`webhook` | The details of the webhook. `webhook.id` and `webhook.businessId` are identifiers, and `webhook.webhookUrl` is the actual URL of the webhook.
`events` | An array of fire.com Account events that will trigger a call to this webhook. 

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
        "type": "LODGEMENT",
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
        "amountBeforeCharges": 300,
        "feeAmount": 3,
        "taxAmount": 0,
        "amountAfterCharges": 297,
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
visibile to anyone, the signature is created using a shared secret that only you and fire.com have access to, so you can be sure that it came from us. 

A JWT looks like this:

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM`
`0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV`
`9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

This needs to be decoded using the library from JWT.io. You should ensure that the signature is valid. There are a set of Webhook API Tokens in the Profile / Webhooks section of the Business fire.com Account application. The Key ID (`kid`) in the JWT header will be the `Webhooks` public token, and you should use the corresponding private token as the secret to verify the signature on the JWT.  


