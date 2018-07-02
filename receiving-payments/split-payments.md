# Split Payments

The split payments feature enables you split transaction fees across two accounts - the main account which is the Paystack merchant and a subaccount.

An ideal scenario for this feature will be a marketplace platform that sells goods on behalf of its vendors. Paystack can automatically split the payouts such that the vendor's bank account is credited with his share and the platform owner gets credited with his own fees too. The good news is that we have built this in such a way that it can work across multiple use cases.

At the moment, payments can only be split across two accounts.

Implementing split payments involve:

- [Step 1: Creating a subaccount](https://developers.paystack.co/v2.0/docs/split-payments#section-creating-a-subaccount)
- [Step 2: Initialize a split payment](https://developers.paystack.co/v2.0/docs/split-payments#section-initialize-a-split-payment)

## Creating a subaccount
Subaccounts can be created [via the Paystack Dashboard](https://dashboard.paystack.com/#/subaccounts) or [via the Paystack API](https://developers.paystack.co/v1.0/reference#create-subaccount).  When a subaccount is created, the `subaccount_code` and the `account_name` is returned. Please endeavor to verify that the bank account details matches what you intended. Paystack will not be liable for payouts to the wrong bank account.

## Initialize a split payment
Split payments can be initialized by passing the parameter `subaccount: "SUB_ACCOUNTCODE"`.

```
var paymentForm = document.getElementById('paymentForm');
paymentForm.addEventListener("submit", payWithPaystack, false);

function payWithPaystack(e) {  
  e.preventDefault();
  var config = {
    key: 'pk_live_xxxxxxxxxxxxxxxxxxxxxxxxx', // Replace with your public key
    email: document.getElementById("email-address").value,
    amount: document.getElementById("amount").value * 100,
    .
    .
    .
    subaccount: 'ACCT_xxxxxxxxx',
    onClose: function(){
      alert('Window closed.');
    },
    callback: function(response){
      var message = 'Payment complete! Reference: ' + response.reference;
      alert(message);
    }
  };
  
var paystackPopup = new Popup(config);
paystackPopup.open();
```

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{
    "reference": "7PVGX8MEk85tgeEpVDtD", 
    "amount": 500000, 
    "email": "customer@email.com",
    "subaccount": "ACCT_xxxxxxxxx",
}' \
-X POST
```

## Other Information

### Flat Fee	

Payments are split on Paystack by percentage i.e. 20% going to main account and the rest going to subaccount. These parameters are required when creating the subaccount. However, there are instances where you will rather collect a flat fee per payment. To do this, pass a paramter called `transaction_charge = 1000 //amount in kobo`.

### Bearer of Paystack Charge

You can use which party bears the Paystack charges when making a split payment between the main account and the subaccount. By default, the charges are borne by the main account. To change this to the subaccount, pass a parameter `bearer: "subaccount"` on intializing a transaction.