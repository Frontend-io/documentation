# Receiving Money Quickstart

You can integrate Paystack into your application and start receiving payments with these easy steps.

1. Choose an integration method
2. Get your API keys
3. Initialize the transaction in your code
4. Verify the transaction and Deliver Value

##Step1 - Choose Integration Method

Paystack has provisions for various methods of integration.

*If you are setting up Paystack on CMS platforms like Wordpress or Joomla, or on e-commerce platforms like Shopify or Opencart, please [follow this link to learn how to setup Paystack on your site](https://paystack.helpscoutdocs.com/category/11-setting-up-paystack). If you do not have a website/app, follow this link to learn [how to use Paystack Payment Pages to start receiving payments](https://paystack.helpscoutdocs.com/article/51-paystack-payment-pages)*

- **Website (Javascript)**: The easiest way to integrate Paystack on your website is using our Javascript inline library. You simply include the Paystack Inline on your page, and link the Paystack setup function to you payment button. When the payment button is clicked, Paystack Checkout form pops where the user completes the payment. [Learn more about Paystack Inline here](https://developers.paystack.co/docs/paystack-inline). 

  Paystack also allows you design your own card checkout form using our PaystackJs library. You have complete control of the look and feel of the card checkout form and the Paystack library handles the secure transfer of the card details and completion of the payment. [Learn more about PaystackJS here](https://developers.paystack.co/docs/paystackjs).

- **Server**: Paystack is completely RESTful API based, so can call Paystack directly from your server. Paystack has official libraries for different programming languages and frameworks. You can [install the Paystack library for the programming language](https://developers.paystack.co/docs/libraries-and-plugins) of your server or you can m[ake http calls directly to the API endpoints](https://developers.paystack.co/v1.0/reference#initialize-a-transaction). 

- **Mobile App**: You can integrate Paystack into your Android or iOS app using our SDKs. Here is a link to [Quickstart on Integrating Paystack On Mobile Apps](https://developers.paystack.co/docs/mobile-quickstart). You can get the [Android SDK here](https://github.com/PaystackHQ/paystack-android) and you can [get the iOS SDK here](https://github.com/PaystackHQ/paystack-ios). For mobile frameworks like Ionic or React Native, [please find the libraries here](https://developers.paystack.co/docs/libraries-and-plugins).

## Step 2 - Get your API keys

All requests to Paystack are authenticated using your API keys. We provide a **secret key** and a **public key**. The **public key** is only used to initialize transactions on your **frontend** Javascript and mobile apps while the **secret key** is used to make all API calls to Paystack from your server. 

We provide 2 sets of keys - the **test** keys and the **live** keys. The **test** keys are used for test transactions while you are still in development, the **live** keys are used for live transactions. The API keys have the key type and domain appended in them so you can identify them. For example, your **live** public key will look like **pk_live_xxxxxxxxxxxxxxxx** and your **test** secret key will look like **sk_test_xxxxxxxxxxxxxxxxx**.

[You can find your API keys here on the Developer/API section of your Paystack Dashboard Settings](http://dashboard.paystack.com/#/settings/developer).

##Step 3 - Initialize Transaction

When you are ready to charge the user, you send the transaction parameters to Paystack to initialize the transaction. 

These parameters are set in the **Transaction Initialization Object** passed to the API or the Javascript initialization methods. [Learn about all the transaction initialization parameters here](https://developers.paystack.co/v1.0/reference#initialize-a-transaction).

**CURL:** [Try it here](https://developers.paystack.co/v1.0/reference#initialize-a-transaction)

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"amount": 500000, "email": "customer@email.com"}' \
-X POST
```

**PHP:**

```
<?php
// Initialize as required by your installation
// Download library from https://github.com/yabacon/paystack-php
$paystack = new Paystack('YOUR_SECRET_KEY');
$responseObj = $paystack->transaction->initialize([
     "reference"=>"7PVGX8MEk85tgeEpVDtD",
     "amount"=>500000,
     "email"=>"customer@email.com",
]);
```

**Inline:**

```
<form >
  <script src="https://js.paystack.co/v1/inline.js"></script>
  <button type="button" onclick="payWithPaystack()"> Pay </button> 
</form>
 
<script>
  function payWithPaystack(){
    var handler = PaystackPop.setup({
      key: 'YOUR_PUBLIC_KEY',
      email: 'customer@email.com',
      amount: 10000,
      ref: ''+Math.floor((Math.random() * 1000000000) + 1),
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

**PaystackJS:**

```
import { Transaction, Card } from 'paystack-js';

const transactionData = {
  email: 'customer@email.com',
  amount: 100, // amount in kobo
  key: 'replace_with_your_public_key',
};

// Request a new transaction
const transactionResponse = await Transaction.request(transactionData); 

// Use the response to create a transaction instance
const transaction = new Transaction(transactionResponse);

// Create a payment method instance that will be used e.g card
const card = new Card({ 
  number: '4084084084084081', 
  cvv: '408', 
  month: '12', 
  year: '20', 
});

// Payment method instances usually provide validation functions
if (card.isValid()) {
  // Set the payment method on the transaction
  transaction.setPaymentMethod(Transaction.PaymentMethods.CARD, card);
}

// Charge the payment method
const chargeResponse = await transaction.card.charge();

// Handle the charge responses
if (chargeResponse.status === 'success') {
  alert('Payment completed!');
}

// Another charge response example
if (chargeResponse.status === 'auth') {
  const token = 123456;
  const authenticationResponse = await transaction.card.authenticate(token);
  if (authenticationResponse.status === 'success') {
    alert('Payment completed!');
  }
}
```

Notice that the server side implementations all required secret key, and the Javascript implementations all required the public key. 

After the transaction is initialized, Paystack checkout form is loaded so the user can complete the transaction. The next step after the user pays is to verify the transaction.

##Step 4 - Verify Transaction and Deliver Value

After the user has paid, you should make an API call from your server to Paystack's `verify` endpoint using the transaction reference to confirm that the transaction was completed successfully. 

[Here is the curl command to verify transaction:](https://developers.paystack.co/v1.0/reference#verify-transaction)

```
curl https://api.paystack.co/transaction/verify/YOUR_TRANSACTION_REFERENCE \
-H "Authorization: Bearer SECRET_KEY"
```

The response contains a `data` object which contains the transaction status in `data->status`. 

The `data` object also contains an object called `authorization` which has the `authorization_code` for the payment instrument the customer paid with which you can store against the user to be used to charge the users subsequently. Learn more at [Recurring Payment Quickstart](https://developers.paystack.co/docs/recurring-quickstart).

Once you verify that the payment was successful you can now deliver value.

##Next Steps

- [Deep Dive into Receiving Payments with Paystack](https://developers.paystack.co/docs/guides/receiving-payments)
- [Charging from your backend](https://developers.paystack.co/docs/guides/server-guide)
- [Initialize Transaction API Reference](https://developers.paystack.co/v1.0/reference#initialize-a-transaction)
- [Deep dive into charging from your mobile app](htttps://developers.paystack.co/docs/guides/mobile)
- [Receiving payment notifications on your server](https://developers.paystack.co/v1.0/docs/events)

