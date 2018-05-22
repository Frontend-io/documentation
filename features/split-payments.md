# Split Payments

The split payments feature enables you split transaction fees across two accounts - the main account which is the Paystack merchant and a subaccount.

An ideal scenario for this feature will be a marketplace platform that sells goods on behalf of its vendors. Paystack can automatically split the payouts such that the vendor's bank account is credited with his share and the platform owner gets credited with his own fees too. The good news is that we have built this in such a way that it can work across multiple use cases.

At the moment, payments can only be split across two accounts.

## Implementation

1. Create a subaccount: Subaccounts can be created [via the Paystack Dashboard](https://dashboard.paystack.com/#/subaccounts) or [via the Paystack API](https://developers.paystack.co/v1.0/reference#create-subaccount).  When a subaccount is created, the `subaccount_code` and the `account_name` is returned. Please endeavor to verify that the account name matches what you intended. Paystack will not be liable for payouts to the wrong bank account.

2. Initialize a split payment: Split payments can be initialized by passing the parameter `subaccount: "SUB_ACCOUNTCODE"`.

   _For Paystack Inline_

   ```
   <form >
          <script src="https://js.paystack.co/v1/inline.js"></script>
          <button type="button" onclick="payWithPaystack()"> Pay </button> 
        </form>
         
        <script>
          function payWithPaystack(){
            var handler = PaystackPop.setup({
              key: 'pk_test_86d32aa1nV4l1da7120ce530f0b221c3cb97cbcc',
              email: 'customer@email.com',
              amount: 10000,
              .
              .
              .
              subaccount: 'ACCT_JIFIJJ',
              callback: function(response){
                  alert('success. transaction ref is ' + response.reference);
              },
              onClose: function(){
                  alert('window closed');
              }
            });
            handler.openIframe();
          }
        </script>
   ```

   _For Paystack Standard_

   ```
   curl https://api.paystack.co/transaction/initialize \
        -H "Authorization: Bearer SECRET_KEY" \
        -H "Content-Type: application/json" \
        -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com", "subaccount": "ACCT_JIFIJJ"}' \
        -X POST
   ```

## Other Information

### Flat Fee	

Payments are split on Paystack by percentage i.e. 20% going to main account and the rest going to subaccount. These parameters are required when creating the subaccount. However, there are instances where you will rather collect a flat fee per payment. To do this, pass a paramter called `transaction_charge = 1000 //amount in kobo`.

For example,

_For Paystack Inline_

```
<form >
       <script src="https://js.paystack.co/v1/inline.js"></script>
       <button type="button" onclick="payWithPaystack()"> Pay </button> 
     </form>
      
     <script>
       function payWithPaystack(){
         var handler = PaystackPop.setup({
           key: 'pk_test_86d32aa1nV4l1da7120ce530f0b221c3cb97cbcc',
           email: 'customer@email.com',
           amount: 10000,
           .
           .
           .
           subaccount: 'SUB_JIFIJJ',
           transaction_charge: 10000,
           callback: function(response){
               alert('success. transaction ref is ' + response.reference);
           },
           onClose: function(){
               alert('window closed');
           }
         });
         handler.openIframe();
       }
     </script>
```

_For Paystack Standard_

```
curl https://api.paystack.co/transaction/initialize \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com", "subaccount": "ACCT_JIFIJJ", "transaction_charge": 10000}' \
     -X POST
```

### Transaction Charges

You can use which party bears the Paystack charges when making a split payment between the main account and the subaccount. By default, the charges are borne by the main account. To change this to the subaccount, pass a parameter `bearer: "subaccount"` on intializing a transaction.

_For Paystack Inline_

```
<form >
       <script src="https://js.paystack.co/v1/inline.js"></script>
       <button type="button" onclick="payWithPaystack()"> Pay </button> 
     </form>
      
     <script>
       function payWithPaystack(){
         var handler = PaystackPop.setup({
           key: 'pk_test_86d32aa1nV4l1da7120ce530f0b221c3cb97cbcc',
           email: 'customer@email.com',
           amount: 10000,
           .
           .
           .
           subaccount: 'SUB_JIFIJJ',
           bearer: 'subaccount',
           callback: function(response){
               alert('success. transaction ref is ' + response.reference);
           },
           onClose: function(){
               alert('window closed');
           }
         });
         handler.openIframe();
       }
     </script>
```

_For Paystack Standard_

```
curl https://api.paystack.co/transaction/initialize \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com", "subaccount": "ACCT_JIFIJJ", "bearer": "subaccount",}' \
     -X POST
```

