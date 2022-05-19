# MobilePay Online

## Table of Contents
  - [Table of Contents](#table-of-contents)
  - [Product description](#product-description)
  - [Development Guide](#development-guide)
  - [Sandbox environment](#sandbox-environment)
  - [Merchants](#merchants)
  - [Payments](#payments)
  - [Dual and single device flow](#dual-device-and-single-device-flow)
  - [Checkout](#checkout)
  - [Request Fishing Scenario](#request-fishing-scenario)
  - [Restrictions](#restrictions)
  - [Strong Customer Authentication (SCA)](#strong-customer-authentication-sca)
  - [Callbacks](#callbacks)
  - [Invalidation of payment](#invalidation-of-a-payment)
  - [Prefilled phone number](#prefilled-phone-number)
  - [Disable remember-me](#disable-remember-me)
  - [Automated integration tests in Sandbox](#automated-integration-tests-in-sandbox)

**Appendix**<br />
  - [Merchant documentation](#merchant-documentation)
  - [Error codes](#error-codes)
  - [Retry policy](#retry-policy)
  - [Allowed currencies](#allowed-currencies)
  - [Allowed card types](#allowed-card-types)
  - [Diagrams](#diagrams)
  - [Embedded Flow](#embedded-flow)

## Product description

MobilePay Online is essentially a way for the user to accept eCommerce/online payments in the MobilePay app. When the user accepts the payment, their card data is transferred to the PSP who can then do the authorization towards the acquirer chosen by the web shop.
[![](./assets/Top-image.png)](./assets/Top-image.png)
You can find an example of a MobilePay Online flow [here](https://mobilepay.dk/erhverv/apps-og-webshops/mobilepay-online#Video-betal-online).

MobilePay is a digital wallet for iOS and Android that can be used for person to person transfers, in-store purchases, subscriptions, invoices, eCommerce transactions etc. The documentation on this page covers the technical setup of the eCommerce functionality and is directed at Payment Service Providers that wish to offer MobilePay as a payment method to their customers (i.e. web shops). 

## Development Guide

In order to implement MobilePay Online you must first obtain an agreement by contacting partnership_online@mobilepay.dk. If you have already signed an agreement with MobilePay for the use of MobilePay Online, you can skip this step

When your agreement is signed, please visit our Developer Portal to start the [technical onboarding](https://developer.mobilepay.dk/products/online/getting-started) <br>
You will also find the API on the [Developer Portal](https://developer.mobilepay.dk/product): click 'Online'.

On the developer portal you will  find information about [test](https://developer.mobilepay.dk/products/online/test) and [how to move to production](https://developer.mobilepay.dk/products/online/verification). 

## Sandbox environment

The MobilePay Sandbox is a self-contained, testing environment that mimics the live MobilePay production environment. It provides you the space to play around and test your implementation and the requests you make to the MobilePay API.
Find information about test app and sandbox test on the [developer portal](https://developer.mobilepay.dk/products/online/test)

## Merchants

As a PSP, you need to create the merchants in MobilePay in order to create payments on their behalf.

This can be done by invoking the "create merchant" endpoint (POST /merchants/).
The details supplied in the merchant onboarding will be used for invoicing purposes and ongoing support. 

When a Merchant is no longer using the solution it must be offboarded using the "delete merchant" endpoint (DELETE /merchants/).

All merchants must be onboarded as individual merchants. If you have super merchants or payment facilitors you can also use them for MobilePay. However each submerchant must be onboarded instead of the super merchant and payment facilitator. 

## Payments

 - In order to create a payment you need to invoke the "create payment" endpoint (POST to /payments/).
To use this you need to provide information about the merchant, the payment, the public key used for encrypting the data, callback-, and redirection urls.
This will return an url the end-user should be redirected to.

 - When the user has accepted the payment in the MobilePay app, you'll receive a callback on the url defined in 1. containing the card data and you can create the authorization.

 - When you have successfully authorized the payment (or it has failed), you must patch the authorisationAttempt and we'll show a receipt (or error message) to the user. Do not expect the user to return client side / in the UI.
	 - If the authorization fail we will allow the user to accept the payment again (encouraging them to use another card). This will result in a new callback to your callback url with a new authorizationAttempt. 

 - When the merchant makes captures, refunds, or cancels the payment the status of the payment must be updated to reflect this. The updates are used for invoicing purposes. To update the payment use these:
	 - POST payments/{paymentId}/captures 
	 - POST payments/{paymentId}/cancels 
	 - POST payments/{paymentId}/refunds
Please see the diagrams in the appendix to understand the [flow](#MobilePay-Online#payments-1)

## Dual device and single device flow
MobilePay Online supports both dual device and single device flows. It depends on which device the user uses when you redirect to redirectToMobilePayUrl. Our landing page will detect if the redirect happens on a mobile device or not. Only the user experience will be affected by type of flow. The payment flow and API requests are the same regardless of flow type. 

 - Single device flow: If the redirect happens on a mobile device the MobilePay app will be launched. The user can then accept or reject the payment and we will redirect the user to redirectFromMobilePayUrl afterwards. This url will always open in the default browser (determined by the operating system) of the mobile device. 
	 - If the MobilePay app is not installed and therefore cannot be opened our landing page will open in the browser instead and the flow will continue as a dual device flow.
	 - It is also possible to pop the app directly instead of using the universal link. To do this, you will need to use the redirectToMobilePayAppUrl given in the response from initiating a payment. This can be useful when using frames. See [Manually engaging the App](#manually-engaging-the-app-from-the-parent-page) for a guide on this.
 - Dual device flow: If the redirect happens on a desktop device, our landing page will open in the browser. The user can then insert the phone number or it can be pre-filled for more convenience (insert link). We will then send a push notification to the user's mobile device and the user can then accept or reject the payment in the MobilePay app. Afterwards the landing page in the desktop browser will redirect to redirectFromMobilePayUrl. 

## Checkout
With MobilePay Checkout webshops can offer their customers to complete purchases even faster by linking payment and delivery details. When using Checkout MobilePay will supply you with the billing address and optionally the delivery address of the customer. When initiating a Checkout payment the user will then be prompted to choose address before accepting the payment in the MobilePay app. 

Please visit our [best practice guide](https://github.com/MobilePayDev/MobilePay-Online/blob/master/Checkout_best_practice.md) for guidelines and recommendations on how to use MobilePay Checkout in a webshop.

**How to initiate a Checkout payment**

Checkout can be used on any MobilePay Online payment and is initiated as any other payment using:

    POST /payments
    
Four parameters are required when initiating a Checkout payment:

    "isCheckout":true,
    "addressCallbackUrl":"http://example.com",
    "deliveryAddressAllowed":true,
    "deliveryAddressDisallowedReasonCode":4, //Only required if "deliveryAddressAllowed":false.

 - isCheckout: This will define the payment as a Checkout payment when set to true
 - addressCallbackUrl: The url where we will deliver the checkout callback with the address details. The Checkout callback will always be delivered before the card/token callback.
 - deliveryAddressAllowed: Will define whether the user is allowed to choose delivery address. If set to false the user can only choose billing address. Can be used for instance if the user has chosen 'Pick up at store' in the webshop.
 - deliveryAddressDisallowedReasonCode: if deliveryAddressAllowed is set to false a reason must be given:
1: Reason not given, 2: Goods don't require physical delivery, 3: 'Pick up at store' already selected in the Webshop, 4:'Parcel Shop' already selected in the Webshop, 5: Shop will select a parcelshop close to your home"

In the appendix you can find a [digram of the checkout payment flow](https://github.com/MobilePayDev/MobilePay-Online#checkout-1). 

**Checkout callback**

The callback will have a JSON body like this:

    {
      'PaymentId': '9369ea35-4b5b-428a-bdf8-c29c29a4b264',
      'AuthorizationAttemptId': 'a8c99cbf-3468-4eb9-9c0e-ddd110e8ed33',
      'Addresses': [{
        'FirstName': 'John',
        'Surname': 'Doe',
        'Attention': '',
        'CompanyName': '',
        'AddressLine1': 'Flower Street 23',
        'AddressLine2': '',
        'PostalCode': '3434',
        'City': 'Great city',
        'CountryCode': 'DK',
        'IsCustomerOfficialAddress': true,
        'IsBillingAddress': true,
        'IsDeliveryAddress': true,
        'AddressValidationMethod': 'DaWa',
        'AddressValidationStatus': 'NotValidated'
      }],
      'EmailAddress': 'johndoe@gmail.com',
      'EmailAddressValidationMethod': 'EmailEnteredTwice',
      'EmailAddressValidationStatus': 'Validated',
      'PhoneNumber': '+4512345678',
      'PhoneNumberValidationMethod': 'SMSChallenge',
      'PhoneNumberValidationStatus': 'Validated'
    }

The address array may contain two items. One for billing address and one for delivery address. If billing and delivery address is the same the array will only contain one item where IsBillingAddress and IsDeliveryAddress is set to true. 

If deliveryAddressAllowed is set to false on payment initiation the array will only contain one item where IsBillingAddress is set to true and IsDeliveryAddress is set to false. 

## Request Fishing Scenario

This scenario is a thought out "attack" where a fraudster tricks someone else to pay for the goods, by sending the request to multiple users from our "dual device" website, until someone accepts the payment. 

Initialize payment (POST /payment/) is idempotent. However, if it is called with the same set of MerchantId and PSPReferenceId (but anything else might have changed) - request fishing will be initiated.
Depending on the scenario a DomainError will be returned stating the problem. If the user initiates  more than 3 payments, with the same MerchantId and OrderId, a permanent DomainError will be returned.

## Restrictions

A payment will time out by default within 35 minutes, meaning that the whole process of user accepting, callbacks made and authorization must be completed within 35 minutes.
Furthermore after you get the callback containing the card data, you must update the status of the authorization to either "authorize-succesfull" or "authorize-failed" within 32 seconds to ensure a smooth experience for the user waiting for the confirmation.

## Strong Customer Authentication (SCA)

We aim to ensure Delegated Authentication (DA). This means that responsibility for authenticating the customer/payer no longer lies with the Issuer, but is delegated to MobilePay.  When/if we fail, and the Issuer is responding to an authorisation attempt with a Soft Decline/"step-up", a 3-D Secure fallback solution must be in place.

All of the below SCA implementations are mandatory. Only Dankort can be skipped if you do not accept Dankort.

Note: Delegated Authentication can only be effectuated by the Issuer if the authorisation is based on a token. If you for some reason receive a PAN-based transaction from us, you should handle data in the card data callback as non-authenticated PAN-transactions according to scheme rules to ensure correct handling regarding SCA. If in doubt, please contact your acquirer

### Delegated Authentication for Dankort
As long as you use the tags and values described here, all is well. Nets will recognize MobilePay and trust our authentication process.

Use POS code: ‘K005K0K00130’.
#### Using Nets SDI specification
In Field S120 tag 36: the value of 8844101001</br>
In Field S120 tag 70 pos 14 (exemption Tag): the value of 3 for Delegated Authentication
#### Using Nets TRG PSIP/ ISO 8583 / Merchant Guide SSL
In Field 47 tag 7R: the value of 8844101001</br>
In Field 47 tag V!: the value of 23

### Delegated Authentication using tokens
Both Visa Token Service (VTS) and Mastercard S4C (MS4C) are based on the EMVCo standard. Please pass the data to the Acquirer, as you would do if the token response was from your own VTS or MS4C integration. In case you have questions to the Acquirer API, please ask the Acquirer.
When you get the token callback, if you internally (without calling anyone) know, that token will not work for this payment, please reply HTTP code 501, we will then immediately fail over to encrypted card data callback.

When you initiate a payment, make sure to use v3 of the API. Here you give a tokenCallbackUrl for all accepted Visa and Mastercard types. However, please also provide a carddataCallbackUrl as failover, because not all cards can be tokenized. For Dynamic Linking, please give us MerchantUrl and MerchantName.

When you recieve the token callback, you´ll find a cardIssuedInCountryCode (possible values DK, FI) you can use for your Acquirer routing logic.

Exactly as for encrypted card data callbacks, make sure you respond to the callback immediately. DO NOT leave the transaction "hanging" while you call out to the Acquirer. If we do not get a response to the callback in proper time we will allow the user to accept the payment again. 

[![](./assets/token.svg)](./assets/token.svg)

### Delegated Authentication for all Visa cards
A Visa Token Service (VTS) response like this: <br />
```
{
	"paymentId": "string",
	"authorizationAttemptId": "string",
	"cardType": "string",	
	"cardIssuedInCountryCode": "string",
	"maskedCardNumber":"string",
	"tokenMethod": "string",
	"tokenData": {
		"vPaymentDataID": "string",
		"cryptogramInfo": {
			"cryptogram": "string",
			"eci": "string"
		},
		"paymentInstrument": {
			"last4": "string",
			"paymentType": {
				"cardBrand": "string"
			},
			"paymentAccountReference": "string"
		},
		"tokenInfo": {
			"token": "string",
			"last4": "string",
			"expirationDate": {
				"month": "string",
				"year": "string"
			}
		},
	},
	"isDelegatedAuthentication": bool
}
```
</br>
Example: </br>

```
{
   "paymentId":"8dab9219-ab03-4524-bae7-f0ad55119da5",
   "authorizationAttemptId":"32eedb2b-a536-4eb6-b618-c2d6c1bf7aab",
   "cardType":"VISA-CREDIT",
   "cardIssuedInCountryCode":"DK",
   "maskedCardNumber":"479694XXXXXX1234",
   "tokenMethod":"VTS",
   "tokenData":{
      "vPaymentDataID":"da17bd1568bdc8b418d71cf80c44ea02",
      "cryptogramInfo":{
         "cryptogram":"/wAAAAwAUkMTObMAAAAAgS0AAAA=",
         "eci":"07"
      },
      "paymentInstrument":{
         "last4":"1234",
         "paymentType":{
            "cardBrand":"VISA"
         },
         "paymentAccountReference":"V0010013020217426481676671969"
      },
      "tokenInfo":{
         "token":"4895737462013403",
         "last4":"3403",
         "expirationDate":{
            "month":"02",
            "year":"2023"
         }
      }
   },
   "isDelegatedAuthentication": false
}
```

### Delegated Authentication for all Mastercards
A Mastercard S4C (MS4C) response like this: <br />
```
{
   "paymentId":"1ba21790-5e10-4db1-8e90-330fb41916e7",
   "authorizationAttemptId":"3205ec7c-2d50-49d2-95dc-326e34edce47",
   "cardType":"MC-CREDIT",
   "cardIssuedInCountryCode":"DK",
   "maskedCardNumber":"520473XXXXXX4792",
   "tokenMethod":"MC S4C",
   "tokenData":{
      "token":{
         "paymentToken":"5204731613942625",
         "tokenExpirationMonth":"05",
         "tokenExpirationYear":"2024",
         "paymentAccountReference":"5001BO8B9NXVVIXCT0HAJU98I512Z"
      },
      "dynamicData":{
         "dynamicDataType":"CARD_APPLICATION_CRYPTOGRAM_SHORT_FORM",
         "dynamicDataValue":"MD1eEaqbngDNAy0iuRqOAAADFEA="
      },
      "eci":"06"
   }
}
```

### 3DSecure Fallback
If Delegated Authentication fails, the 3DSecure fallback solution applies. 

[![](./assets/3dsfallback.svg)](./assets/3dsfallback.svg)

When the user has completed the challenge, please immediately redirect to https://products.mobilepay.dk/remote-website/apppages/done3ds.html
For Sandbox use: https://sandprod-products.mobilepay.dk/remote-website/apppages/done3ds.html

If the user cancels the 3DS challenge on the 3DS website or if it fails in some way, you should fail the authorization attempt with reasonCode=1000 and redirect to the done3ds.html page. This will cancel the 3DS flow in the MobilePay app and allow the user to retry with another card (starting a new authorization attempt).

Optional: As soon as you have 3DS crypto from ACS and before retrying the authorization-attempt towards Acquirer and Issuer, you can call the MP backend with reasonCode=1009. This enables us to prevent the user from retrying with another card.

## Callbacks

As a rule of thumb, MobilePay Online is idempotent in all operations. Likewise, we expect PSPs to be able to handle the same callback more than once in the event of transient errors on network, ours or your side.
This means that if we make a callback to you on a given payment id or a given authorization attempt, you may receive the same data more than once and should ensure that your systems are able to handle that.
We will retry our callbacks for 15 seconds in the event of network errors or non 200-range http status codes in your responses. After this we will mark the callback as failed and allow the user to accept the payment again. In this case you will receive a new callback with a new authorization attempt. Please note that the 15 second limit may be subject to change. 

### Card data callback

A callback will be made on the encryptedPanCallbackUrl when the user swipes to accept the payment. The callback will have a JSON body like this:
```
{
  'EncryptedCardData': 'fsfnsdjkfbgdft34895u7345',
  'PaymentId': 'a84781b3-af34-42ae-b296-260cfb6859fe',
  'AuthorizationAttemptId': 'ba12c5d5-8fd1-49cc-bc3f-2cb2ecb888c7',
  'PublicKeyId': 263012,
  'CardType' : 'DANKORT'
}
```
The EncryptedCardData is encrypted according to this OAEP algorithm&padding scheme: RSA/NONE/OAEPWithSHA256AndMGF1Padding . 
Once decrypted, you´ll see:
{"timestampticks":123456789123456789,"encryptedCardData": { "cardNumber": 1234567812345678, "expiryMonth": 12, "expiryYear": 28 }

Make sure you respond to the callback immediately, once you´ve decrypted the content. DO NOT leave the transaction "hanging" while you call out to the Acquirer. 

Strictly process the payment on the card type given in field "CardType"! This is the only way MobilePay can offer card type picking for co-branded cards in accordance with PSD2 requirements. 
Also the SCA method vary on the cobranded Visa/Dankort. This means that if you process the PAN given with CardType=DANKORT as a Visa, the SCA is missing and the (unwanted) 3DS step-up is likely to happen.

Note: When receiving a PAN-based transaction from us, it can't be regarded as authenticated. You should handle these according to scheme rules to ensure correct handling regarding SCA. If in doubt, please contact your acquirer.

### Failed callback

In case the payment times out, a callback is made to the FailedPaymentCallbackUrl url supplied in the initiate payment call. Note that this callback is not sent if the user rejects the payment, only when payment times out. 

```
{
    'Code': '100',
    'Reason': 'Payment expired',
    'PaymentId': '8d72ece4-1b0b-464b-98d9-6bbb02199dc8'
}
```

### Endpoint security
To ensure no unauthorized calls to your callback endpoints, we strongly suggest you apply either (or both) of these limitations:
1. Firewall rules. These are the IP-ranges that MobilePay backend will be calling from: 212.93.32.0/19 and 185.218.228.0/22
2. SSL "Common name" inspection. The "Common name" in our SSL client certificate should never change. Even when the certificate itself changes, or is issued to a different root.

## Invalidation of a payment
In some cases the user goes back to the merchant webshop and adds something to the shopping basket after the payment has been initiated. The user could end up with several requests with different amounts and there would be a possibility, that the user accepts the wrong one in MobilePay. In this case payment invalidation endpoint can be useful. When the invalidation is completed, it means that the user cannot create a request for the payment or accept the payment. Active requests will also expired immediately.

### Invalidation before callback
If the invalidation request is received by MobilePay before the callback to the PSP is made, no callback will be performed and the invalidation will complete.

![Invalidation before callback](./assets/invalidation-before-callback.svg)

### Invalidation after callback
If the invalidation request is received by MobilePay after the callback to the PSP is performed, the PSP must also PATCH the authorizationAttempt with succeeded=false and the ReasonCode 1010 - RejectedForInvalidatePayment.

In this case the invalidation request will be processed in the MobilePay backend according to these rules:
1. If a successful authorization already exists on the payment, the invalidation endpoint will return the error code 2100.
2. If a callback has been sent but the authorization attempt has not yet been patched or 3DS is ongoing, the invalidation endpoint will return the error code 2101. The authorization attempt must be patched with reasoncode 1010 before invalidation.
3. A failed authorization will result in a successful invalidation.

![Invalidation after callback](./assets/invalidation-after-callback.svg)

## Prefilled phone number
You can provide a phone number to be prefilled in the phone number field on the MobilePay webpage. 
You do this by adding an URI encoded "alias" parameter with the phone number to the search parameters of the "redirectToMPUrl". The phone number must be fully specified including country code. For "+45 12 34 56 78" you would add the following to the url: `&alias=004512345678`

> /remote-website/index.html?page=request&id=d7380749-f73c-4317-bd01-58daa249212f&alias=004512345678

## Disable remember-me
If you, for instance, have a public terminal where MobilePay is used, you may want to disable the option to remember the current phone number.
You can do this by adding the query parameter key `disable-remember-me` to the request page url. 

> /remote-website/index.html?page=request&id=d7380749-f73c-4317-bd01-58daa249212f&disable-remember-me&alias=004512345678

## Automated integration tests in Sandbox

The API supports automated integration tests running in Sandbox and can be found at [sandbox-developer](https://sandbox-developer.mobilepay.dk/product) under 'Online User Simulation' and then 'Cardpassthrough RegressionTester For Product'. The API support simulation of the required user actions from payment request to capture. For more information on the general payment flow, please see [the diagram section](#Diagrams).

**Please refrain from any overly aggressive testing strategies, we expect any continously running tests to call at most once per second.**


# Appendix

## Merchant documentation
The documentation towards your customers, the merchants, must - at least - tell about

 - *Merchant registration*: How to setup MobilePay Online including as minimum information about: 
	 - *Name* displayed in the app to the end user
	 - *LogoUrl* linking to an image file displaying the merchant logo in the app to the end user
		 - 250x250 pixels
		 - Hosted using a secure HTTPS connection
		 - PNG or JPG
		 - Set content-type in the HTTP header using *MIME* types e.g. *image/png* or *image/jpeg* 
 - *Appswitch feature*: How the solution is used from a native app (API enabled)
 - *Checkout feature*: How to setup and use Checkout
 
 - *Common pitfalls* of 'context switch' on client side<br />
A. Scenario - ‘browser A’ -> MP App -> ‘browser B’<br/> 
The Merchant return page should not rely on any sort of session object (e.g. a cookie), to recognise the returning customer. It should solely rely on data given in the redirect (redirectFromMobilePayUrl).<br /> 
B. Scenario - ‘browser' -> MP App’ <br/>
The Merchant should not rely on the customer returning client side. Rather they should process the purchase when the PSP´s server-to-server callback is received, or after getting confirmation on a status endpoint.

## Error codes

The following will describe the error codes thrown and in which cases they can occur.

The error format for status code 409 will be the following:

```json
{
    "code": "2020",
    "message": "Some description",
    "correlationId": "8d72ece4-1b0b-464b-98d9-6bbb02199dc8"
}
```

| Code | Endpoint(s) | Description
|:---|:---|:---|
| 2000 | POST /payments | Merchant doesn't exist
| 2010 | POST /payments | The merchant isn't created by you
| 2020 | POST /payments | The merchant is deleted
| 2030 | POST /payments | Allowed card types are not set
| 2040 | POST /payments | One or more of the allowed card types are invalid
| 2050 | POST /payments | Currency code is invalid
| 2100 | PUT payments/{paymentId:guid}/invalidate | Can't invalidate payment with completed authorization attempt 
| 2101 | PUT payments/{paymentId:guid}/invalidate | Can't invalidate payment - the authorization attempt has not yet been patched with success or failure. Try again later.

## Retry policy
Even though we have above 99% uptime and handle millions of transactions each week, external factors, such as network related issues, can contribute to momentary disturbances in our response times.
To reduce the risk of such issues having negative impact on the customer experience, you should implement a retry mechanism.

Our suggestions:
* If you do not get a response within 2 seconds, try again. 
* If you get an unexpected error (HTTP > 299) wait 1 second, then try again. 
* Suggested retry count is 2 (three attempts in total). You should under no circumstance retry more than 6 times. As this could worsen any ongoing situation in our setup.

You’re welcome to make use of more advanced retry strategies (exponential back-off, circuit breakers etc.), as long as you take the suggestions above into consideration. 


## Allowed currencies

Currencies must be specified according to the ISO-4217 standard. Either using the alphabetic or numeric version.
Allowed currencies are:

| Name | Alphabetic code | Numeric code
|:---|:---|:---|
| Danish kroner | DKK | 208
| Euro | EUR | 978
| Norwegian kroner | NOK | 578
| Swedish kroner | SEK | 752
| United States dollar | USD | 840
| Pound sterling | GBP | 826

## Allowed card types

The following card types are allowed:

| Card name | Code
|:---|:---|
| Visa electron | ELEC-DEBIT |
| Mastercard credit | MC-CREDIT |
| Mastercard debit | MC-DEBIT |
| Maestro | MTRO-DEBIT |
| Visa credit | VISA-CREDIT |
| Visa debit | VISA-DEBIT |
| Dankort | DANKORT |

## Diagrams

### Merchants

![Merchants](./assets/merchant-sequence-diagram.svg)

### Payments

![Payments](./assets/payment-sequence-diagram.svg)
![Payments](./assets/payment-with-native-app-switching-sequence-diagrams.svg)

### Checkout

![Checkout](./assets/checkout-sequence-diagram.svg)

### When acquirer or issurer rejects a payment

![When acquirer or issurer rejects a payment](./assets/acquirer-or-issuer-reject-payment-sequence-diagram.svg)

### When the user rejects a payment

![When the user rejects a payment](./assets/user-rejects-payment-sequence-diagram.svg)

### After authorization

![After authorization](./assets/after-authorization-sequence-diagram.svg)

## Embedded Flow

Web shops and Payment Service Providers (PSPs) may embed the MobilePay landing
page inside their own website by nesting the page in an IFrame and then
listen to the events emitted from the IFrame.

Embedding is especially relevant for 'dual-flow' use; where the user starts by
entering his or her phone number in a browser and then continues the flow in the
app to accept the payment.

When the user has accepted the payment (swiped) in the app, the landing page
will by default redirect to the url provided when the payment was created.

Build your logic on the parent page, listen for the events published by the
IFrame and redirect the user to the right page based on the returned data,
e.g. depending on whether the user cancelled or completed the payment.

### Embed the website in an IFrame

Add an "IFrame" to the html source and set the IFrame "src" property to the URL
returned from the payment link creation endpoint.

On mobile devices it is expected that the MobilePay flow visually covers the
whole screen (simple header and footer is acceptable).

Be aware that you might want to show the user different content if the user is inside
an IFrame or not. If you support both IFrame and full window, we recommend that
you have a neutral return page without visual content. 

The width should be 375px.

Example
```html
 <IFrame 
   scrolling="no"
   src="https://products.mobilepay.dk/remote-website/index.html?page=request&id=83554a83-cd90-4ac9-bf6e-39357c21dca5&version=2"
   style="width: 375px; height: 480px; border: 0;" >
</IFrame>
```

### Add an Event Listener to the parent page of the IFrame

The parent page can listen for events by adding an event listener to the IFrame.

Before the IFrame redirects, it will post a message to all registered event
handlers, giving any parent page the alternative option of handling the page
navigation instead, for example in the case where you want the parent page to
change too.

```javascript
window.addEventListener( 
    "message",
    function(event) {
        if (event.data && event.data.startsWith("mobilepay:")){
            // Do your logic
            // Continue purchase processing
            alert(event.data);
        }
    },
    false);
```

When the user terminates the payment flow, the published event will look like as
follows. The code and matching description indicate how the flow was terminated,
i.e. either because the payment was completed, rejected, cancelled by the user
or because the timer expired.

```js
mobilepay:rc={code}&message={description}

// example
mobilepay:rc=0&message=completed
```

| Response Code | Description
|:--|:----------|
| 0 | completed |
| 1 | rejected  |
| 3 | expired   |
| 4 | cancelled |


### Manually engaging the App from the parent page

On mobile devices the app is not guaranteed to engage when the website is nested
inside an IFrame.

To preserve the expected behavior of engaging the app on mobile devices, the
parent of the IFrame may try to engage the app with the following javascript
snippet that use a custom url registered for both Android and iOS. 
This custom url (redirectToMobilePayAppUrl) is part of the response received, when a payment is initiated.

You should still display the landing page inside the IFrame in case the app is
not installed or the user navigates back to your page.

```javascript
function tryToOpenMobilePayApp(paymentId) {
   try {
      window.open("_redirectToMobilePayAppUrl", "_parent");
   } catch {
      // window.open should just fail without throwing aside from logging an 
      // error to console.
      //
      // This catch is strictly for good measure to avoid any incidents
      // in legacy or future browser settings.
   }
}
```
