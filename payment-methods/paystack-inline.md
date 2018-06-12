# Paystack Inline

Paystack Inline offers a simple, secure and convenient payment flow for web. It can be integrated with a few lines of code thereby making it the easiest way to start accepting payments. It also makes it possible to start and end the payment flow on the same page using a modal, thus combating redirect fatigue.

## Implementation

To implement Paystack, collect user information such as email, amount, etc either from database, session or a HTML form and pass to the `payWithPaystack` function.

Here is a sample below:

```javascript
<form >
  <script src="https://js.paystack.co/v1/inline.js"></script>
  <button type="button" onclick="payWithPaystack()"> Pay </button> 
</form>
 
<script>
  function payWithPaystack(){
    var handler = PaystackPop.setup({
      key: 'pk_test_86d32aa1nV4l1da7120ce530f0b221c3cb97cbcc',
      email: 'customer@email.com',
      amount: 10000, //amount is passed in kobo. Multiply by 100 to pass a Naira amount
      ref: ''+Math.floor((Math.random() * 1000000000) + 1), // generates a pseudo-unique reference. Please replace with a reference you generated. Or remove the line entirely so our API will generate one for you
      metadata: {
         custom_fields: [
            {
                display_name: "Mobile Number",
                variable_name: "mobile_number",
                value: "+2348012345678"
            }
         ]
      },
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

A lazy way to load Paystack Inline is to pass the parameters as data attributes in a script tag as shown below:

```javascript
<form action="/process" method="POST" >
  <script
    src="https://js.paystack.co/v1/inline.js" 
    data-key="pk_test_221221122121"
    data-email="customer@email.com"
    data-amount="10000"
    data-ref=<UNIQUE TRANSACTION REFERENCE>
  >
  </script>
</form>
```

When the user enters their card details, Paystack will validate the card, charge the card, and pass a `response` object (containing the status of the transaction including reference as `reference`) to your callback function. If no callback function is set, we will insert a hidden field named `paystack-reference` on the parent form and submit the form to whatever action you set.

To confirm success of transaction, set your callback function to be a script that uses the verify endpoint on the Paystack API to check status of transaction.

More about this in **Verifying Transactions**.

### Parameters

`key`* - Your [Paystack Public Key](https://dashboard.paystack.com/#/settings/developer). Use test Public Keys for testing and live Public Keys for live transactions.

`email`* - Email address of customer. If you do not collect customer emails, use another parameter that can be used to identify customer. For example, if you have customers sign up with phone numbers, use 08030000000@mybusinessname.com if customer's number is 08030000000.

`amount`* - Amount (in kobo) you are debiting customer. Do not pass this if creating subscriptions.

`ref` - Unique case sensitive transaction reference. Only `-`,`.`, `=`and alphanumeric characters allowed. If you do not pass this parameter, Paystack will generate a unique reference for you.

`metadata` - Object containing any extra information you want recorded with the transaction. Fields within the `custom_field` object will show up on customer receipt and within the transaction information on the Paystack Dashboard.

`callback` - Function that runs when payment is successful. This should ideally be a script that uses the verify endpoint on the Paystack API to check status of transaction.

`onClose` - Javascript function that is called if the customer closes the payment window instead of making a payment.

`currency` - Currency charge should be performed in. Default is NGN.

`channels` - An array of payment channels, `['card']`, `[ 'bank']` or `['card','bank']` to control what channels you want to make available to the user to make a payment with.

`label`: String that replaces customer email as shown on the checkout form

`data-custom-button` - ID of custom button if you do not want to use the default Paystack button. To be used only if you are using the latter (lazy) method of loading Paystack using script tags.

*For Subscriptions*

`plan`* - Plan code generated from creating a plan. This makes the payment to be a subscription payment.

`quantity` - Used to apply a multiple to the amount returned by the plan code above.

*For Split Payment*

`subaccount`* - The code for the subaccount that owns the payment. e.g. `ACCT_8f4s1eq7ml6rlzj`

`transaction_charge` - A flat fee to charge the subaccount for this transaction**, in kobo**. This overrides the split percentage set when the subaccount was created. Ideally, you will need to use this if you are splitting in flat rates (since subaccount creation only allows for percentage split). e.g. `7000` for a 70 naira flat fee.

`bearer` - Decide who will bear Paystack transaction charges between `account` and `subaccount`. Defaults to `account`.

