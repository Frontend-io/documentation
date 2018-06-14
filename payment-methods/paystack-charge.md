# Paystack Charge

The Paystack charge payment method allows merchants to accept payments on the various Paystack payment channels (card, bank, USSD, etc) directly i.e. without having to interact with the Paystack Checkout form.

On other payment methods - Paystack Popup and Redirect - customers are shown a the Paystack Checkout form which incorporates all available Paystack channels. The charge endpoint however allows merchants to define their method of collection for each channel while Paystack does the processing on the backend.

This is achieved by making a `POST` request to the charge endpoint of the Paystack API.

## Implementation

### Charging a Card

To use the charge endpoint, your server must be PCI-DSS compliant as you will be transmitting card details from the client to the server.

To charge a card, call the [charge endpoint](https://developers.paystack.co/reference#charge) passing along a `card` object that contains the card `number`, `cvv`, `expiry_month` and `expiry_year`. 

```
{
  email:"some@body.nice",
  amount:"10000",
  card:{
    cvv:"408",
    number:"4084084084084081",
    expiry_month:"01",
    expiry_year:"99"
  }
}
```

You can also tokenise cards, which will return an authorization code that can be used to charge cards in the future via the charge endpoint. In such a case, charging that card in the future only requires an `email`, `authorization_code` and `amount`.

```
{
  email:"some@body.nice",
  amount:"10000",
  authorization_code: "AUTH_IOCE9X,
}
```

When a charge is made, the default `status` is `pending` as the payment is being processed in the background. The status can then change to several different status before being a `"success"` or failing as a `"failed"` transaction.

Here are the possible `status` responses.

- `data.status` == `"pending"` - Transaction is being processed. Call [Check pending charge](https://developers.paystack.co/reference#check-pending-charge) at least 10 seconds after getting this status to check status.

- `data.status` == `"timeout"` - Transaction has failed. You may start a new charge after showing `data.message` to user.

- `data.status` == `"success"` - Transaction is successful. Give value after checking to see that all is in order.

- `data.status` == `"send_pin"` - Paystack needs PIN from customer to complete the transaction. Show `data.display_text` to user with an input that accepts PIN and submit the PIN to the  [Submit PIN](https://developers.paystack.co/reference#submit-pin) endpoint with reference and PIN.

- `data.status` == `"send_otp"` - Paystack needs OTP from customer to complete the transaction. Show `data.display_text` to user with an input that accepts OTP and submit the OTP to the   [Submit OTP](https://developers.paystack.co/reference#submit-otp) with `reference` and `otp`.

- `data.status` == `"open_url"` - You need to let customers authenticate the transaction by filling in OTP on a page. Open `data.url` in user’s browser.

  You can specify an optional url to which we should redirect the user after the attempt is complete by adding `redirect_to=[url]` as a GET parameter.

  Call [Check pending charge](https://developers.paystack.co/reference#check-pending-charge) at least 5 seconds after user closes browser page or after 5 minutes, whichever comes first.

- `data.status` == `"failed"` - Transaction failed. No remedy for this, start a new charge after showing `data.message`to user

- `data.status` == `false` - Log so you may debug your logic.

To tokenize a card before starting the charge flow, call [Tokenize](https://developers.paystack.co/reference#charge-tokenize) with the customer's email and the card details to get an `authorization_code`. Note that this is optional since you may start the charge with the card.

## Charging A Bank Account

The  Pay with Bank feature allows your customers pay you by providing their bank account number and an OTP sent to their phone. In the case of GTB, your customers will be shown the ibank interface to conclude payment.

Collect your customer's bank account number directly to start a Pay With Bank transaction. In test mode, use one of our [test bank accounts](https://developers.paystack.co/v1.0/docs/test-bank-accounts) to test. 

Note that only banks listed by this api call: <https://api.paystack.co/bank?gateway=emandate&pay_with_bank=true> are supported at the time your customer is about to pay.

To start a transaction, show a list of available banks by calling this url: <https://api.paystack.co/bank?gateway=emandate&pay_with_bank=true>. Send `email`, `amount`, `metadata`, `bank` (an object that includes code of `bank`and `account_number` supplied by customer), `birthday` to our [Charge](https://developers.paystack.co/reference#charge) endpoint as available to start.

```
{
  email:"some@body.nice",
  amount:"10000",
  bank:{
      code:"057",
      account_number:"0000000000"
  }
}
```

When a charge is made, the default `status` is `pending` as the payment is being processed in the background. The status can then change to several different status before being a `"success"` or failing as a `"failed"` transaction.

Here are the possible `status` responses.

- `data.status` == `"pending"` - Transaction is being processed. Call [Check pending charge](https://developers.paystack.co/reference#check-pending-charge) at least 10 seconds after getting this status to check status.
- `data.status` == `"timeout"` - Transaction has failed. You may start a new charge after showing `data.message` to user.
- `data.status` == `"success"` - Transaction is successful. Give value after checking to see that all is in order.
- `data.status` == `"send_birthday"` - Customer's birthday is needed to complete the transaction. Show `data.display_text` to user with an input that accepts the birthdate and submit to the  [Submit Birthday](https://developers.paystack.co/reference#submit-birthday) endpoint with `reference` and `birthday`.
- `data.status` == `"send_otp"` - Paystack needs OTP from customer to complete the transaction. Show `data.display_text` to user with an input that accepts OTP and submit the OTP to the   [Submit OTP](https://developers.paystack.co/reference#submit-otp) with `reference` and `otp`.
- `data.status` == `"failed"` - Transaction failed. No remedy for this, start a new charge after showing `data.message`to user
- `data.status` == `false` - Log so you may debug your logic.

Charging returning customers directly is not currently available. Simply provide the account number again to start the transaction for a returning customer. Because bank authorizations are not currently reusable so doing a [charge authorization](https://developers.paystack.co/reference#charge-authorization) on them will fail. This document will be updated as soon as direct debits are available.