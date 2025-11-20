---
title: Selecting and Validating Payment Methods in Order Processing
---
# Overview of Selecting Payment Method in Order Processing

Selecting a payment method is a fundamental part of the order processing workflow in nopCommerce. This process involves identifying the payment option chosen by the customer during checkout and ensuring that it is valid, active, and ready to be used for the transaction.

The system loads the details of the selected payment method to verify its availability. This step is essential to prevent orders from proceeding with invalid or inactive payment options, which could lead to payment failures or inconsistencies in order handling.

# Validation and Exception Handling

During order processing, the service attempts to load the payment method details. If the payment method cannot be found, the service throws a 'Payment method couldn't be loaded' exception. Similarly, if the payment method is found but marked as inactive, a 'Payment method is not active' exception is thrown. These exceptions halt the order processing to ensure that only valid payment methods are used.

This robust validation mechanism is implemented in the `OrderProcessingService` class, which centralizes the logic for handling payment method selection and validation within the order workflow.

# Calculating Additional Fees for Payment Methods

After successful validation, the service calculates any additional fees associated with the selected payment method. These fees may include processing charges or taxes that apply specifically to the payment option chosen by the customer.

The calculated fees are then added to the order total, ensuring that the final amount billed to the customer accurately reflects all costs related to the payment method.

# Implementation Details in OrderProcessingService

The selection and validation logic is encapsulated within the `OrderProcessingService` class located in <SwmPath>[src/…/Orders/OrderProcessingService.cs](src/Libraries/Nop.Services/Orders/OrderProcessingService.cs)</SwmPath>. Between lines 1420 and 1435, the service performs the loading of the payment method, checks its active status, and throws exceptions if necessary.

This implementation ensures that the order processing workflow maintains integrity by preventing invalid payment methods from being used and by correctly applying any additional fees.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
