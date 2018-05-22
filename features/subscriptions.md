# Subscriptions

Subscriptions allow customers to pay a specific amount every hour, day, week, month, or year depending on the recurring interval you set. With recurring payments on Paystack, the customer enters their card details once, and they're dutifully debited without them having to input their card details in future.

The steps for creating a subscription for a customer are:

1. [Create a plan](https://developers.paystack.co/v1.0/reference#create-plan): A **plan** is a framework for a subscription. This is where you decide the amount, interval of the subscription and currency in which the subscription will be paid. Every subscription needs to be on a plan.

2. Create a subscription: Subscriptions can be created in two ways. 

   - The most popular method is to create a subscription by simply passing in a plan code as a parameter. 

     _In Paystack Inline_

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
           amount: 10000,
           .
           .
           .
           plan: 'PLN_JIFIJJ',
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

     _In Paystack Standard_

     ```curl
     curl https://api.paystack.co/transaction/initialize \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"reference": "7PVGX8MEk85tgeEpVDtD", "amount": 500000, "email": "customer@email.com", "plan": "PLN_JIFIJJ"}' \
     -X POST
     ```

   - A second way to create subscriptions is to run the [create subscription endpoint](https://developers.paystack.co/v1.0/reference#create-subscription) via the Paystack API.

     ```
     curl https://api.paystack.co/subscription \
     -H "Authorization: Bearer SECRET_KEY" \
     -H "Content-Type: application/json" \
     -d '{"customer": "CUS_xnxdt6s1zg1f4nx", "plan": "PLN_gx2wn530m0i3w3m"}' \
     -X POST
     ```

     ## Other Information

   - Subscriptions are not retried. When a payment attempt fails, it will not be attempted again. This makes subscriptions ideal for situations where value is delivered after payment such as payment for internet service or a streaming service. Where if payment fails, value is held till customer pays successfully via a one-time payment.

   - Subscriptions are disabled via the [Disable Subscription](https://developers.paystack.co/v1.0/reference#disable-subscription) endpoint. Customers are also sent emails before a subscription is charged to allow them cancel if they so wish.

