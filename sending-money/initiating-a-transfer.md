# Initiating a transfer

Paystack allows you to send money to any Nigerian Bank account using our **Transfers** APIs. You can initiate both single and bulk transfers. Here are the steps.

- Step 1: Get your **Secret Key**
- Step 2: Collect and **verify the Account Number**
- Step 3: Create a **Transfer Recipient**
- Step 4: Initiate **Transfer**
- Step 5: Listen for **Transfer status notification**

## Get Your Secret Key
All transfer related API requests are to be done from your server, using a **secret key**. Your dashboard has both a **test secret key** that you can use while developing, and a **live secret key** that you use for live transactions. You can [get your secret key from the API Key section your dashboard settings](https://dashboard.paystack.com/#/settings/developers). 

## Collect and Verify the Account Number
You need to collect the destination account number and confirm that the account number is valid. You can use our [Resolve Account Number API](https://developers.paystack.co/v2.0/reference#resolve-account-number) to ensure the account number is the right one. This step is important to avoid sending money to a wrong or invalid account.

To resolve an account number, send a `GET` request to the Resolve Account Number API endpoint with the account number and the bank code. Ideally, you shoud have a form on your site or app where you can collect and resolve the account number.

```
curl "https://api.paystack.co/bank/resolve?account_number=0022728151&bank_code=063" \
-H "Authorization: Bearer SECRET_KEY" \
-X GET
```

When an account number is resolved, it returns the **name on the bank account**. [Here is the API endpoint to get the list of banks and their codes](https://developers.paystack.co/v1.0/reference#list-banks).

## Create A Transfer Recipient
Before sending money to the account, you need to create a `transfer recipient` with the account details. This will return a `recipient code` that you can then initiate the transfer to. This way, you can verify an account number once and use the transfer recipient code to represent the bank account number, to prevent mistakes subsequently.

To create the transfer recipient, send the user's account number and bank code to the [transfer recipient endpoint](https://developers.paystack.co/v2.0/reference#create-transfer-recipient).

```
curl -X POST -H "Authorization: Bearer SECRET_KEY" -H "Content-Type: application/json" -d '{ 
   "type": "nuban",
   "name": "Account 1029",
   "description": "Customer1029 bank account",
   "account_number": "01000000010",
   "bank_code": "044",
   "currency": "NGN",
 }' "https://api.paystack.co/transferrecipient"
```

This returns a recipient code and you can store it in your database against your user.

## Initiate Transfer

When you are ready to send money to the account, you call the [Initate Transfer endpoint](https://developers.paystack*.co/v1.0/reference#initiate-transfer) to perform the funds transfer. The endpoint takes:

- `amount` - in kobo (Naira value * 100)
- `recipient` code
- `reference` - A reference code or ID that links this transfer to your database. If you don't pass a reference, Paystack will generate one for you.
- `source` - This is either your Paystack balance or a source bank account (coming soon)
- `reason` - Optional description for this transfer

```
curl https://api.paystack.co/transfer \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"source": "balance", "reason": "Holiday Flexing", "amount":3794800, "recipient": "RCP_gx2wn530m0i3w3m"}' \
-X POST
```

When you send this request, if there are no errors, the response comes back with a `pending` status, while the transfer is being processed. Test transfers always return success, because there is no processing involved. The live transfers processing usually take between a few seconds and a few minutes. When it's done processing, a notification is sent to your webhook URL.

The response for a transfer also contains a **unique transfer code** to identify this transfer. You can use this code to call the [Fetch Transfer endpoint](https://developers.paystack.co/v1.0/reference#fetch-transfer) to fetch status and details of the transfer anytime you want.

### Bulk Transfers

You can also initiate bulk transfers to send money to multiple transfer recipients at once. The [bulk transfers endpoint](https://developers.paystack.co/v2.0/reference#initiate-bulk-transfer) takes an array of objects containing `transferrecipients` and `amount` and queues them for processing.

```
curl https://api.paystack.co/transfer/bulk \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{
	"currency": "NGN",
	"source": "balance",
	"transfers": [
	{
	"amount": 50000,
	"recipient": "RCP_db342dvqvz9qcrn"
	},
	{
	"amount": 50000,
	"recipient": "RCP_db342dvqvz9qcrn"
	}
	]
}' \
-X POST
```

## Listen For Transfer Status Notification

As mentioned in **Step 4** above, when the transfers are initiated, the take a few seconds or minutes to process. When they have been processed, Paystack sends the final status as a POST event to your webhook URL. 

If the transfer is successful, you will get a `transfer.success` notification. If the transfer failed, you will get a `transfer.failed` event. You should get any of this notifications at most 15 minutes after initializing the transfer. [Learn more about Paystack events and webhook here](https://developers.paystack.co/docs/events).