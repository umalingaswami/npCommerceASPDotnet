---
title: Checkout Payment Overview
---
# Overview of Checkout Payment

Checkout Payment is the process that manages payment information and transactions during the checkout phase of an order. It ensures that the payment details provided by the customer are properly validated and processed to successfully complete the purchase.

# Core Components Involved

Several key components collaborate to handle checkout payments effectively. The PaymentService manages available payment methods, processes payment requests, and handles actions after payment completion. The OrderProcessingService calculates any additional fees related to the chosen payment method, including applicable taxes, ensuring the final order total is accurate.

Meanwhile, the CheckoutController orchestrates user interactions during the payment phase of checkout. It manages the flow of the payment process, including error handling and localization of messages to provide clear and user-friendly feedback. The CheckoutModelFactory prepares the data models used in the checkout views, incorporating warnings and messages related to payment and order totals to keep customers informed.

# Payment Processing Flow

During checkout, when a customer submits payment information, the PaymentService validates the selected payment method. If the payment method cannot be loaded or is invalid, the service throws an exception to halt the process, maintaining the integrity of the transaction. After validation, the OrderProcessingService calculates any additional fees or taxes associated with the payment method, updating the order total accordingly.

The CheckoutController then updates the user interface based on the outcome, displaying any warnings or errors through models prepared by the CheckoutModelFactory. This ensures that customers receive immediate and clear feedback about their payment status and any additional charges.

# Example Scenario

For example, if a customer selects a payment method that is no longer available or improperly configured, the PaymentService will detect this during processing and throw an exception. This prevents the checkout from proceeding with an invalid payment method, prompting the CheckoutController to display an appropriate error message localized to the customer's language.

This mechanism ensures robustness and reliability in the payment process, preventing incomplete or erroneous transactions.

# Summary

In summary, the checkout payment process in this platform integrates multiple services and controllers to validate payment methods, calculate fees, and provide user feedback. This integration ensures a smooth and secure payment experience during checkout.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
