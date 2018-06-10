# Transfers

The funds transfers feature enables you send money directly from your Paystack balance to any Nigerian Bank account.

Payments are made from your Paystack balance so ensure your balance is funded by carrying out a Topup or [changing your settlement schedule](https://paystack.helpscoutdocs.com/article/32-difference-between-manual-and-next-day-paystack-payouts) to MANUAL.

## Implementation

1. [Create a transfer recipient](https://developers.paystack.co/reference#create-transfer-recipient) to make a transfer to. If the same bank details of an existing transfer recipient is passed, that recipient's recipient code is returned as opposed to a new recipient with the same details being created.

   ```
   curl -X POST -H "Authorization: Bearer SECRET_KEY" -H "Content-Type: application/json" -d '{ 
      "type": "nuban",
      "name": "James Bond",
      "description": "Agent 007",
      "account_number": "01000000010",
      "bank_code": "044",
      "currency": "NGN",
      "metadata": {
         "description": "License to kill"
       }
    }' "https://api.paystack.co/transferrecipient"
   ```

   

2. [Initiate a transfer](https://developers.paystack.co/reference#initiate-transfer).

   ```
   curl https://api.paystack.co/transfer \
   -H "Authorization: Bearer SECRET_KEY" \
   -H "Content-Type: application/json" \
   -d '{"source": "balance", "reason": "From Russia, with Love", "amount":3794800, "recipient": "RCP_gx2wn530m0i3w3m"}' \
   -X POST
   ```

   

3. If OTP has not been disabled, [finalise transfer](https://developers.paystack.co/v1.0/reference#finalize-transfer). OTPs are required to conclude a transfer by default but you can disable this requirement by calling the [Disable OTP requirement for Transfers](https://developers.paystack.co/reference#disable-otp-requirement-for-transfers) endpoint or from the Settings page in the Paystack Dashboard. An OTP will be sent to your phone to confirm the disable request. Use the [Finalize Disabling of OTP requirement for Transfers](https://developers.paystack.co/reference#finalize-disabling-of-otp-requirement-for-transfers) endpoint to submit this OTP and conclude the process if you are disabling via the API. Future Transfers will not request OTP before starting.

   ```
   curl https://api.paystack.co/transfer/disable_otp \
   -H "Authorization: Bearer SECRET_KEY" \
   -X POST
   ```

   

## Other Information

When a transfer is initiated, the expected status is `"pending"` as Paystack's service tries to complete the transfer.

When transfer is completed i.e. the status changes to `"success"` or `"failed"`, an event notification is sent to your webhook.

Here are some sample events for Transfers.

```
{
  event: "transfer.success",
  data: {
    domain: "live",
    amount: 10000,
    currency: "NGN",
    source: "balance",
    source_details: null,
    reason: "Bless you",
    recipient: {
      domain: "live",
      type: "nuban",
      currency: "NGN",
      name: "Someone",
      details: {
        account_number: "0123456789",
        account_name: null,
        bank_code: "058",
        bank_name: "Guaranty Trust Bank"
      },
      description: null,
      metadata: null,
      recipient_code: "RCP_xoosxcjojnvronx",
      active: true
    },
    status: "success",
    transfer_code: "TRF_zy6w214r4aw9971",
    transferred_at: "2017-03-25T17:51:24.000Z",
    created_at: "2017-03-25T17:48:54.000Z"
  }
}
```

```
{
  "event": "transfer.failed",
  "data": {
    "domain": "test",
    "amount": 10000,
    "currency": "NGN",
    "source": "balance",
    "source_details": null,
    "reason": "Test",
    "recipient": {
      "domain": "live",
      "type": "nuban",
      "currency": "NGN",
      "name": "Test account",
      "details": {
        "account_number": "0000000000",
        "account_name": null,
        "bank_code": "058",
        "bank_name": "Zenith Bank"
      },
      "description": null,
      "metadata": null,
      "recipient_code": "RCP_7um8q67gj0v4n1c",
      "active": true
    },
    "status": "failed",
    "transfer_code": "TRF_3g8pc1cfmn00x6u",
    "transferred_at": null,
    "created_at": "2017-12-01T08:51:37.000Z"
  }
}
```

For more information, see [Events](https://developers.paystack.co/v2.0/docs/events).