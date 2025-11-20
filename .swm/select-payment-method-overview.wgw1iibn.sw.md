---
title: Select Payment Method Overview
---
# Select Payment Method Overview

The Select Payment Method step is a critical part of the checkout process where customers choose their preferred payment option from the list of available methods. This choice directly impacts the order processing flow, including fee and tax calculations.

During checkout, the system presents all active and valid payment methods to the customer. The selection is then validated to ensure the chosen method is currently enabled and can be properly loaded by the system.

If no payment method is selected, or if the selected method is inactive or cannot be loaded, the system interrupts the checkout process by throwing specific exceptions. This validation prevents errors and ensures that only valid payment methods are used for order completion.

# Role in Checkout Workflow

The CheckoutController and CheckoutModelFactory are the primary components responsible for managing the payment method selection. The controller handles user input and triggers validation, while the model factory prepares the data needed to display payment options and reflect the selected method.

Once a payment method is selected and validated, services such as OrderTotalCalculationService and OrderProcessingService utilize this information to calculate any additional fees or taxes associated with the payment method. This ensures accurate order totals before finalizing the purchase.

# Validation and Exception Handling

Validation logic in the checkout process includes checks for the presence and status of the payment method. For example, if the payment method is missing, an exception with the message 'Payment method is not selected' is thrown. Similarly, if the method cannot be loaded or is inactive, exceptions like 'Payment method couldn't be loaded' or 'Payment method is not active' are raised.

These safeguards ensure that the checkout process cannot proceed with invalid or unavailable payment options, maintaining the integrity of the order processing.

# Summary

In summary, the Select Payment Method process ensures that customers choose a valid and active payment option during checkout. It involves presenting available methods, validating the selection, handling exceptions for invalid choices, and integrating with order calculation services to apply relevant fees and taxes.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
