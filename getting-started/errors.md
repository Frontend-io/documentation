# Errors

Paystack's API is RESTful and as such, uses conventional HTTP response codes to indicate the success or failure of requests.

## Status code

`200`,` 201` - Request was successful and intended action was carried out. Note that we will always send a 200 if a charge or verify request was made. Do check the data object to know how the charge went (i.e. successful or failed).

`400` - A validation or client side error occurred and the request was not fulfilled.

`401` - The request was not authorized. This can be triggered by passing an invalid secret key in the authorization header or the lack of one

`404` - Request could not be fulfilled as the request resource does not exist.

`500`, `501`, `502`, `503`, `504` - Request could not be fulfilled due to an error on Paystack's end. This shouldn't happen so please report as soon as you encounter any instance of this.

## Common Errors
We discuss some common errors and their causes below

### Authorization code does not exist or has been deactivated

- All authorizations are inactive until one transaction succeeds on the card. Attempting charge auth on an inactive authorization will give this message.
- You can only charge an authorization for the customer who saved it. If an authorization is paired with the wrong email, you may get this error message.

### This authorization is not reusable

Only reusable authorizations can be used with our charge authorization endpoint. Please confirm reusability on the authorization object before making an attempt to avoid this message.

### Invalid Subaccount

All subaccounts codes on Paystack start with SUB_ and only exist on the integration that created them. And in the domain they were created. Common situations when you will get this message include:

1. Sending subaccount id or name or any value other than a valid code. Rather don't send any subaccount if you do not intend to split the transaction.
2. Sending subaccount code from another integration.
3. Sending subaccount code created in the test domain while specifying a live key and vice-versa.

### Email does not match Authorization code. Please confirm

- You can only charge an authorization for the customer who saved it. If an authorization is paired with the wrong email, you may get this error message.

### Invalid Split transaction values

This happens when you have attempted to split the payment in a way that is mathematically impossible. The transaction may have specified the subaccount as bearer though the subaccount's share doesn't cover paystack's fees or vice-versa. You may also have specified a transaction charge that was above the amount being paid.

### Duplicate Transaction Reference

Every transaction on your integration in a domain must have a unique refrence. You will get this message if you attempt to start a new transaction with a reference that has already been used.

### Invalid

All objects on Paystack are tied to a domain and integration. Getting an "Invalid X" message might mean the one specified does not exist in the space commanded by the key presented.