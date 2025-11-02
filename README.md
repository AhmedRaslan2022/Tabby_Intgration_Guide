# Tabby Integration Handbook: 

This guide outlines the backend steps for integrating the Tabby payment gateway.

> **📚 Highly Recommended:** Before starting, please review the [General Payment Integration Handbook](https://github.com/AhmedRaslan2022/payment-integration-handbook/tree/main) to understand the complete payment flow and core concepts like webhooks.

-----

## 1\. 📖 Review Documentation

Start by carefully reading the official Tabby API documentation to understand all available endpoints, request/response structures, and authentication methods.

  * **Official Tabby Docs:** [https://docs.tabby.ai/](https://www.google.com/search?q=https://docs.tabby.ai/)
  * **API Reference:** [https://docs.tabby.ai/api-reference/](https://www.google.com/search?q=https://docs.tabby.ai/api-reference/)

-----

## 2\. 🔗 Register Your Webhook

Register your webhook endpoint to receive asynchronous updates from Tabby (e.g., payment status changes).

  * **API Doc:** [Create a Session (for webhook registration info)](https://docs.tabby.ai/api-reference/checkout/create-a-session)
  * If you are unfamiliar with webhooks, review the [General Payment Integration Handbook](https://github.com/AhmedRaslan2022/payment-integration-handbook/tree/main).

 * After a successful registration call, you must call the GET Retrieve all webhooks endpoint. Check the response list to confirm that your new webhook URL is present. This verifies that the registration was successful.

> 💡 **Tip:** If the registration API request fails in Postman, try sending the request using cURL.

-----

## 3\. 🛠 Implement "Create Session" Endpoint

Implement the "Create Session" API call. This is the core endpoint to initiate a payment.

  * **API Doc:** [https://docs.tabby.ai/api-reference/checkout/create-a-session](https://docs.tabby.ai/api-reference/checkout/create-a-session)
  * **Ensure all required fields** are included in your request.
  * **Handle logic exceptions:** If a field is not compatible with your business logic (e.g., `shipping_address` for digital services), clarify this omission with the gateway integration team in your coordination email.

> 🔎 **Debugging Tip:** Use a tool like JsonGrid to facilitate debugging and quickly find any missing fields or data formatting errors.

-----

## 4\. 🔎 Implement Background Pre-Scoring Check

Before displaying Tabby as a payment option to the user, you must perform a background check to verify their eligibility.

1.  When the frontend requests available payment options (e.g., on the checkout page), send a **"Create Session" request** (identical to Step 3) to Tabby.
2.  This request checks the user's eligibility and whether the order amount is within the allowed limits for your merchant account.
3.  Based on the response:
      * **If approved:** Return Tabby as an available option to the frontend. Your response must include the Title, Description, and Logo URL provided in the docs. 
      * **If rejected:** Return Tabby as a not selectable option. You must also return the rejection reason, mapping it to the standard messages provided in the documentation (this allows the frontend to show why it's disabled).

<!-- end list -->

  * **API Doc:** [Background Pre-scoring Check](https://docs.tabby.ai/pay-in-4-custom-integration/checkout-flow#background-pre-scoring-check)

  * **Display Details Reference:**  For details on what display fields to return (Title, Description, Logo), refer to the Tabby on Checkout section: https://docs.tabby.ai/pay-in-4-custom-integration/checkout-flow#tabby-on-checkout
-----

## 5\. 🔀 Implement Redirection URLs

After the user attempts payment on Tabby's page, they will be redirected back to your site. You must create endpoints to handle these return URLs.

  * **Success URL:** Handles successful payments.
  * **Failure URL:** Handles failed payments.
  * **Cancel URL:** Handles user-canceled payments.

**Important:** Use the exact user-facing messages (in both Arabic and English) as specified in the documentation for each case.

  * **API Doc:** [Redirection to the Store](https://docs.tabby.ai/pay-in-4-custom-integration/checkout-flow)

-----

## 6\. 📢 Handle Webhook Events 

Implement the server logic to process asynchronous webhook notifications from Tabby. This is critical for confirming payment status independently of the user's redirection.

> **❗ CRITICAL:** You **must** respond to the webhook request with an **HTTP 200 OK** status code *immediately* before processing any business logic.
>
> Failure to do so will cause Tabby to assume the notification failed and repeatedly retry sending it.

  * Refer to the [General Payment Integration Handbook](https://github.com/AhmedRaslan2022/payment-integration-handbook/tree/main) for best practices on handling webhooks.
  * **API Doc:** [Webhooks](https://docs.tabby.ai/pay-in-4-custom-integration/webhooks)


-----

## 7\. Capture the Authorization

Call the capture endpoint to transfer the authorized amount to the merchant account once you get authorized in the webhook.

  * **API Doc:** [Capture]([https://docs.tabby.ai/pay-in-4-custom-integration/webhooks](https://docs.tabby.ai/api-reference/payments/capture-a-payment)


 -----


## 8\. 🌎 Handle Geographic Restrictions

  * If the application **only** operates in KSA (Saudi Arabia), ensure your logic does not return Tabby as a payment option for clients outside of KSA.
  * Alternatively, ensure other countries are removed during the user registration process.
