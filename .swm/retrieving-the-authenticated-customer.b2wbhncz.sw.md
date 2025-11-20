---
title: Retrieving the Authenticated Customer
---
This document describes the flow of identifying and retrieving the authenticated customer based on the current HTTP context. It covers checking existing authentication, handling external authentication methods, and validating the customer to ensure they are active and registered before returning the customer object for use in the system.

```mermaid
flowchart TD
 node1["Starting the Customer Authentication Retrieval
(Starting the Customer Authentication Retrieval)"]:::HeadingStyle
 click node1 goToHeading "Starting the Customer Authentication Retrieval"
 node1 --> node2{"Is cached customer available?
(Starting the Customer Authentication Retrieval)"}:::HeadingStyle
 click node2 goToHeading "Starting the Customer Authentication Retrieval"
 node2 -->|"Yes"| node3["Return authenticated customer
(Starting the Customer Authentication Retrieval)"]:::HeadingStyle
 click node3 goToHeading "Starting the Customer Authentication Retrieval"
 node2 -->|"No"| node4["Handling External Authentication Requests
(Handling External Authentication Requests)"]:::HeadingStyle
 click node4 goToHeading "Handling External Authentication Requests"
 node4 --> node5{"Did authentication succeed?
(Handling External Authentication Requests)"}:::HeadingStyle
 click node5 goToHeading "Handling External Authentication Requests"
 node5 -->|"No"| node6["No authenticated customer
(Starting the Customer Authentication Retrieval)"]:::HeadingStyle
 click node6 goToHeading "Starting the Customer Authentication Retrieval"
 node5 -->|"Yes"| node7{"Is username identification enabled?
(Finalizing Customer Retrieval and Validation)"}:::HeadingStyle
 click node7 goToHeading "Finalizing Customer Retrieval and Validation"
 node7 -->|"Yes"| node8["Get customer by username
(Finalizing Customer Retrieval and Validation)"]:::HeadingStyle
 click node8 goToHeading "Finalizing Customer Retrieval and Validation"
 node7 -->|"No"| node9["Get customer by email
(Finalizing Customer Retrieval and Validation)"]:::HeadingStyle
 click node9 goToHeading "Finalizing Customer Retrieval and Validation"
 node8 --> node10{"Is customer valid and active?
(Finalizing Customer Retrieval and Validation)"}:::HeadingStyle
 click node10 goToHeading "Finalizing Customer Retrieval and Validation"
 node9 --> node10
 node10 -->|"No"| node6
 node10 -->|"Yes"| node3
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Starting the Customer Authentication Retrieval

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is cached customer available?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:102:104"
    node1 -->|"Yes"| node2["Return cached customer"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:103:104"
    node1 -->|"No"| node3["Handling External Authentication Requests"]
    
    node3 --> node4{"Did authentication succeed?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:108:109"
    node4 -->|"No"| node5["Return null"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:108:109"
    node4 -->|"Yes"| node6{"Is username identification enabled?"}
    click node6 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:112:113"
    node6 -->|"Yes"| node7["Get customer by username"]
    click node7 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:114:118"
    node6 -->|"No"| node8["Get customer by email"]
    click node8 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:122:126"
    node7 --> node9{"Is customer valid and active?"}
    click node9 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:129:131"
    node8 --> node9
    node9 -->|"No"| node5
    node9 -->|"Yes"| node10["Cache and return authenticated customer"]
    click node10 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:134:136"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node3 goToHeading "Handling External Authentication Requests"
node3:::HeadingStyle
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs" line="100">

---

Here we start by checking if we already have a cached customer to avoid redundant work. Then, we authenticate the HTTP context using the configured scheme. If that fails, we bail out early. This sets the stage for fetching the customer details, which we do next by calling external authentication services to handle more complex scenarios.

```c#
        public virtual async Task<Customer> GetAuthenticatedCustomerAsync()
        {
            //whether there is a cached customer
            if (_cachedCustomer != null)
                return _cachedCustomer;

            //try to get authenticated user identity
            var authenticateResult = await _httpContextAccessor.HttpContext.AuthenticateAsync(NopAuthenticationDefaults.AuthenticationScheme);
            if (!authenticateResult.Succeeded)
                return null;

```

---

</SwmSnippet>

## Handling External Authentication Requests

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="248">

---

In `AuthenticateAsync` we first verify the external authentication plugin is active for the current store and user. If not, we return an error. Then, we check if there's a logged-in user and try to find an associated user for the external credentials. If found, we proceed to authenticate that user.

```c#
        public virtual async Task<IActionResult> AuthenticateAsync(ExternalAuthenticationParameters parameters, string returnUrl = null)
        {
            if (parameters == null)
                throw new ArgumentNullException(nameof(parameters));

            if (!await _authenticationPluginManager.IsPluginActiveAsync(parameters.ProviderSystemName, await _workContext.GetCurrentCustomerAsync(), (await _storeContext.GetCurrentStoreAsync()).Id))
                return ErrorAuthentication(new[] { "External authentication method cannot be loaded" }, returnUrl);

            //get current logged-in user
            var currentLoggedInUser = await _customerService.IsRegisteredAsync(await _workContext.GetCurrentCustomerAsync()) ? await _workContext.GetCurrentCustomerAsync() : null;

            //authenticate associated user if already exists
            var associatedUser = await GetUserByExternalAuthenticationParametersAsync(parameters);
            if (associatedUser != null)
                return await AuthenticateExistingUserAsync(associatedUser, currentLoggedInUser, returnUrl);

```

---

</SwmSnippet>

### Authenticating an Existing External User

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is current user a guest?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:88:98"
    node1 -->|"Yes"| node2["Log in guest user"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:89:91"
    node1 -->|"No"| node3{"Is external account assigned to another user?"}
    click node3 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:92:95"
    node3 -->|"Yes"| node4["Return error: account already assigned"]
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:94:95"
    node3 -->|"No"| node5["Return successful login"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:96:98"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="86">

---

`AuthenticateExistingUserAsync` handles three cases: no logged-in user, so it signs in the associated user; logged-in user differs from associated user, so it returns an error; or the user tries to log in as themselves, so it just confirms success.

```c#
        protected virtual async Task<IActionResult> AuthenticateExistingUserAsync(Customer associatedUser, Customer currentLoggedInUser, string returnUrl)
        {
            //log in guest user
            if (currentLoggedInUser == null)
                return await _customerRegistrationService.SignInCustomerAsync(associatedUser, returnUrl);

            //account is already assigned to another user
            if (currentLoggedInUser.Id != associatedUser.Id)
                return ErrorAuthentication(new[] { await _localizationService.GetResourceAsync("Account.AssociatedExternalAuth.AccountAlreadyAssigned") }, returnUrl);

            //or the user try to log in as himself. bit weird
            return SuccessfulAuthentication(returnUrl);
        }
```

---

</SwmSnippet>

### Signing In the Customer and Migrating Cart

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start sign-in process"] --> node2{"Is current customer different from signing-in customer?"}
    click node1 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:415:416"
    node2 -->|"Yes"| node3["Transfer shopping cart to new customer"]
    click node2 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:417:421"
    node3 --> node4["Set new customer as current"]
    click node3 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:420:423"
    node2 -->|"No"| node4
    node4 --> node5["Sign in customer"]
    click node4 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:422:426"
    node5 --> node6["Publish customer logged-in event"]
    click node5 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:426:429"
    node6 --> node7["Log customer login activity"]
    click node6 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:429:434"
    node7 --> node8{"Is return URL specified?"}
    click node7 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:432:435"
    node8 -->|"Yes"| node9["Redirect to return URL"]
    click node8 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:436:438"
    node8 -->|"No"| node10["Redirect to homepage"]
    click node9 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:436:438"
    click node10 openCode "src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs:439:440"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs" line="415">

---

In `SignInCustomerAsync` we check if the current customer differs from the one signing in. If so, we migrate their shopping cart to the new customer and update the current customer context.

```c#
        public virtual async Task<IActionResult> SignInCustomerAsync(Customer customer, string returnUrl, bool isPersist = false)
        {
            if ((await _workContext.GetCurrentCustomerAsync())?.Id != customer.Id)
            {
                //migrate shopping cart
                await _shoppingCartService.MigrateShoppingCartAsync(await _workContext.GetCurrentCustomerAsync(), customer, true);

                await _workContext.SetCurrentCustomerAsync(customer);
            }

```

---

</SwmSnippet>

<SwmSnippet path="/src/Libraries/Nop.Services/Customers/CustomerRegistrationService.cs" line="425">

---

After updating the current customer, `SignInCustomerAsync` signs in the user, triggers login events and activity logs, then redirects to the return URL or homepage.

```c#
            //sign in new customer
            await _authenticationService.SignInAsync(customer, isPersist);

            //raise event       
            await _eventPublisher.PublishAsync(new CustomerLoggedinEvent(customer));

            //activity log
            await _customerActivityService.InsertActivityAsync(customer, "PublicStore.Login",
                await _localizationService.GetResourceAsync("ActivityLog.PublicStore.Login"), customer);

            //redirect to the return URL if it's specified
            if (!string.IsNullOrEmpty(returnUrl))
                return new RedirectResult(returnUrl);

            return new RedirectToRouteResult("Homepage", null);
        }
```

---

</SwmSnippet>

### Handling New External User Authentication

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="264">

---

After trying to authenticate an existing user, the flow moves to `AuthenticateNewUserAsync` to handle new user association or registration.

```c#
            //or associate and authenticate new user
            return await AuthenticateNewUserAsync(currentLoggedInUser, parameters, returnUrl);
        }
```

---

</SwmSnippet>

## Associating or Registering New External Users

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is user currently logged in?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:113:114"
    node1 -->|"Yes"| node2["Associate external account with logged-in user"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:115:116"
    node2 --> node3["Return successful authentication"]
    click node3 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:117:118"
    node1 -->|"No"| node4{"Is user registration enabled?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:121:122"
    node4 -->|"Yes"| node5["Register new user"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:122:123"
    node5 --> node3
    node4 -->|"No"| node6["Return error: Registration disabled"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:125:126"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="110">

---

`AuthenticateNewUserAsync` checks if there's a logged-in user to associate the external account with. If not, it tries to register a new user unless registration is disabled, returning errors accordingly.

```c#
        protected virtual async Task<IActionResult> AuthenticateNewUserAsync(Customer currentLoggedInUser, ExternalAuthenticationParameters parameters, string returnUrl)
        {
            //associate external account with logged-in user
            if (currentLoggedInUser != null)
            {
                await AssociateExternalAccountWithUserAsync(currentLoggedInUser, parameters);

                return SuccessfulAuthentication(returnUrl);
            }

            //or try to register new user
            if (_customerSettings.UserRegistrationType != UserRegistrationType.Disabled)
                return await RegisterNewUserAsync(parameters, returnUrl);

            //registration is disabled
            return ErrorAuthentication(new[] { "Registration is disabled" }, returnUrl);
        }
```

---

</SwmSnippet>

## Registering a New User via External Authentication

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is email already registered?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:139:141"
    node1 -->|"Yes"| node2["Return error: Email already exists"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:142:144"
    node1 -->|"No"| node3["Create registration request and register user"]
    click node3 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:151:160"
    node3 --> node4{"Did registration succeed?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:161:162"
    node4 -->|"No"| node5["Return registration errors"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:162:162"
    node4 -->|"Yes"| node6["Publish registration events and send notifications"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:164:172"
    node6 --> node7["Associate external account with user"]
    click node7 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:174:175"
    node7 --> node8{"Is registration approved immediately?"}
    click node8 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:147:149"
    node8 -->|"Yes"| node9["Sign in user and return success"]
    click node9 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:179:185"
    node8 -->|"No"| node10{"User registration type"}
    click node10 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:189:200"
    node10 -->|"EmailValidation"| node11["Send email validation message and redirect"]
    click node11 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:191:195"
    node10 -->|"AdminApproval"| node12["Redirect to admin approval page"]
    click node12 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:199:200"
    node10 -->|"Other"| node13["Return generic registration error"]
    click node13 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:202:202"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="137">

---

In `RegisterNewUserAsync` we first check if the email is already registered to avoid duplicates. Then, we create a registration request and call the customer registration service to register the user. After that, we trigger events and notifications, and associate the external account with the new user.

```c#
        protected virtual async Task<IActionResult> RegisterNewUserAsync(ExternalAuthenticationParameters parameters, string returnUrl)
        {
            //check whether the specified email has been already registered
            if (await _customerService.GetCustomerByEmailAsync(parameters.Email) != null)
            {
                var alreadyExistsError = string.Format(await _localizationService.GetResourceAsync("Account.AssociatedExternalAuth.EmailAlreadyExists"),
                    !string.IsNullOrEmpty(parameters.ExternalDisplayIdentifier) ? parameters.ExternalDisplayIdentifier : parameters.ExternalIdentifier);
                return ErrorAuthentication(new[] { alreadyExistsError }, returnUrl);
            }

            //registration is approved if validation isn't required
            var registrationIsApproved = _customerSettings.UserRegistrationType == UserRegistrationType.Standard ||
                (_customerSettings.UserRegistrationType == UserRegistrationType.EmailValidation && !_externalAuthenticationSettings.RequireEmailValidation);

            //create registration request
            var registrationRequest = new CustomerRegistrationRequest(await _workContext.GetCurrentCustomerAsync(),
                parameters.Email, parameters.Email,
                CommonHelper.GenerateRandomDigitCode(20),
                PasswordFormat.Hashed,
                (await _storeContext.GetCurrentStoreAsync()).Id,
                registrationIsApproved);

            //whether registration request has been completed successfully
            var registrationResult = await _customerRegistrationService.RegisterCustomerAsync(registrationRequest);
            if (!registrationResult.Success)
                return ErrorAuthentication(registrationResult.Errors, returnUrl);

            //allow to save other customer values by consuming this event
            await _eventPublisher.PublishAsync(new CustomerAutoRegisteredByExternalMethodEvent(await _workContext.GetCurrentCustomerAsync(), parameters));

            //raise customer registered event
            await _eventPublisher.PublishAsync(new CustomerRegisteredEvent(await _workContext.GetCurrentCustomerAsync()));

            //store owner notifications
            if (_customerSettings.NotifyNewCustomerRegistration)
                await _workflowMessageService.SendCustomerRegisteredNotificationMessageAsync(await _workContext.GetCurrentCustomerAsync(), _localizationSettings.DefaultAdminLanguageId);

            //associate external account with registered user
            await AssociateExternalAccountWithUserAsync(await _workContext.GetCurrentCustomerAsync(), parameters);

            //authenticate
            if (registrationIsApproved)
            {
                await _workflowMessageService.SendCustomerWelcomeMessageAsync(await _workContext.GetCurrentCustomerAsync(), (await _workContext.GetWorkingLanguageAsync()).Id);

                //raise event       
                await _eventPublisher.PublishAsync(new CustomerActivatedEvent(await _workContext.GetCurrentCustomerAsync()));

                return await _customerRegistrationService.SignInCustomerAsync(await _workContext.GetCurrentCustomerAsync(), returnUrl, true);
            }

```

---

</SwmSnippet>

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="188">

---

After registering the user, `RegisterNewUserAsync` handles cases where registration requires email validation or admin approval by sending emails and redirecting accordingly. If registration fails, it returns an error.

```c#
            //registration is succeeded but isn't activated
            if (_customerSettings.UserRegistrationType == UserRegistrationType.EmailValidation)
            {
                //email validation message
                await _genericAttributeService.SaveAttributeAsync(await _workContext.GetCurrentCustomerAsync(), NopCustomerDefaults.AccountActivationTokenAttribute, Guid.NewGuid().ToString());
                await _workflowMessageService.SendCustomerEmailValidationMessageAsync(await _workContext.GetCurrentCustomerAsync(), (await _workContext.GetWorkingLanguageAsync()).Id);

                return new RedirectToRouteResult("RegisterResult", new { resultId = (int)UserRegistrationType.EmailValidation, returnUrl });
            }

            //registration is succeeded but isn't approved by admin
            if (_customerSettings.UserRegistrationType == UserRegistrationType.AdminApproval)
                return new RedirectToRouteResult("RegisterResult", new { resultId = (int)UserRegistrationType.AdminApproval, returnUrl });

            return ErrorAuthentication(new[] { "Error on registration" }, returnUrl);
        }
```

---

</SwmSnippet>

## Finalizing Customer Retrieval and Validation

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start authentication"] --> node2{"Usernames enabled?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:111:113"
    node2 -->|"Yes"| node3["Try get customer by username"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:112:119"
    node2 -->|"No"| node4["Try get customer by email"]
    click node4 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:120:126"
    node3 --> node5{"Customer found and valid?"}
    click node3 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:114:119"
    node4 --> node5
    click node5 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:129:131"
    node5 -->|"No"| node6["Return null"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:131:131"
    node5 -->|"Yes"| node7["Cache authenticated customer"]
    click node7 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:134:134"
    node7 --> node8["Return authenticated customer"]
    click node8 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:136:136"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs" line="111">

---

We pick username or email based on settings, validate the customer, cache if valid, or return null.

```c#
            Customer customer = null;
            if (_customerSettings.UsernamesEnabled)
            {
                //try to get customer by username
                var usernameClaim = authenticateResult.Principal.FindFirst(claim => claim.Type == ClaimTypes.Name
                    && claim.Issuer.Equals(NopAuthenticationDefaults.ClaimsIssuer, StringComparison.InvariantCultureIgnoreCase));
                if (usernameClaim != null)
                    customer = await _customerService.GetCustomerByUsernameAsync(usernameClaim.Value);
            }
            else
            {
                //try to get customer by email
                var emailClaim = authenticateResult.Principal.FindFirst(claim => claim.Type == ClaimTypes.Email
                    && claim.Issuer.Equals(NopAuthenticationDefaults.ClaimsIssuer, StringComparison.InvariantCultureIgnoreCase));
                if (emailClaim != null)
                    customer = await _customerService.GetCustomerByEmailAsync(emailClaim.Value);
            }

            //whether the found customer is available
            if (customer == null || !customer.Active || customer.RequireReLogin || customer.Deleted || !await _customerService.IsRegisteredAsync(customer))
                return null;

            //cache authenticated customer
            _cachedCustomer = customer;

            return _cachedCustomer;
        }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
