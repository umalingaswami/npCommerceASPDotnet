---
title: Updating shipping options flow
---
This document describes the flow that updates shipping options when the shipping address changes. It clears previous options, validates the new address, requests updated shipping options, and updates the UI to reflect the available choices.

```mermaid
flowchart TD
 node1["Starting the Shipping Address Update"]:::HeadingStyle
 click node1 goToHeading "Starting the Shipping Address Update"
 node2["Validating and Initiating Shipping Options Retrieval"]:::HeadingStyle
 click node2 goToHeading "Validating and Initiating Shipping Options Retrieval"
 node3["Processing and Displaying Shipping Options"]:::HeadingStyle
 click node3 goToHeading "Processing and Displaying Shipping Options"
 node4["Validating and Setting the Selected Shipping Option"]:::HeadingStyle
 click node4 goToHeading "Validating and Setting the Selected Shipping Option"
 node5["Finalizing Shipping Options Display and Error Handling"]:::HeadingStyle
 click node5 goToHeading "Finalizing Shipping Options Display and Error Handling"

 node1 --> node2
 node2 -->|"Address valid"| node3
 node2 -->|"Address invalid"| node5
 node3 --> node4
 node3 -->|"No options or response failed"| node5

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

```mermaid
graph TD;
      84fb5b805948b72da2e2bae6afc067e720a45af804cf933d68f376a855b8825a(src/…/js/public.estimateshipping.popup.js::createEstimateShippingPopUp) --> 6a17ed3fb6b1ac3237c5e8329f5648d9ab32ecb499f75f70fdb47ee62c700092(src/…/js/public.estimateshipping.popup.js::addressChangedHandler):::mainFlowStyle

3b2e85b181e075f546a334c4a8f6e733616f432018b39242be0206b8b2e60064(src/…/js/public.estimateshipping.popup.js::init) --> 6a17ed3fb6b1ac3237c5e8329f5648d9ab32ecb499f75f70fdb47ee62c700092(src/…/js/public.estimateshipping.popup.js::addressChangedHandler):::mainFlowStyle


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9

%% Swimm:
%% graph TD;
%%       84fb5b805948b72da2e2bae6afc067e720a45af804cf933d68f376a855b8825a(<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>::<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="1:3:3" line-data="﻿function createEstimateShippingPopUp(settings) {">`createEstimateShippingPopUp`</SwmToken>) --> 6a17ed3fb6b1ac3237c5e8329f5648d9ab32ecb499f75f70fdb47ee62c700092(<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>::<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="55:3:3" line-data="      var addressChangedHandler = function () {">`addressChangedHandler`</SwmToken>):::mainFlowStyle
%% 
%% 3b2e85b181e075f546a334c4a8f6e733616f432018b39242be0206b8b2e60064(<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>::init) --> 6a17ed3fb6b1ac3237c5e8329f5648d9ab32ecb499f75f70fdb47ee62c700092(<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>::<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="55:3:3" line-data="      var addressChangedHandler = function () {">`addressChangedHandler`</SwmToken>):::mainFlowStyle
%% 
%% 
%% classDef mainFlowStyle color:#000000,fill:#7CB9F4
%% classDef rootsStyle color:#000000,fill:#00FFF4
%% classDef Style1 color:#000000,fill:#00FFAA
%% classDef Style2 color:#000000,fill:#FFFF00
%% classDef Style3 color:#000000,fill:#AA7CB9
```

# Starting the Shipping Address Update

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="55">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="55:3:3" line-data="      var addressChangedHandler = function () {">`addressChangedHandler`</SwmToken> clears previous shipping options, fetches the current shipping address, and triggers fetching new shipping options based on that address. Calling <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="58:3:3" line-data="        self.getShippingOptions(address);">`getShippingOptions`</SwmToken> next is necessary to update the available shipping options according to the new address.

```javascript
      var addressChangedHandler = function () {
        self.clearShippingOptions();
        var address = self.getShippingAddress();
        self.getShippingOptions(address);
      };
