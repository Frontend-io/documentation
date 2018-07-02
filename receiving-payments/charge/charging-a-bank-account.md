# Charging a Bank Account

The  Pay with Bank feature allows your customers pay you by providing their bank account number and an OTP sent to their phone. In the case of GTB, your customers will be shown the ibank interface to conclude payment.

Collect your customer's bank account number directly to start a Pay With Bank transaction. In test mode, use one of our test bank accounts to test. 

Note that only banks listed by this api call: https://api.paystack.co/bank?gateway=emandate&pay_with_bank=true are supported at the time your customer is about to pay.

## Step 1: List Banks

To start a transaction, show a list of available banks by calling this url: https://api.paystack.co/bank?gateway=emandate&pay_with_bank=true. 

The banks can be listed in a dropdown or any other format that allows the user to easily pick their bank of choice.

## Step 2: Create a charge

Send email, amount, metadata, bank (an object that includes code of bank and account_number supplied by customer), birthday to our Charge endpoint as available to start.

```
curl https://api.paystack.co/transaction/charge \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{
    "reference": "7PVGX8MEk85tgeEpVDtD", 
    "amount": 500000, 
    "email": "customer@email.com",
    "bank":{
      code:"057",
      account_number:"0000000000"
    },
}' \
-X POST
```
 When a charge is made, the default status is pending as the payment is being processed in the background. The status can then change to several different status before being a "success" or failing as a "failed" transaction.

Here are the possible status responses.

- data.status == "pending" - Transaction is being processed. Call Check pending charge at least 10 seconds after getting this status to check status.
- data.status == "timeout" - Transaction has failed. You may start a new charge after showing data.message to user.
- data.status == "success" - Transaction is successful. Give value after checking to see that all is in order.
- data.status == "send_birthday" - Customer's birthday is needed to complete the transaction. Show data.display_text to user with an input that accepts the birthdate and submit to the  Submit Birthday endpoint with reference and birthday.
- data.status == "send_otp" - Paystack needs OTP from customer to complete the transaction. Show data.display_text to user with an input that accepts OTP and submit the OTP to the   Submit OTP with reference and otp.
- data.status == "failed" - Transaction failed. No remedy for this, start a new charge after showing data.messageto user
- data.status == false - Log so you may debug your logic.

Charging returning customers directly is not currently available. Simply provide the account number again to start the transaction for a returning customer. Because bank authorizations are not currently reusable so doing a charge authorization on them will fail. This document will be updated as soon as direct debits are available.