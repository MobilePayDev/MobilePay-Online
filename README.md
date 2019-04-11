## MobilePay Online

## Product description

MobilePay Online is essentially a way for the user to accept online payments in the Mobilepay app. When the user accepts the payment, their card data is encrypted and transferred to the PSP who can then do the authorization towards the merchants chosen acquirer.

## API guidelines

As a rule of thumb, the APIs are RESTful. You can expect POSTs to return the id of the resource created.

## Partner (PSP)

A PSP is created manually by the support team, after signing an agreement with the MobilePay Business unit. The Partner must give these input: Name, VATNumber, BusinessContactEmail, BusinessContactPhone, OperationalContactEmail and OperationalContactPhone. Attach a copy of PCI-DSS AoC and a RSA public key as a X.509 SubjetPublicKeyInfo (using ASN.1 DER Encoding) represented in PEM encoding. Key length must be 4096 bits. This public key will be used for encrypting card data for the callback. Make sure you keep the private key safe. It is suggested to make different keys for our Sand-prod (integration) and Prod environments.
The Partner will be assigned a PartnerId, and recieve the security credentials for both the Sand-Prod (integration environment) and Production environment.


## Merchants

As a PSP, you need to create the merchants in MobilePay in order to create payments on their behalf.

This can be done by invoking the [create merchant](https://www.youtube.com/watch?v=dQw4w9WgXcQ) endpoint.

When a Merchant is no longer using the solution it must be offboarded using the [delete merchant](https://www.youtube.com/watch?v=dQw4w9WgXcQ) endpoint.

## Payments

1. In order to create a payment you need to invoke the [create payment](https://www.youtube.com/watch?v=dQw4w9WgXcQ) endpoint.
To use this you need to provide information about the merchant, the payment, the public key used for encrypting the data, callback-, and redirection urls.
This will return an url the end-user should be redirected to.

2. When the user has accepted the payment in the MobilePay app, you'll receive a callback on the url defined in 1. containing the encrypted card data and you can create the authorization.

3. When you have successfully authorized the payment (or it has failed), you'll call us ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)) and we'll show a receipt (or error message) to the user

4. When the merchant makes captures ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)), refunds ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)), or cancels the payment ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)) the status of the payment must be updated to reflect this to give the best possible user experience.

## Restrictions

A payment will time out within 20 minutes, meaning that the whole process of user accepting, callbacks made and authorization must be completed within 20 minutes.
Furthermore after you get the callback containing the card data, you must update the status of the authorization to either [authorize-succesfull](https://www.youtube.com/watch?v=dQw4w9WgXcQ) or [authorize-failed](https://www.youtube.com/watch?v=dQw4w9WgXcQ) within 32 seconds to ensure a smooth experience for the user waiting for the confirmation.
