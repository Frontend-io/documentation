# Sending Money

Paystack allows users send money to any Nigerian bank account. This is ideal for services that need to disburse funds into their user's bank account, like paying vendors, disbursing loans, payroll, creditting winners on a gaming platform. You can initiate both simple transfer to a single bank account and bulk transfers to multiple accounts.

*[You can transfers from your Paystack dashboard if you want to send money without using the API.][https://paystack.helpscoutdocs.com/article/50-paystack-transfers]* 

The source of the money can be directly from your bank account or from your Paystack Balance. Before you send a transfer using the API, you need to first create a `transfer recipient`

##Transfer Recipient

A transfer recipient is a code that represents the bank account and customer you are sending the payment to. When the transfer recipient is created, Paystack returns a recipient code which you can then store against the user on your database.

Here is the curl request format for creating transfer recipients:

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

##Initiating a Transfer

Like we mentioned earlier, we you can make single and  bulk transfers to the transfers endpoint. Below are the curl codes for initating both transfers.

#### Single Transfer

```
curl https://api.paystack.co/transfer \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"source": "balance", "reference": "dsio02ie2eui", "reason": "Your investment Returns", "amount":3794800, "recipient": "RCP_gx2wn530m0i3w3m"}' \
-X POST
```

The endpoint accepts recipient code, amount (in kobo), source, reference and a reason/description. The reference can be a unique identifier from your system to identify this particular transaction. The response will contain a transfer code.

#### Bulk Transfer

```
curl https://api.paystack.co/transfer \
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

This one takes the a `transfers` array of objects containing `amount` and `recipient` 