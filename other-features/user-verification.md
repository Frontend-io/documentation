# User Verification

Paystack provides a number of tools that help you verify user information. Here's a list of identification and verification tools that Paystack currently provides:

1. BVN Verification
2. BVN - Account Number match
3. Resolve Bank Account Number 
4. Resolve Card BIN
5. Phone number verification

##BVN Verification

We provide a way for you to verify a user's identity using their Nigerian Bank Verification Number. Users supplies their BVN number and you call [BVN Resolve endpoint](https://developers.paystack.co/v1.0/reference#resolve-bvn) with it. 

Here is a sample request:

```
curl "https://api.paystack.co/bank/resolve_bvn/USERS_BVN" \
-H "Authorization: Bearer YOUR_SECRET_KEY" 
-X GET
```

It returns the following information about the user: 

- First name
- Last name
- Phone number
- Date of birth
- BVN number

To verify that user, you can withhold the information returned, then send an OTP to the BVN phone number which they can enter into your site or app to confirm they own the BVN. You can also prompt for their date of birth and confirm if it matches with the one returned with the BVN.

This service cost N10 per call, but you get 10 free calls per month.

##BVN - Account Boolean Match

 [This endpoint](https://developers.paystack.co/v1.0/reference#match-bvn) takes a user's account details and their BVN and tells you whether the account is linked to the BVN. This verification is very useful if you want to be sure that funds are only disbursed to a particular user's account.

Here is a sample code:

```
curl -X GET \
  'https://api.paystack.co/bank/match_bvn?account_number=0026733598&bank_code=058&bvn=22248185000' \
  -H 'Authorization: Bearer YOUR_SECRET_KEY' \
  -H 'Cache-Control: no-cache' \
  -H 'Postman-Token: d01ebe5e-e12e-5d34-84f5-1cd54c8e9eb9'
```

[You can get the list of banks and their bank codes using this endpoint](https://developers.paystack.co/v1.0/reference#list-banks).

This service cost N10 per call, you also get 10 free calls per month.

## Resolve Bank Account Number

This endpoint takes an account number and bank code and returns the user's account name. With this, both you and your user can confirm that they are inputting the right account details.

```
curl "https://api.paystack.co/bank/resolve?account_number=0022728151&bank_code=063" \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-X GET
```

[You can get the list of banks and their bank codes using this endpoint](https://developers.paystack.co/v1.0/reference#list-banks).

This service is free.

## Resolve Card Bin

This endpoint takes the first 6 digits of a card PAN and returns information about the card.

```
curl "https://api.paystack.co/decision/bin/539983" \
-H "Authorization: Bearer YOUR_SECRET_KEY" \
-X GET
```

Details returned include the card brand (Mastercard, Visa, etc), card type (credit, debit), country name, country code and bank.

This service is free

## Phone Number Verification

In collaboration with Truecaller, Paystack has an endpoint that you can be used to verify user's phone numbers.