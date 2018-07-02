# Charge

The Paystack charge payment method allows merchants to accept payments on the various Paystack payment channels (card, bank, USSD, etc) directly i.e. without having to interact with the Paystack Checkout form.

On other payment methods - Paystack Popup and Redirect - customers are shown a form designed by Paystack which incorporates all available Paystack channels. The charge endpoint however allows merchants to define their method of collection for each channel while Paystack does the processing on the backend.

This is achieved by making a POST request to the charge endpoint of the Paystack API.