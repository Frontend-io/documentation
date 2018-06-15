# Sending Money

Paystack allows users send money to any Nigerian bank account. This is ideal for services that need to disburse funds into their user's bank account, like paying vendors, disbursing loans, payroll, creditting winners on a gaming platform. You can initiate both simple transfer to a single bank account and bulk transfers to multiple accounts.

*[You can transfers from your Paystack dashboard if you want to send money without using the API.][https://paystack.helpscoutdocs.com/article/50-paystack-transfers]* 

The source of the money can be directly from your bank account or from your Paystack Balance. Before you send a transfer using the API, you need to first create a `transfer recipient`

## Transfer Recipient

A transfer recipient is a code that represents the bank account and customer you are sending the payment to. When the transfer recipient is created, Paystack returns a recipient code which you can then store against the user on your database.

[Here is the curl request format for creating transfer recipients][https://developers.paystack.co/v1.0/reference#create-transfer-recipient]:

```
curl 
-X POST 
-H "Authorization: Bearer YOUR_SECRET_KEY" 
-H "Content-Type: application/json" -d '{ 
   "type": "nuban",
   "name": "Zombie",
   "description": "Zombier",
   "account_number": "01000000010",
   "bank_code": "044",
   "currency": "NGN",
   "metadata": {
      "job": "Flesh Eater"
    }
 }' 
 "https://api.paystack.co/transferrecipient"
```

If you are building an application that will constantly be sending money to the customers, you can create the `transfer recipient` during user registration, and store the the `recipient_code` against the user. Subsequently, when you want to send money to the user, you can just retrieve the recipient code and use it it to initiate the transfer.

## Initiating a Transfer

Like we mentioned earlier, we you can make single and  bulk transfers to the transfers endpoint. Here are the curl codes for initating both transfers.

#### Single Transfer

```
curl https://api.paystack.co/transfer \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"source": "balance", "reference": "dsio02ie2eui", "reason": "Your investment Returns", "amount":3794800, "recipient": "RCP_gx2wn530m0i3w3m"}' \
-X POST
```

[The endpoint accepts recipient code, amount (in kobo), source, reference and a reason/description][https://developers.paystack.co/v1.0/reference#initiate-transfer]. The reference can be a unique identifier from your system to identify this particular transaction. The response will contain a transfer code.

#### Bulk Transfer

```
curl -X POST \
  https://api.paystack.co/transfer/bulk \
  -H 'authorization: Bearer sk_test_36658e3260b1d1668b563e6d8268e46ad6da3273' \
  -H 'content-type: application/json' \
  -d '{
	"currency": "NGN",
	"source": "balance",
	"transfers": [
		{
		"amount": 50000,
		"recipient": "RCP_s97mdaiv1qgeyn0",
		"reference": "32e929ie29ie29"
		},
		{
		"amount": 50000,
		"recipient": "RCP_v43ra0ql43b7h78",
		"reference": "90f92302j39d203"
		}
	]
}'
```

[This one takes the a `transfers` array of objects][https://developers.paystack.co/v1.0/reference#initiate-bulk-transfer] containing `amount   ` ,  `recipient` and `reference` for each of the transfers. The response should tell you that the transfers have been queued. If you don't pass the references, Paystack will generate for you. 

### Transfer Status Notification

Because the transfers take a few minutes to process, the transfer status always start as `pending`. When the transfer is processed successfully, Paystack sends an event notification to your webhook URL. If the transfer succeeds, we send a `transfer.success` event, if it fails, we send a `transfer.failed` event. [Please follow this link to learn more about Paystack events and webhooks][https://developers.paystack.co/v1.0/docs/events].  

To check the status of a transfer at anytime, [you can call the fetch transfer endpoint][https://developers.paystack.co/v1.0/reference#fetch-transfer] using the transfer code.

```
curl "https://api.paystack.co/transfer/TRF_2x5j67tnnw1t98k" \
-H "Authorization: Bearer YOUR_SECRET_KEY" 
-X GET
```

### Topping Up Balance

When sending transfers from your balance, sometimes you may not have enough money on your Paystack Balance. Paystack allows you topup your balance directly to make up the finds for the transfer. [Learn more about how to topup your balance here.][https://paystack.helpscoutdocs.com/article/67-the-paystack-topup-feature]

We also have an API that you can query to get your Paystack balance. You can have this linked to your internal dashboard so you can always know how much is in your Paystack Balance, you can also query the endpoint before making a transfer to ensure you have enough money for the transfer. [Here is a link to the balance API documentation][https://developers.paystack.co/v1.0/reference#check-balance].

#### Balance and Settlement

When money is paid to you by customers, the money isn't immediately available for transfers. It is only available the next day after Paystack Settlement has run to subtract Paystack Transaction Fees and any other deductibles on the account like chargebacks, BVN charges, etc. The money that is left what is available for transfers. So if you received N1,000,000 today in transactions, you will not be able to transfer that N1,000,000 until tomorrow after settlement.

This is why we made provision to top up your balance directly. The topup goes directly into your balance and is available for transfers instantly.

### Dashboard Settings For Transfer

Here a few settings you need to know about on your Dashboard that enable you use the Transfers API. 

1. **Switch To Manual Settlement**: To enable transfers from your Paystack Balance, your Settlement Schedule needs to be on Manual Settlement. This means that Paystack will not automatically pay money into your settlment bank account everyday for your transactions from the day before. This will allow the funds to be available for transfers. [Here is link to your Paystack Dashboard Settings Preferences][https://dashboard.paystack.com/#/settings/preferences] where you can change the settlement schedule [Learn more about Paystack Settlement Schedules here][https://paystack.helpscoutdocs.com/article/32-difference-between-manual-and-next-day-paystack-payouts].
2. **Disable OTP Confirmation on Transfers**: Before money is sent out of your Paystack balance, Paystack requires OTP confirmation. We send an OTP to the business phone number on your account. If you don't have OTP disabled, every transfer will require a second step where you send the OTP to finalize the transfer. [Here is link to the Finalize Transfer reference][https://developers.paystack.co/v1.0/reference#finalize-transfer]. The OTP is a useful security check for simple dashboard transfers but is not feasible for a fully automated applications. So you can disable OTP Confirmations on your Dashboard Settings Preferences. [Follow this link to the dashboard preferences to disable it][https://developers.paystack.co/v1.0/reference#finalize-transfer].

### USE CASE - Marketplace Disbursement 