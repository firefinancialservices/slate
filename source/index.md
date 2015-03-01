---
title: Pay with Fire Business Account API Reference

language_tabs:
  - shell: cURL
  - java: Java

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:

search: true
---

# Getting Started with the Pay with Fire Business Account API

The Pay with Fire API allows you to deeply integrate our banking features into your application.

# Authentication

```shell
# With shell, you can just pass the correct header with each request
curl <https://paywithfire.com/business/v1/me>
  -H "Authorization: privateToken"
```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");
```

Replace `privateToken` in the code samples with your private token from your Business Account Profile page.


# External Bank Accounts 

```shell
# JSON representation of an External Account
{
	"externalAccountId": 23492,
	"alias": "BoI Current Account",
	"currency": "EUR",
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
You can retrieve the details of an external account by its `externalAccountId`. 

### HTTP Request

`GET https://paywithfire.com/business/v1/me/externalaccounts/{externalAccountId}`

Parameter | Description
--------- | -----------
`externalAccountId` | This is the ID of the external account Id to be returned.

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
curl https://paywithfire.com/business/v1/me/externalaccounts/379487
  -X POST
  -d @paymentdetails.json
  -H "Authorization: privateToken"

```

```java
PayWithFireSDK businessAccount 
	= new PayWithFireSDK(Environment.SANDBOX, "privateToken");

Payment payment = new Payment()
	.setAmount(3748)
	.setCurrency("EUR")
	.setNarrative("REF 2347839")
	.setComment("Paid for March Bill");

try {
	GenericResult res 
		= businessAccount.externalAccounts().pay(payment);
} catch (PaymentExceedsLimitException pele) {
	// Check the status for reason
	pel.getStatus();
} catch (NoSuchExternalAccountException nseae) {
	// dang
}

```

You can pay from any of your Fire Accounts to an existing External Bank Account. 

Depending on the authorisation rules configured by your account administrator, the API call may only set up the transfer rather than actually execute it. Check the status of the response to know what happened. 

Some administrators may not allow payments via the API at all, and others can place retrrictions on the total value of payments over a period of time. If you receive an error 403, check the status of the response for more information.


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

Delete an external account from your profile.
