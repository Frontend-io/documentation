# Transfers

The funds transfers feature enables you send money directly from your Paystack balance to any Nigerian Bank account.

Payments are made from your Paystack balance so ensure your balance is funded by carrying out a Topup or [changing your settlement schedule](https://paystack.helpscoutdocs.com/article/32-difference-between-manual-and-next-day-paystack-payouts) to MANUAL.

Here is how to create a transfer:

1. [Create a transfer recipient](https://developers.paystack.co/reference#create-transfer-recipient) to make a transfer to. If the same bank details of an existing transfer recipient is passed, that recipient's recipient code is returned as opposed to a new recipient with the same details being created.
2. [Initiate a transfer](https://developers.paystack.co/reference#initiate-transfer).
3. If OTP has not been disabled, [finalise transfer](https://developers.paystack.co/v1.0/reference#finalize-transfer). OTPs are required to conclude a transfer by default but you can disable this requirement by calling the [Disable OTP requirement for Transfers](https://developers.paystack.co/reference#disable-otp-requirement-for-transfers) endpoint. An OTP will be sent to your phone to confirm the disable request. Use the [Finalize Disabling of OTP requirement for Transfers](https://developers.paystack.co/reference#finalize-disabling-of-otp-requirement-for-transfers) endpoint to submit this OTP and conclude the process. Future Transfers will not request OTP before starting.

