# Verifying the transaction

When a payment is completed, you need a way to check that the transaction was successful before providing value to the customer.

Your script needs to 

- Verify the transaction via our API
- Check if the transaction reference has already been given value.
- Redirect to the success page or echo JSON as required by your integration.

Verification happens on the backend and makes use of your secret key. To verify transactions, call the [verify endpoint of the Paystack API](https://developers.paystack.co/v2.0/reference#verify-transaction). For example:

```
curl https://api.paystack.co/transaction/verify/DG4uishudoq90LD \
-H "Authorization: Bearer SECRET_KEY"
```

<p class="callout danger"><b>Verifying the transaction</b></br>
When checking if transaction is successful, check for `data.status==='success'` not `status=='success'`. The first checks for if the actual transaction was successful while the second checks if the API call was successful.</p>
