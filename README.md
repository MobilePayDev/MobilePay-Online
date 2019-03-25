# MobilePay Online

## Product description

MobilePay Online is essentially a way for the user to accept online payments in the Mobilepay app. When the user accepts the payment, their card data is encrypted and transferred to the PSP who can then do the authorization towards the merchants chosen acquirer.

## API guidelines

As a rule of thumb, the APIs are RESTful. You can expect POSTs to return the id of the resource created.

## Merchants

As a PSP, you need to create the merchants in MobilePay in order to create payments on their behalf.

This can be done by invoking the [create merchant](https://www.youtube.com/watch?v=dQw4w9WgXcQ) endpoint.

## Creating a payment

1. In order to create a payment you need to invoke the [create payment](https://www.youtube.com/watch?v=dQw4w9WgXcQ) endpoint.
To use this you need to provide information about the merchant, the payment, the public key used for encrypting the data, callback-, and redirection urls.
This will return an url the end-user should be redirected to.

2. When the user has accepted the payment in the MobilePay app, you'll receive a callback on the url defined in 1. containing the encrypted card data and you can create the authorization.

3. When you have successfully authorized the payment (or it has failed), you'll call us ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)) and we'll show a receipt (or error message) to the user

4. When the merchant makes captures ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)), refunds ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)), or cancels the payment ([here](https://www.youtube.com/watch?v=dQw4w9WgXcQ)) the status of the payment must be updated to reflect this to give the best possible user experience.

## Restrictions

A payment will time out within 20 minutes, meaning that the whole process of user accepting, callbacks made and authorization must be completed within 20 minutes.
Furthermore after you get the callback containing the card data, you must update the status of the authorization within 20 seconds to ensure a smooth experience for the user waiting for the confirmation.
