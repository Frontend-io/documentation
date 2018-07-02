# Payment Pages

Payment Pages are the simplest way to collect payment through Paystack. 

With Payment Pages, you can generate a link that allows customers to make one-time or recurring payments. You can send this link to customers via WhatsApp, email Twitter DM, or any other digital medium of your choice.

Payment Pages can be created [right from the Paystack Dashboard](https://dashboard.paystack.com/#/pages) or [via the Paystack API](https://developers.paystack.co/v1.0/reference#create-page).

## Other Information

Once you have generated a payment page link like https://paystack.com/pay/sample-page, you can provide query parameters to prefill in the provided fields on the payment page.

For example, https://paystack.com/pay/sample-page?first_name=John&last_name=Doe will prefill first name with "John" and last name with "Doe".

### Parameters

- `first_name` - prefills the first name field
- `last_name` - prefills the last name field
- `email` - prefills the email address field
- `amount` - prefills the amount field if amount has not been fixed

You can also prefill custom fields. Parameter name is gotten by passing the field name and replacing spaces with an underscore `_`. if you have a custom field `Street Address`, the corresponding parameter is: `street_address`.

