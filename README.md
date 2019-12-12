# MobilePay Online

## Product description

MobilePay Online is essentially a way for the user to accept online payments in the Mobilepay app. When the user accepts the payment, their card data is encrypted and transferred to the PSP who can then do the authorization towards the merchants chosen acquirer.

## API guidelines

As a rule of thumb, the APIs are RESTful. You can expect POSTs to return the id of the resource created.

## Sandbox environment

The MobilePay Sandbox is a self-contained, testing environment that mimics the live MobilePay production environment. It provides you the space to play around and test your implementation and the requests you make to the MobilePay APIs without affecting MobilePay for the users.
Link: https://sandbox-developer.mobilepay.dk/

## Developer portal

The description of the Online API and the endpoints can be seen on the developer portal.
Link: https://developer.mobilepay.dk/node/2871

## Partner (PSP)

A PSP is created manually by the support team, after signing an agreement with the MobilePay Business unit. The Partner must give these input: 
* Name
* VATNumber
* BusinessContactEmail
* BusinessContactPhone
* OperationalContactEmail
* OperationalContactPhone

Attach a copy of PCI-DSS AoC and a RSA public key as a X.509 SubjetPublicKeyInfo (using ASN.1 DER Encoding) represented in PEM encoding. Key length must be 4096 bits. This public key will be used for encrypting card data for the callback. Make sure you keep the private key safe. It is suggested to make different keys for our Sandbox (integration) and Prod environments.

The Partner will be assigned a PartnerId, and recieve the security credentials for both the Sandbox (integration environment) and Production environment.

## Merchants

As a PSP, you need to create the merchants in MobilePay in order to create payments on their behalf.

This can be done by invoking the "create merchant" endpoint.

When a Merchant is no longer using the solution it must be offboarded using the "delete merchant" endpoint.

## Payments

1. In order to create a payment you need to invoke the "create payment" endpoint.
To use this you need to provide information about the merchant, the payment, the public key used for encrypting the data, callback-, and redirection urls.
This will return an url the end-user should be redirected to.

2. When the user has accepted the payment in the MobilePay app, you'll receive a callback on the url defined in 1. containing the encrypted card data and you can create the authorization.

3. When you have successfully authorized the payment (or it has failed), you'll call us and we'll show a receipt (or error message) to the user

4. When the merchant makes captures, refunds, or cancels the payment the status of the payment must be updated to reflect this to give the best possible user experience.

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

A callback will be made on the CardDataCallbackUrl when the user swipes to accept the payment. The callback will have a JSON body like this:

```
{
  'EncryptedCardData': 'fsfnsdjkfbgdft34895u7345',
  'PaymentId': 'a84781b3-af34-42ae-b296-260cfb6859fe',
  'AuthorizationAttemptId': 'ba12c5d5-8fd1-49cc-bc3f-2cb2ecb888c7',
  'PublicKeyId': 263012
}
```

## PSP Onboarding Guide
### If you are onboarding MobilePay Online for the first time 
When you as a new PSP wants to be onboarded for the Online solution, you need to send an email to the MobilePay business support with this information:
* The PSP name
* PCI-DSS AoC
* VATNumber
* BusinessContactName
* BusinessContactEmail
* OperationsEmail
* OperationsPhonenumber
*Two PublicKeys for Card encryption: The RSA public key should be provided as a X.509 SubjectPublicKeyInfo (using ASN.1 DER Encoding) represented in PEM encoding (use PEM file extension). The public key must have a length of 4096 bits. Clearly stating in the file name wich one is for Sand-Prod and wich is for Prod. 

### If you are already using MobilePay Online
When you as an existing PSP wants to be onboarded for the new Online solution, you need to send an email to the MobilePay business support with this information:
* The PSP name
* A new PublicKey for Card encryption in SandBox environment: The RSA public key should be provided as a X.509 SubjectPublicKeyInfo (using ASN.1 DER Encoding) represented in PEM encoding (use PEM file extension). The public key must have a length of 4096 bits. 
* BusinessContactName
* BusinessContactEmail
* OperationsEmail
* OperationsPhonenumber

### How to call the Online APIs in SandBox
1. Go to sandbox-developer.mobilepay.dk and log in with your credentials.
2. Next you select your account > My Apps > Create new App to register a new application. 
3. Retrieve the Client Id and Client Secret for the newly created App. IMPORTANT: Please make a note of your Client Secret as you will only see this once!
4. Send the SandBox Client Id to MobilePay Support.
5. Subscribe the App to the Online API.
6. You should have received a PublicKeyId for SandBox from MobilePay Support . This Id should be used when the payments are initiated.
7. Call the endpoints in the Online API using these headers:
       --header 'x-ibm-client-id: REPLACE_THIS_KEY' 
       --header 'x-ibm-client-secret: REPLACE_THIS_KEY'

### How to call the Online APIs in production
Same guide as above, but use this url https://developer.mobilepay.dk and use the PublicKeyId for production.

### How to test MobilePay Online in SandBox
1. You will need a special version of the MobilePay App for the SandBox environment. A link to it can be requested from MobilePay Support.
2. MobilePay Support will send a Phonenumber and Password, which can be used for login to the App in SandBox.
3. If you need more cards to test on, they can be requested by MobilePay Support.
