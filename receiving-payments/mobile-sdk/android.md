# Android

The Paystack Android SDK allows businesses using Paystack to collect payments within their Android app.

The SDK shoulders the burden of PCI compliance, allowing your app to send card information directly to Paystack servers where we can process them. All your server needs to do is call Paystack to verify the status of the transaction. The SDK also validates card details client side and provides an interface for collecting additional verification like card PIN and OTP.

We support Android back to version 16, which is the first SDK version that includes `TLSv1.2` which is required by our servers. Native app support for user devices older than API 16 will not be available.

## Installation

To install the library on [Android Studio](http://developer.android.com/tools/studio/index.html) or [IntelliJ](https://www.jetbrains.com/help/idea/2016.3/getting-started-with-android-development.html), add the following lines to your project's `build.gradle` file.

```
dependencies {
    compile 'co.paystack.android:paystack:3.0.10'
}
```

To install on [Eclipse](https://www.eclipse.org/),

- Clone the [Paystack Android repository](https://github.com/PaystackHQ/paystack-android).
- Import the **paystack** folder into your [Eclipse](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.user/tasks/tasks-importproject.htm) project
- In your project settings, add the **paystack** project under the Libraries section of the Android category.

## Configuration

### Enable Permissions

To prepare for use, you must ensure that your app has internet permissions by making sure the `uses-permission` line below is present in the AndroidManifest.xml.

```
<uses-permission android:name="android.permission.INTERNET" />
```

### Set Your Public Key

Before you can charge a card with the `PaystackSdk` class, you need to set your public key. Your public key is available on the [Paystack Dashboard](https://dashboard.paystack.com/#/settings/developer). There are two methods of accomplishing this:

- Setting your public key in your project's AndroidManifest.xml.
- Setting the public key in your code

#### Using AndroidManifest.xml

Add the following lines to the `<application></application>` tag of your AndroidManifest.xml

```
<meta-data
    android:name="co.paystack.android.PublicKey"
    android:value="your public key obtained from: https://dashboard.paystack.co/#/settings/developer"/>
```

#### Setting the Public Key in your code

This can be done anytime in your code. Just be sure to initialize before calling `chargeCard`.

```
class Bootstrap {
    public static void setPaystackKey(String publicKey) {
        PaystackSdk.setPublicKey(publicKey);
    }
}
```

### Initialize SDK

To use the Paystack Android SDK, you need to first initialize it using the `PaystackSdk` class.

```
public class App extends Application{
    @Override
    public void onCreate() {
        super.onCreate();

        PaystackSdk.initialize(getApplicationContext());
    }
}
```

Make sure to call this method in the `onCreate` method of your Fragment or Activity or Application.

## Collecting card information

Create a form that collects card information. You need to collect your customers’ card numbers, expiration dates and CVC. Our `Card` class allows you collect and verify these. The library provides validation methods to validate the fields of the card.

- card.validNumber - This method helps to perform a check if the card number is valid.
- card.validCVC - Method that checks if the card security code is valid.
- card.validExpiryDate - Method checks if the expiry date (combination of year and month) is valid.
- card.isValid - Method to check if the card is valid. Always do this check, before charging the card.
- card.getType - This method returns an estimate of the string representation of the card type.

Add this code to your MainActivity.java file

```
public class MainActivity extends AppCompatActivity {

  // This sets up the card and check for validity
  // This is a test card from paystack
   String cardNumber = "4084084084084081";
   int expiryMonth = 11; //any month in the future
   int expiryYear = 18; // any year in the future. '2018' would work also! 
   String cvv = "408";  // cvv of the test card
   
   Card card = new Card(cardNumber, expiryMonth, expiryYear, cvv);
    if (card.isValid()) {
       // charge card
    } else {
      //do something
    }
}
```

## Charge User

There are two ways to charge a card using the Android SDK:

- Transaction is initialized from server
- Transaction is started by app

### Transaction initialized from server

This method uses an access code which is returned as a response when the initialize endpoint is called. Run this code on your server and pass the access code to the app.

```
curl https://api.paystack.co/transaction/initialize \
-H "Authorization: Bearer SECRET_KEY" \
-H "Content-Type: application/json" \
-d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com"}' \
-X POST
```
This can be used to charge the card by doing`charge.setAccessCode({value from backend})`. Once an access code is set, the only other parameter relevant to the transaction is the card. Others will be ignored.

### Transaction started by app

Using the functions: `setCurrency`, `setPlan`, `setSubaccount`, `setTransactionCharge`, `setAmount`, `setEmail`, `setReference`, `setBearer`, `putMetadata`, `putCustomField`, you can set up a fresh transaction direct from the SDK. Documentation for these parameters are same as for initializing a transaction.

## Transaction Callback

When an error occurs or transaction or concludes successfully, we will call the methods available in the callback you provided.

- `OnSuccess` will be called once the charge succeeds.
- `beforeValidate` is called every time the SDK needs to request user input. This function currently only allows the app know that the SDK is requesting further user input.OnError
- `onError` is called if an error occurred during processing. Some Exception types that you should watch include
  - *ExpiredAccessCodeException*: This would be thrown if the access code has already been used to attempt a charge.
  - *ChargeException*: This would be thrown if the charge failed. It would hold the message from the server.

```
public class MainActivity extends AppCompatActivity {


  // This is the subroutine you will call after creating the charge
  // adding a card and setting the access_code
   public void performCharge(){
         //create a Charge object
         Charge charge = new Charge(); 
         charge.setCard(card); //sets the card to charge
   
       PaystackSdk.chargeCard(MainActivity.this, charge, new Paystack.TransactionCallback() {
           @Override
           public void onSuccess(Transaction transaction) {
               // This is called only after transaction is deemed successful.
               // Retrieve the transaction, and send its reference to your server
               // for verification.
           }

           @Override
           public void beforeValidate(Transaction transaction) {
               // This is called only before requesting OTP.
               // Save reference so you may send to server. If
               // error occurs with OTP, you should still verify on server.
           }

           @Override
           public void onError(Throwable error, Transaction transaction) {
             //handle error here
           }

       });
   }
}
```

Note that once `chargeCard` is called, depending on settings agreed with Paystack's Customer Success team, the SDK *may* prompt the user to provide their PIN, an OTP or conclude Bank Authentication. These are currently being managed entirely by the SDK. Your app will only be notified via the `beforeValidate` function of the callback when OTP or Bank Authentication is about to start.

## Verifying the transaction

Send the reference to your backend and verify by calling our REST API. An authorization will be returned which will let you know if its code is reusable. You can learn more about our verify call [here](https://developers.paystack.co/reference#verifying-transactions).

Below is a sample authorization object returned along with the transaction details:

```
   {
     "status": true,
     "message": "Verification successful",
     "data": {
       "amount": 10000,
       "currency": "NGN",
       "transaction_date": "2017-04-06T21:28:41.000Z",
       "status": "success",
       "reference": "d68rbovh4a",
       "domain": "live",
       "metadata": {
         "custom_fields": [
           {
             "display_name": "Started From",
             "variable_name": "started_from",
             "value": "sample charge card backend"
           },
           {
             "display_name": "Requested by",
             "variable_name": "requested_by",
             "value": "some person"
           },
           {
             "display_name": "Server",
             "variable_name": "server",
             "value": "some.herokuapp.com"
           }
         ]
       },
       "gateway_response": "Approved",
       "message": "Approved",
       "channel": "card",
       "ip_address": "41.31.21.11",
       "log": null,
       "fees": 150,
       "authorization": {
         "authorization_code": "AUTH_blahblah",
         "bin": "412345",
         "last4": "6789",
         "exp_month": "10",
         "exp_year": "2345",
         "channel": "card",
         "card_type": "mastercard debit",
         "bank": "Some Bank",
         "country_code": "NG",
         "brand": "mastercard",
         "reusable": true,
         "signature": "SIG_IJOJidkpd0293undjd"
       },
       "customer": {
         "id": 22421,
         "first_name": "Guava",
         "last_name": "Juice",
         "email": "guava@juice.me",
         "customer_code": "CUS_6t6che6w8hmt",
         "phone": "",
         "metadata": {},
         "risk_action": "default"
       },
       "plan": null
     }
   }
```