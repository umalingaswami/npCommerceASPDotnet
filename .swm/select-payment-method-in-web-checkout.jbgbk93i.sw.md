---
title: Select Payment Method in Web Checkout
---
# Overview of Select Payment Method in Web Checkout

The Select Payment Method step in the web presentation checkout process allows customers to choose their preferred payment option before completing their purchase. This step is essential as it determines how the payment will be processed for the order.

# Purpose of Selecting a Payment Method

Selecting a payment method is a necessary part of the checkout workflow because it enables the system to collect and validate payment details. Without this selection, the checkout process cannot proceed to finalize the order, ensuring that no incomplete or unpaid orders are processed.

# Implementation in CheckoutController and CheckoutModelFactory

The functionality for selecting a payment method is implemented primarily in two components: the CheckoutController and the CheckoutModelFactory. The CheckoutModelFactory is responsible for preparing and providing the list of available payment methods to the checkout view, allowing customers to make their selection. Meanwhile, the CheckoutController handles the processing of the customer's choice, validating that a payment method has been selected before moving forward.

# Validation and Error Handling

To maintain the integrity of the checkout process, the system enforces validation to ensure a payment method is selected. If the customer attempts to proceed without choosing a payment method, the CheckoutController throws an exception with a clear message indicating that 'Payment method is not selected'. This prevents the order from advancing without necessary payment information.

# Data Flow During Payment Method Selection

When the checkout page loads, the CheckoutModelFactory fetches and compiles the list of available payment methods from the backend. This list is then rendered in the user interface for selection. Upon submission, the CheckoutController receives the selected payment method, validates it, and uses it to gather further payment details required to complete the transaction.

# Example Scenario

For example, if a customer reaches the payment step but does not select any payment method and tries to continue, the system will immediately halt the process and raise an exception. This ensures that the checkout cannot be completed without a valid payment method, thereby preventing potential errors or incomplete orders.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
