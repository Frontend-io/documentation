# Charging via USSD

The Pay via USSD channel allows your customers to pay you by dialling a USSD code on their mobile device. This code is usually in the form of * followed by some code and ending with #. The user is prompted to authenticate the transaction with a PIN and then it is confirmed.

All you need to initiate a USSD charge is the customer email and the amount to charge.  

When the user pays, a response will be sent to your webhook. Hence, for this to work properly as expected, webhooks must be set up on your Paystack Dashboard.

## Step 1: Create a charge

Send an email and amount to the [charge](https://developers.paystack.co/v2.0/reference#charge) endpoint. Specify the USSD type you are charging as well. Available types at the moment is the GTB `737` USSD code.

```
curl https://api.paystack.co/transaction/charge \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{
    "reference": "7PVGX8MEk85tgeEpVDtD", 
    "amount": 500000, 
    "email": "customer@email.com",
    "ussd" : {
          "type" : "737"
      }
}' \
-X POST
```

When a charge is made, the default response provides a USSD code for the customer to dial to complete the payment.

```
{
    "status": true,
    "message": "Charge attempted",
    "data": {
        "reference": "yjr1r8rwhedara4",
        "status": "pay_offline",
        "display_text": "Please dial *737*33*4*18791# on your mobile phone to complete the transaction",
        "ussd_code": "*737*33*4*18791#"
    }
}
```
## Step 2: Handle response

When the user completes payment, a response is sent to the merchant’s webhook. Hence, for this to work properly as expected, webhooks must be set up for the merchant..

The `charge.success` event is raised on successful payment. The sample response to be sent to the user’s webhook would look like:

```
{
  "event": "charge.success",
  "data": {
    "id": 53561,
    "domain": "live",
    "status": "success",
    "reference": "2ofkbk0yie6dvzb",
    "amount": 150000,
    "message": "madePayment",
    "gateway_response": "Payment successful",
    "paid_at": "2018-06-25T12:42:58.000Z",
    "created_at": "2018-06-25T12:38:59.000Z",
    "channel": "ussd",
    "currency": "NGN",
    "ip_address": "54.246.237.22, 162.158.38.185, 172.31.15.210",
    "metadata": "",
    "log": null,
    "fees": null,
    "fees_split": null,
    "authorization": {
      "authorization_code": "AUTH_4c6mhnmmeusp4yd",
      "bin": "XXXXXX",
      "last4": "XXXX",
      "exp_month": "05",
      "exp_year": "2018",
      "channel": "ussd",
      "card_type": "offline",
      "bank": "Guaranty Trust Bank",
      "country_code": "NG",
      "brand": "offline",
      "reusable": false,
      "signature": null
    },
    "customer": {
      "id": 16200,
      "first_name": "John",
      "last_name": "Doe",
      "email": "customer@email.com",
      "customer_code": "CUS_bpy9ciomcstg55y",
      "phone": "",
      "metadata": null,
      "risk_action": "default"
    },
    "plan": {
      
    },
    "subaccount": {
      
    },
    "paidAt": "2018-06-25T12:42:58.000Z"
  }
}
```

Charging returning customers directly is not currently available. Simply call the endpoint to start a new transaction.