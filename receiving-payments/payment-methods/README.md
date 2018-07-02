# Payment Methods

**Before you begin**

To receive payments on Paystack, you need API keys to authenticate your payments. You can find your keys on the Paystack Dashboard under [Settings → API Keys & Webhooks](https://dashboard.paystack.com/#/settings/developer). To learn more about API keys, click here.

Paystack provides a best in class checkout form for you to receive payments for your business. You can implement a checkout experience for your users in any of the following payment methods:

## Popup
This is the simplest way to collect payments on Paystack. Include this javascript library in your frontend code to load a checkout form designed by Paystack. This requires use of your public key.

_HTML_
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
    <button type="submit" onclick="payWithPaystack()"> Pay </button> 
  </div>
</form>

<script src="https://js.paystack.co/v2/popup.js"></script>
```

_Javascript_
```
var paymentForm = document.getElementById('paymentForm');
paymentForm.addEventListener("submit", payWithPaystack, false);

function payWithPaystack(e) {  
  e.preventDefault();
  var config = {
    key: 'pk_test_448800889e222223b1407c1bae6e57b612aeb8f0', // Replace with your public key
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
}
```

## Generate a link using the API

The Paystack API [provides an endpoint](https://developers.paystack.co/v2.0/docs/paystack-redirect) `POST transaction/initialize` to generate a payment link that you can share with your customers. There are libraries available for different programming languages and frameworks which simplify its use. You can install the Paystack library for the programming language of your server or you can make http calls directly to the API endpoints.

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com"}' \
-X POST
```
## Mobile SDK

You can integrate Paystack directly into your [Android](https://developers.paystack.co/v2.0/docs/android) or [iOS](https://developers.paystack.co/v2.0/docs/ios) app using our mobile SDK. For mobile frameworks like Ionic or React Native, please find the libraries [here](https://developers.paystack.co/v2.0/docs/libraries-and-plugins).

## Build a custom checkout
[Paystack.js](https://developers.paystack.co/v2.0/docs/paystack-custom) allows you to completely control the look and feel of your form. You build your own checkout form while we handle the processing.

## Plugins

If you use Shopify, WordPress, Magento, Joomla, Drupai or any popular Content Management System, plugins make it real easy to accept payments. See full list of our plugins [here](https://developers.paystack.co/v2.0/docs/plugins).