```

---

</SwmSnippet>

# Validating and Initiating Shipping Options Retrieval

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="77">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="77:1:1" line-data="    getShippingOptions: function (address) {">`getShippingOptions`</SwmToken> validates the address first to avoid fetching options for invalid addresses.

```javascript
    getShippingOptions: function (address) {
      if (!this.validateAddress(address))
        return;

```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="295">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="295:1:1" line-data="    validateAddress: function (address) {">`validateAddress`</SwmToken> checks required fields on the address using settings to decide which fields are mandatory and localized messages for errors. It clears old errors, collects new ones if fields like country, city, or postal code are missing, then shows errors if any exist. It assumes the address has specific properties and uses settings to tailor validation.

```javascript
    validateAddress: function (address) {
      this.clearErrorMessage();

      var errors = [];
      var localizedData = this.settings.localizedData;

      if (!(address.countryName && address.countryId > 0))
        errors.push(localizedData.countryErrorMessage);

      if (this.settings.useCity && !address.city)
        errors.push(localizedData.cityErrorMessage);

      if (!this.settings.useCity && !address.zipPostalCode)
        errors.push(localizedData.zipPostalCodeErrorMessage);

      if (errors.length > 0)
        this.showErrorMessage(errors);

      return errors.length === 0;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="81">

---

After returning from <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="78:7:7" line-data="      if (!this.validateAddress(address))">`validateAddress`</SwmToken> in <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="58:3:3" line-data="        self.getShippingOptions(address);">`getShippingOptions`</SwmToken>, the function sets a loading state, triggers a load handler if present, cancels any ongoing request, then makes an AJAX call to fetch shipping options. On success, it calls <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="101:3:3" line-data="              self.successHandler(address, response);">`successHandler`</SwmToken> to process and display the options.

```javascript
      var self = this;

      self.setLoadWaiting();

      if (self.settings.handlers.load)
        self.settings.handlers.load();

      clearTimeout(self.params.delayTimer);
      self.params.delayTimer = setTimeout(function () {
        if (self.params.jqXHR && self.params.jqXHR.readyState !== 4)
          self.params.jqXHR.abort();

        var url = self.settings.urlFactory(address);
        if (url) {
          self.params.jqXHR = $.ajax({
            cache: false,
            url: url,
            data: $(self.settings.form).serialize(),
            type: 'POST',
            success: function (response) {
              self.successHandler(address, response);
            },
            error: function (jqXHR, textStatus, errorThrown) {
```

---

</SwmSnippet>

## Processing and Displaying Shipping Options

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Check if response is successful and shipping options exist"]
    node1 -->|"Yes"| node2["Validating and Setting the Selected Shipping Option"]
    node2 --> node4{"Is active option found?"}
    node4 -->|"No"| node5["Set first shipping option as active"]
    node4 -->|"Yes"| node6["Check if popup is open and selected option matches address"]
    node6 -->|"Yes"| node7["Reload selected shipping option"]
    node6 -->|"No"| node8["Set active shipping option"]
    node1 -->|"No"| node3["Clear shipping options and show errors"]

    click node1 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:123:127"
    
    click node3 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:163:170"
    click node4 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:148:155"
    click node5 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:148:155"
    click node6 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:158:160"
    click node7 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:158:160"
    click node8 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:161:162"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node2 goToHeading "Validating and Setting the Selected Shipping Option"
node2:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Check if response is successful and shipping options exist"]
%%     node1 -->|"Yes"| node2["Validating and Setting the Selected Shipping Option"]
%%     node2 --> node4{"Is active option found?"}
%%     node4 -->|"No"| node5["Set first shipping option as active"]
%%     node4 -->|"Yes"| node6["Check if popup is open and selected option matches address"]
%%     node6 -->|"Yes"| node7["Reload selected shipping option"]
%%     node6 -->|"No"| node8["Set active shipping option"]
%%     node1 -->|"No"| node3["Clear shipping options and show errors"]
%% 
%%     click node1 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:123:127"
%%     
%%     click node3 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:163:170"
%%     click node4 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:148:155"
%%     click node5 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:148:155"
%%     click node6 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:158:160"
%%     click node7 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:158:160"
%%     click node8 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:161:162"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node2 goToHeading "Validating and Setting the Selected Shipping Option"
%% node2:::HeadingStyle
```

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="120">

---

In <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="120:1:1" line-data="    successHandler: function (address, response) {">`successHandler`</SwmToken>, the function clears existing shipping options UI, checks if the response is successful, then iterates over shipping options to find the active one and adds each option to the UI by calling <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="144:3:3" line-data="            self.addShippingOption(option.Name, option.DeliveryDateFormat, option.Price);">`addShippingOption`</SwmToken>. This populates the UI with new options.

```javascript
    successHandler: function (address, response) {
      $('.shipping-options-body', $(this.settings.contentEl)).empty();

      if (response.Success) {
        var activeOption;

        var options = response.ShippingOptions;
        if (options && options.length > 0) {
          var self = this;
          var selectedShippingOption = this.params.selectedShippingOption;

          $.each(options, function (i, option) {
            // try select the shipping option with the same provider and address
            if (option.Selected ||
              (selectedShippingOption &&
                selectedShippingOption.provider === option.Name &&
                self.addressesAreEqual(selectedShippingOption.address, address))) {
              activeOption = {
                provider: option.Name,
                price: option.Price,
                address: address,
                deliveryDate: option.DeliveryDateFormat
              };
            }
            self.addShippingOption(option.Name, option.DeliveryDateFormat, option.Price);
          });

```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="206">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="206:1:1" line-data="    addShippingOption: function (name, deliveryDate, price) {">`addShippingOption`</SwmToken> creates a UI row for a shipping option using <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="213:57:61" line-data="          .append($(&#39;&lt;input/&gt;&#39;).addClass(&#39;estimate-shipping-radio&#39;).attr({ &#39;type&#39;: &#39;radio&#39;, &#39;name&#39;: &#39;shipping-option&#39; + &#39;-&#39; + this.settings.contentEl }))">`this.settings.contentEl`</SwmToken> to scope elements. It shows name, price, and delivery date (or '-' if missing). It adds a click handler to select the option and update UI classes for active state, then appends it to the container.

```javascript
    addShippingOption: function (name, deliveryDate, price) {
      if (!name || !price) return;

      var shippingOption = $('<div/>').addClass('estimate-shipping-row shipping-option');

      shippingOption
        .append($('<div/>').addClass('estimate-shipping-row-item-radio')
          .append($('<input/>').addClass('estimate-shipping-radio').attr({ 'type': 'radio', 'name': 'shipping-option' + '-' + this.settings.contentEl }))
          .append($('<label/>')))
        .append($('<div/>').addClass('estimate-shipping-row-item shipping-item').text(name))
        .append($('<div/>').addClass('estimate-shipping-row-item shipping-item').text(deliveryDate ? deliveryDate : '-'))
        .append($('<div/>').addClass('estimate-shipping-row-item shipping-item').text(price));

      var self = this;

      shippingOption.on('click', function () {
        $('input[name="shipping-option' + '-' + self.settings.contentEl + '"]', $(this)).prop('checked', true);
        $('.shipping-option.active', $(self.settings.contentEl)).removeClass('active');
        $(this).addClass('active');
      });

      $('.shipping-options-body', $(this.settings.contentEl)).append(shippingOption);
    },
```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="147">

---

After adding shipping options in <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="101:3:3" line-data="              self.successHandler(address, response);">`successHandler`</SwmToken>, the code picks a default active option if none matched earlier, then calls <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="159:3:3" line-data="            this.selectShippingOption(activeOption);">`selectShippingOption`</SwmToken> to update the selected option state and UI accordingly.

```javascript
          // select the first option
          if (!activeOption) {
            activeOption = {
              provider: options[0].Name,
              price: options[0].Price,
              deliveryDate: options[0].DeliveryDateFormat,
              address: address
            };
          }

          // if we have the already selected shipping options with the same address, reload it
          if (!$.magnificPopup.instance.isOpen && selectedShippingOption && this.addressesAreEqual(selectedShippingOption.address, address))
            this.selectShippingOption(activeOption);

          this.setActiveShippingOption(activeOption);
        } else {
          this.clearShippingOptions();
        }
      } else {
        this.params.displayErrors = true;
        this.clearErrorMessage();
```

---

</SwmSnippet>

### Validating and Setting the Selected Shipping Option

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is shipping option valid?"}
    click node1 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:198:201"
    node1 -->|"Yes"| node2["Update selected shipping option"]
    click node2 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:201:202"
    node2 --> node3{"Is handler for selected option present?"}
    click node3 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:202:204"
    node3 -->|"Yes"| node4["Invoke handler with selected option"]
    click node4 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:203:204"
    node3 -->|"No"| node5["End"]
    node1 -->|"No"| node5["End"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1{"Is shipping option valid?"}
%%     click node1 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:198:201"
%%     node1 -->|"Yes"| node2["Update selected shipping option"]
%%     click node2 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:201:202"
%%     node2 --> node3{"Is handler for selected option present?"}
%%     click node3 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:202:204"
%%     node3 -->|"Yes"| node4["Invoke handler with selected option"]
%%     click node4 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:203:204"
%%     node3 -->|"No"| node5["End"]
%%     node1 -->|"No"| node5["End"]
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="198">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="198:1:1" line-data="    selectShippingOption: function (option) {">`selectShippingOption`</SwmToken> validates the option's address before setting it as selected.

```javascript
    selectShippingOption: function (option) {
      if (option && option.provider && option.price && this.validateAddress(option.address))
        this.params.selectedShippingOption = option;

```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="202">

---

After validating and setting the selected option in <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="159:3:3" line-data="            this.selectShippingOption(activeOption);">`selectShippingOption`</SwmToken>, the function calls a handler if defined to notify other parts of the system about the selection, enabling custom reactions.

```javascript
      if (this.settings.handlers.selectedOption)
        this.settings.handlers.selectedOption(option);
    },
```

---

</SwmSnippet>

### Finalizing Shipping Options Display and Error Handling

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Clear previous shipping options"]
    click node1 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:168:169"
    node1 --> node2{"Are there errors in the response?"}
    click node2 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:169:170"
    node2 -->|"Yes"| node3["Show error messages"]
    click node3 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:169:170"
    node2 -->|"No"| node4["Check if success callback is defined"]
    click node4 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:172:173"
    node3 --> node4
    node4 -->|"Yes"| node5["Invoke success callback with address and response"]
    click node5 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:173:174"
    node4 -->|"No"| node6["End"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Clear previous shipping options"]
%%     click node1 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:168:169"
%%     node1 --> node2{"Are there errors in the response?"}
%%     click node2 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:169:170"
%%     node2 -->|"Yes"| node3["Show error messages"]
%%     click node3 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:169:170"
%%     node2 -->|"No"| node4["Check if success callback is defined"]
%%     click node4 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:172:173"
%%     node3 --> node4
%%     node4 -->|"Yes"| node5["Invoke success callback with address and response"]
%%     click node5 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:173:174"
%%     node4 -->|"No"| node6["End"]
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="168">

---

After returning from <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="159:3:3" line-data="            this.selectShippingOption(activeOption);">`selectShippingOption`</SwmToken> in <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="101:3:3" line-data="              self.successHandler(address, response);">`successHandler`</SwmToken>, the function clears shipping options and shows errors if the response failed, then calls a success handler if defined to finalize the flow.

```javascript
        this.clearShippingOptions();
        this.showErrorMessage(response.Errors);
      }

      if (this.settings.handlers.success)
        this.settings.handlers.success(address, response);
    },
```

---

</SwmSnippet>

## Completing the Shipping Options Request with Error Handling

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start getShippingOptions"]
    click node1 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node1 --> node2{"Is shipping address valid?"}
    click node2 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node2 -->|"No"| node3["Trigger error handler: Invalid address"]
    click node3 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:176:183"
    node2 -->|"Yes"| node4["Send request for shipping options"]
    click node4 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node4 --> node5{"Are shipping options returned?"}
    click node5 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node5 -->|"No"| node6["Trigger error handler: No shipping options"]
    click node6 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:176:183"
    node5 -->|"Yes"| node7["Show shipping options to user"]
    click node7 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node7 --> node8["User selects shipping option"]
    click node8 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"
    node8 --> node9["Return selected shipping option"]
    click node9 openCode "src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js:104:113"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="58:3:3" line-data="        self.getShippingOptions(address);">`getShippingOptions`</SwmToken>"]
%%     click node1 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node1 --> node2{"Is shipping address valid?"}
%%     click node2 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node2 -->|"No"| node3["Trigger error handler: Invalid address"]
%%     click node3 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:176:183"
%%     node2 -->|"Yes"| node4["Send request for shipping options"]
%%     click node4 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node4 --> node5{"Are shipping options returned?"}
%%     click node5 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node5 -->|"No"| node6["Trigger error handler: No shipping options"]
%%     click node6 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:176:183"
%%     node5 -->|"Yes"| node7["Show shipping options to user"]
%%     click node7 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node7 --> node8["User selects shipping option"]
%%     click node8 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%%     node8 --> node9["Return selected shipping option"]
%%     click node9 openCode "<SwmPath>[src/…/js/public.estimateshipping.popup.js](src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js)</SwmPath>:104:113"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="104">

---

After returning from <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="101:3:3" line-data="              self.successHandler(address, response);">`successHandler`</SwmToken> in <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="58:3:3" line-data="        self.getShippingOptions(address);">`getShippingOptions`</SwmToken>, the function defines error and complete handlers for the AJAX request. Calling <SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="104:3:3" line-data="              self.errorHandler(jqXHR, textStatus, errorThrown);">`errorHandler`</SwmToken> on failure manages cleanup and error display.

```javascript
              self.errorHandler(jqXHR, textStatus, errorThrown);
            },
            complete: function (jqXHR, textStatus) {
              if (self.settings.handlers.complete)
                self.settings.handlers.complete(jqXHR, textStatus);
            }
          });
        }
      }, self.settings.requestDelay);
    },
```

---

</SwmSnippet>

<SwmSnippet path="/src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" line="176">

---

<SwmToken path="src/Presentation/Nop.Web/wwwroot/js/public.estimateshipping.popup.js" pos="176:1:1" line-data="    errorHandler: function (jqXHR, textStatus, errorThrown) {">`errorHandler`</SwmToken> ignores aborted requests, clears shipping options on errors, and calls an error handler if defined to manage UI and state cleanup.

```javascript
    errorHandler: function (jqXHR, textStatus, errorThrown) {
      if (textStatus === 'abort') return;

      this.clearShippingOptions();

      if (this.settings.handlers.error)
        this.settings.handlers.error(jqXHR, textStatus, errorThrown);
    },
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
