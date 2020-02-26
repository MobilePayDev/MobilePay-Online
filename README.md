# MobilePay Online

## Table of Contents
**[Product description](#product-description)**<br />
**[Development Guide](#development-guide)**<br />
**[API guidelines](#api-guidelines)**<br />
**[Sandbox environment](#sandbox-environment)**<br />
**[Merchants](#merchants)**<br />
**[Payments](#payments)**<br />
**[Request Fishing Scenario](#request-fishing-scenario)**<br />
**[Restrictions](#restrictions)**<br />
**[Callbacks](#callbacks)**<br />

**Appendix**<br />
**[Error Codes](#error-codes)**<br />
**[Allowed currencies](#allowed-currencies)**<br />
**[Allowed card types](#allowed-card-types)**<br />
**[Diagrams](#diagrams)**

## Product description

MobilePay Online is essentially a way for the user to accept online payments in the Mobilepay app. When the user accepts the payment, their card data is encrypted and transferred to the PSP who can then do the authorization towards the merchant's chosen acquirer.

## Development Guide

### Step 1 Read and understand the documentation 
Please read both the description here in GitHub and the API in Developer Portal: https://developer.mobilepay.dk/product (click 'Online').
The diagrams in the appendix [Diagrams](#diagrams) should also be helpful to understand the different flows.


### Step 2 Technical onboarding
When you as a PSP wants to be onboarded for the Online solution, you must first have an agreement with MobilePay. Please contact partnership_online@mobilepay.dk to obtain this. When the agreement is signed, please visit our developer portal to start the technical onboarding: https://developer.mobilepay.dk/products/online/getting-started

### Step 3 Make your own test merchant
By posting to /merchants/. Further reading: https://github.com/MobilePayDev/MobilePay-Online/blob/master/README.md#merchants

### Step 4 Initiate a payment
By posting to /payments/. Further reading: https://github.com/MobilePayDev/MobilePay-Online/blob/master/README.md#payments

You´ll get the redirectToMPUrl in reponse. Now is also a good time to start working on your callback endpoint (the service you expose on cardDataCallbackUrl). Make sure it at least consumes the POST and reply http200.

### Step 5 Try the payment in the app
Open the "redirectToMPUrl" in a browser (or from an app), and try the payment flow.

### Step 6 Decrypt the card data, call the Acquirer and update the authorisationAttempt
Decrypt the cardData from the callback and call the Acquirer.
When the Acquirer reply (or timeout), make sure you Patch our authorisationAttempt with the new status.

### Step 7 Move to hidden Production
Ask developer@mobilepay.dk for an invitation to the Production Developer Portal https://developer.mobilepay.dk/ to get Production credentials, similar to sandbox. 
Deploy your solution into "hidden production". Make a test webshop, and share the link to it with us (developer@mobilepay.dk) for a "slim certification". Do not proceed to step 8 before we´re happy!

### Step 8 Public production
Document everything (including Checkout with all features) towards your Merchants in a fantastic documentation. Just the way your customers want it. Go live!

## API guidelines

As a rule of thumb, the APIs are RESTful. You can expect POSTs to return the id of the resource created.

## Sandbox environment

The MobilePay Sandbox is a self-contained, testing environment that mimics the live MobilePay production environment. It provides you the space to play around and test your implementation and the requests you make to the MobilePay APIs without affecting MobilePay for the users.
Link: https://sandbox-developer.mobilepay.dk/

Installing the sandbox MobilePay app and logging in to it
1.	Open one of these links on the phone were you want to install the sandbox app: 
•	Android DK: https://dbg.tpa.io/p/KnSXxG8NQ8Mv0yhct5iC
•	iOS DK: https://dbg.tpa.io/p/h-XHpPXMT3PgvNiKtalW

2.	Download and install the app from the link (please just disregard any “POS vendor” references).
•	Android: To install, you’ll have to allow installation from “unknown sources”.
•	iOS: When the app is installed, you have to accept “MobilePay A/S” as app distributor under Settings > General > Profiles & Device management, to make it work.
3.	After installation, open the app (If you get a prompt about a new version being available, just install it).
4.	Select “Log på (Eksisterende bruger)”
5.	Ensure that the environment selector is set to ”Integrator Sandbox (With Login)
6.	Enter a valid Sandbox phone number (you can get one from the Developer Support team)
7.	Enter “1234” as pin
8.	Enter “123456” as activation code and press OK
9.	You should now be logged in and see the main page of MobilePay

Testing a MobilePay Online payment
1.	As PSP, you can now initiate a MobilePay Online payment against our Sandprod environment.
2.	When you see our payment website, enter the phone number you used in the sandbox app.
3.	Ideally, you should now get a notification (not yet working in sandbox)
4.	Login to the app
5.	You should automatically see a confirmation page for the payment
6.	Swipe to approve

## Merchants

As a PSP, you need to create the merchants in MobilePay in order to create payments on their behalf.

This can be done by invoking the "create merchant" endpoint (POST /merchants/).

When a Merchant is no longer using the solution it must be offboarded using the "delete merchant" endpoint.

## Payments

1. In order to create a payment you need to invoke the "create payment" endpoint (POST to /payments/).
To use this you need to provide information about the merchant, the payment, the public key used for encrypting the data, callback-, and redirection urls.
This will return an url the end-user should be redirected to.

2. When the user has accepted the payment in the MobilePay app, you'll receive a callback on the url defined in 1. containing the encrypted card data and you can create the authorization.

3. When you have successfully authorized the payment (or it has failed), you'll patch the authorisationAttempt endpoint and we'll show a receipt (or error message) to the user.

4. When the merchant makes captures, refunds, or cancels the payment the status of the payment must be updated to reflect this to give the best possible user experience. See the API.

## Request Fishing Scenario

This scenario is a thought out "attack" where a fraudster tricks someone else to pay for the goods, by sending the request to multiple users from our "dual device" website, until someone accepts the payment. 

Initialize payment (POST /payment/) is idempotent. However, if it is called with the same set of MerchantId and OrderId, but anything else has changed - request fishing will be initiated.
Depending on the scenario a DomainError will be returned stating the problem. If the user initiates  more than 3 payments, with the same MerchantId and OrderId, a permanent DomainError will be returned.

## Restrictions

A payment will time out within 20 minutes, meaning that the whole process of user accepting, callbacks made and authorization must be completed within 20 minutes.
Furthermore after you get the callback containing the card data, you must update the status of the authorization to either "authorize-succesfull" or "authorize-failed" within 32 seconds to ensure a smooth experience for the user waiting for the confirmation.

## Callbacks

As a rule of thump, MobilePay Online is idempotent in all operations. Likewise, we expect PSPs to be able to handle the same callback more than once in the event of transient errors on network, ours or your side.
This means that if we make a callback to you on a given payment id or a given authorization attempt, you may receive the same data more than once and should ensure that your systems are able to handle that.
We will retry our callbacks for more than 5 seconds in the event of network errors or non 200-range http status codes in your responses.

### Card data callback

A callback will be made on the CardDataCallbackUrl when the user swipes to accept the payment. The callback will have a JSON body like this:
```
{
  'EncryptedCardData': 'fsfnsdjkfbgdft34895u7345',
  'PaymentId': 'a84781b3-af34-42ae-b296-260cfb6859fe',
  'AuthorizationAttemptId': 'ba12c5d5-8fd1-49cc-bc3f-2cb2ecb888c7',
  'PublicKeyId': 263012
}
```
The EncryptedCardData is encrypted according to the OAEP padding scheme. Once decrypted, you´ll see:
{
  "encryptedCardData": {
    "cardNumber": "1234567812345678",
    "expiryMonth": "12",
    "expiryYear": "28"
  }
}

### Failed callback

In case the payment times out, a callback is made to the FailedPaymentCallbackUrl url supplied in the initiate payment call.

```
{
    'code': '100',
    'reason': 'Payment expired',
    'paymentId': '8d72ece4-1b0b-464b-98d9-6bbb02199dc8'
}
```

### Checkout callback

A callback will be made on the AddressCallbackUrl when the user swipes to accept the payment and isCheckout is set to true. The callback will have a JSON body like this:

```
{
  'PaymentId': '9369ea35-4b5b-428a-bdf8-c29c29a4b264',
  'AuthorizationAttemptId': 'a8c99cbf-3468-4eb9-9c0e-ddd110e8ed33',
  'Addresses [
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
  ],
  'EmailAddress': 'johndoe@gmail.com',
  'EmailAddressValidationMethod': 'EmailEnteredTwice',
  'EmailAddressValidationStatus': 'Validated',
  'PhoneNumber': '+4512345678',
  'PhoneNumberValidationMethod': 'SMSChallenge',
  'PhoneNumberValidationStatus': 'Validated'
}
```

# Appendix

## Error codes

The following will describe the error codes thrown and in which cases they can occur.

The error format will be the following:

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

## Allowed currencies

Currencies must be specified according to the ISO-4217 standard. Either using the alpha or the numeric version.
Allowed currencies are:

| Name | Alphabetic code | Numeric code
|:---|:---|:---|
| Danish kroner | DKK | 208
| Euro | EUR | 978
| Norwegian kroner | NOK | 578
| Swedish kroner | SEK | 752

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

### Checkout

![Checkout](./assets/checkout-sequence-diagram.svg)

### When acquirer or issurer rejects a payment

![When acquirer or issurer rejects a payment](./assets/acquirer-or-issuer-reject-payment-sequence-diagram.svg)

### When the user rejects a payment

![When the user rejects a payment](./assets/user-rejects-payment-sequence-diagram.svg)

### After authorization

![After authorization](./assets/after-authorization-sequence-diagram.svg)
