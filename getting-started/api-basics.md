# API Basics
The Paystack API gives you access to pretty much all the features you can use on our dashboard and lets you extend them for use in your application. It strives to be RESTful and is organized around the main resources you would be interacting with - with a few notable exceptions.

Before you do anything, you should [create a free Paystack account](https://dashboard.paystack.co/#/signup) that you can test the API against. We will provide you with test keys that you can use to make API calls.

## Sample Requests
We provide sample API calls next to each method using [cURL](http://curl.haxx.se/). All you need to do is drop in your specific parameters, and you can test the calls from the command line. See this [tutorial on using cURL with APIs](http://kit.com/resources/HTTP-from-the-Command-Line/). 

Another easy way to test APIs is to use [Postman](https://www.getpostman.com/apps) - a tool made for developers to test APIs via a GUI interface. We have a Postman collection [here](https://documenter.getpostman.com/view/2770716/paystack-api/7187aMn) to make this easy.

## Authentication
All requests to Paystack are authenticated using your API keys. We provide a **secret key** and a **public key**. The **public key** is only used to initialize transactions on your **frontend** Javascript and mobile apps while the **secret key** is used to make all API calls to Paystack from your server. 

We provide 2 sets of keys - the **test** keys and the **live** keys. The **test** keys are used for test transactions while you are still in development, the **live** keys are used for live transactions. The API keys have the key type and domain appended in them so you can identify them. For example, your **live** public key will look like **pk_live_xxxxxxxxxxxxxxxx** and your **test** secret key will look like **sk_test_xxxxxxxxxxxxxxxxx**.

[You can find your API keys here on the Developer/API section of your Paystack Dashboard Settings](http://dashboard.paystack.com/#/settings/developer).

<p class="callout danger"><b>Secret Keys</b><br />
Do not commit your secret keys to git, or use them in client-side code.</p>

Authorization headers should be in the following format: `Authorization: Bearer SECRET_KEY`

API requests made without authentication will fail with the status code `401: Unauthorized`

All API requests must be made over HTTPS.

## Input/Output Format
Both request body data and response data are formatted as JSON. Content type for responses will always be `application/json`.

Generally, all responses will be in the following format:

<p class="callout danger"><b>Standard API response format</b><br />
{
          "status": [boolean] - Only true if the details provided could be processed and no error occured while processing,
          "message": [string] - Explains why status is false... Entirely informational. Please only log this but do not use for your checks,
          "data": [object] - contains actionable result of processing if present
}
</p>


While we generally recommend that developers use HTTP status codes to determine the result of an API call, we have provided a handy `status` key to let you know upfront if the request was successful or not.

The message key is a string which will contain a summary of the response and its status. For instance when trying to retrieve a list of customers, message might read “Customers retrieved”. In the event of an error, the message key will contain a description of the error as with the authorization header situation above. This is the only key that is universal across requests.

The data key is where you want to look at for the result of your request. It can either be an object, or an array depending on the request made. For instance, a request to retrieve a single customer will return a customer object in the data key, while the key would be an array of customers if a list is requested instead.

The meta key is used to provide context for the contents of the data key. For instance, if a list of transactions performed by a customer is being retrieved, pagination parameters can be passed along to limit the result set. The meta key will then contain an object with the following attributes

```
"meta": {
    "total": 2,
    "skipped": 0,
    "perPage": 50,
    "page": 1,
    "pageCount": 1
}
```

**Total** is a total number of transactions that were performed by the customer.
**Skipped** is the number of records skipped before the first record in the array returned. 
**PerPage** is the maximum number of records that will be returned per request. This can be modified by passing a  new value as a `perPage` query parameter. *Default: 50*
**Page** is the current `page` being returned. This is dependent on what page was requested using the `page` query parameter. *Default: 1*
**PageCount** is how many pages in total are available for retrieval considering the maximum records per page specified. For context, if there are 51 records and perPage is left at its default value, pageCount will have a value of 2.

In the event of a validation error when creating or updating an object, response format is slightly different and contains an error key instead of a data key.

This example is a response you will get when you try to create a customer without passing an email parameter, which happens to be required.

```
{
    "status": false,
    "message": "Validation error has occured",
    "errors": {
        "email": [
            {
                "rule": "required",
                "message": "Email is required"
            }
        ]
    }
}
```