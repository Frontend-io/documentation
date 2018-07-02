# Using Popup

Paystack Pop offers a simple, secure and convenient payment flow for web. It can be integrated with a few lines of code thereby making it the easiest way to start accepting payments. It also makes it possible to start and end the payment flow on the same page using a modal, thus combating redirect fatigue.

## Implementation

To implement Paystack Pop, follow the following steps

- [Step 1: Collect customer information](https://developers.paystack.co/v2.0/docs/paystack-popup#section-step-1-collect-customer-information)
- [Step 2: Initialize transaction](https://developers.paystack.co/v2.0/docs/paystack-popup#section-step-2-initialize-transaction)
- [Step 3: Implement transaction callback.](https://developers.paystack.co/v2.0/docs/paystack-popup#section-step-3-implement-transaction-callback-)
- [Step 4: Verify transaction](https://developers.paystack.co/v2.0/docs/verifying-transaction)
- Step 5: Handle webhook

### Collect customer information
To charge the customer, first collect information such as email, amount, etc. Email and amount are required for a transaction to occur.

<p class="callout info"><b>Tip:</b>
If you don’t collect email addresses from your customers, you can generate an email address by combining the information you have with your website url. For example, if you only collect phone numbers from your customers, you can create email addresses like 08030000000@mybusiness.com.</p>

You can pass this information from a database or user session if customers are already signed up. Otherwise you can display a HTML form which collects user information to initialize the transaction.

For example:

```
<form id="paymentForm">
  <div class="form-group">
    <label for="email">Email Address</label>
    <input type="email" id="email-address" required />
    <br>
  </div>  
  <div class="form-group">
    <label for="amount">Amount</label>
    <input type="tel" id="amount" required />
    <br>
  </div>  
  <div class="form-group">
    <label for="first-name">First Name</label>
    <input type="text" id="first-name" />
    <br>
  </div>  
  <div class="form-group">
    <label for="last-name">Last Name</label>
    <input type="text" id="last-name" />
    <br>
  </div>  
  <div class="form-submit">
    <button type="submit"> Pay </button> 
  </div>
</form>

<script src="https://js.paystack.co/v2/popup.js"></script>
```
### Initialize transaction
When you are ready to charge the user, call the `payWithPaystack` function, passing in the `email`, `amount` any other information of user that you need to pass to Paystack.
```
var paymentForm = document.getElementById('paymentForm');
paymentForm.addEventListener("submit", payWithPaystack, false);

function payWithPaystack(e) {  
  e.preventDefault();
  var config = {
    key: 'pk_live_xxxxxxxxxxxxxxxxxxxxxxxxx', // Replace with your public key
    email: document.getElementById("email-address").value,
    amount: document.getElementById("amount").value * 100,
    firstname: document.getElementById("first-name").value,
    lastname: document.getElementById("first-name").value,
    ref: ''+Math.floor((Math.random() * 1000000000) + 1), // generates a pseudo-unique reference. Please replace with a reference you generated. Or remove the line entirely so our API will generate one for you
    // label: "Optional string that replaces customer email"
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
  A lazy way to load Paystack Popup is to pass the parameters as data attributes in a script tag. Paystack will automatically create a payment button that users can click to initialize a transaction.
  ```<form action="/process" method="POST" >
  <script
    src="https://js.paystack.co/v1/inline.js" 
    data-key="pk_live_xxxxxxxxxxxxxxxxxxxxxxxxx"
    data-email="customer@email.com"
    data-amount="10000"
    data-ref=<UNIQUE TRANSACTION REFERENCE>
  >
  </script>
</form>
```
When the user enters their card details, Paystack will validate the card, charge the card, and pass a `response` object (containing the status of the transaction including reference as `reference`) to your callback function. If no callback function is set, we will insert a hidden field named `paystack-reference` on the parent form and submit the form to whatever action you set.

### Implement transaction callback.
The callback function is what runs when the transaction is successful. Any action you want your service to carry out after payment should be included in this function.

This action is typically verification of transaction to confirm the status. To verify the transaction, set a script in your callback function that calls your server. When your server is called it should use the Paystack verify endpoint to check the status of the transaction and return a response.

You can call your server in two ways. One method is to redirect to the server by setting a window.location to the URL where the verification endpoint is set on your server.

```
// On the  callback function, you can redirect to another page, passing the transaction reference 
callback: function(response){
  window.location = "http://www.yoururl.com/file.php?reference=" + response.reference;
}

//So on that redirected page, you can run verification. 
```
Another method if you are familiar with Ajax, would be to do a POST call that sends transaction reference to your verification script on your server.

```
//Paystack inline callback function
// Using Ajax to verify the transaction
callback: function(response){
  $.ajax({
    url: 'http://www.yoururl.com/file.php?reference=' + response.reference,
    method: 'post',
    success: function (data) {
      //Do whatever you like
    }
  });
}
```
## Parameters

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