---
title: GetAuthenticatedCustomerAsync flow
---
# Starting the customer authentication check

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is cached customer available?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:102:104"
    node1 -->|"Yes"| node2["Return cached customer"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:103:104"
    node1 -->|"No"| node3["Handling external user authentication"]
    
    node3 --> node4{"Did authentication succeed?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:108:109"
    node4 -->|"No"| node5["Return null"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:108:109"
    node4 -->|"Yes"| node6{"Is UsernamesEnabled?"}
    click node6 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:112:113"
    node6 -->|"Yes"| node7["Get customer by username"]
    click node7 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:115:118"
    node6 -->|"No"| node8["Get customer by email"]
    click node8 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:123:126"
    node7 --> node9{"Is customer valid and active?"}
    node8 --> node9
    click node9 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:130:131"
    node9 -->|"No"| node5
    node9 -->|"Yes"| node10["Cache and return authenticated customer"]
    click node10 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:134:136"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node3 goToHeading "Handling external user authentication"
node3:::HeadingStyle
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs" line="100">

---

We first return a cached customer if available, then authenticate the HTTP context with nopCommerce's scheme. If that fails, we return null. We rely on ExternalAuthenticationService next to handle external login details and claims.

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

## Handling external user authentication

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is external authentication plugin active?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:253:254"
    node1 -->|"No"| node2["Return error: External authentication method cannot be loaded"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:254:254"
    node1 -->|"Yes"| node3{"Is there an associated user?"}
    click node3 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:260:261"
    node3 -->|"No"| node4["Authenticate and associate new user"]
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:264:266"
    node3 -->|"Yes"| node5{"Is there a current logged-in user?"}
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:257:258"
    node5 -->|"No"| node6["Sign in associated user"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:88:91"
    node5 -->|"Yes"| node7{"Does current logged-in user match associated user?"}
    click node7 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:92:97"
    node7 -->|"No"| node8["Return error: Account already assigned"]
    click node8 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:94:94"
    node7 -->|"Yes"| node9["Return successful authentication"]
    click node9 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:97:97"

    class node2,node8 errorNode
    classDef errorNode fill:#f96,stroke:#333,stroke-width:2px;
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="248">

---

In `AuthenticateAsync`, we first validate the input parameters and check if the external authentication plugin is active for the current store and customer. Then we get the current logged-in user if any. Next, we try to find an existing user linked to the external authentication parameters. If found, we proceed to authenticate that existing user by calling `AuthenticateExistingUserAsync`.

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

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="86">

---

We sign in the associated user if no one is logged in, error if logged-in user differs, else confirm success.

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

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="264">

---

After handling existing users in `AuthenticateAsync`, if no associated user is found, we proceed to `AuthenticateNewUserAsync` to either link the external account to the current user or register a new user. This step covers new user scenarios and keeps the flow complete.

```c#
            //or associate and authenticate new user
            return await AuthenticateNewUserAsync(currentLoggedInUser, parameters, returnUrl);
        }
```

---

</SwmSnippet>

## Managing new user authentication and registration

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is user currently logged in?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:110:126"
    node1 -->|"Yes"| node2["Associate external account with logged-in user"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:113:117"
    node2 --> node3["Return successful authentication"]
    click node3 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:117:118"
    node1 -->|"No"| node4{"Is user registration enabled?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:121:122"
    node4 -->|"Yes"| node5["Register new user"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:122:123"
    node5 --> node3
    node4 -->|"No"| node6["Return error: Registration is disabled"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:125:126"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="110">

---

In `AuthenticateNewUserAsync`, we either link the external account to the logged-in user or try to register a new user if registration is enabled. If registration is disabled, we return an error. This handles new user scenarios and decides the next step accordingly.

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

## Registering a new user via external authentication

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start user registration"] --> node2{"Is email already registered?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:137:138"
    node2 -->|"Yes"| node3["Return error: Email already exists"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:139:145"
    node2 -->|"No"| node4["Check if registration is approved immediately"]
    click node4 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:147:149"
    node4 --> node5["Register new user"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:151:161"
    node5 --> node6{"Did registration succeed?"}
    click node6 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:161:163"
    node6 -->|"No"| node7["Return registration error"]
    click node7 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:161:163"
    node6 -->|"Yes"| node8["Publish registration events and notify store owner"]
    click node8 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:164:173"
    node8 --> node9["Associate external account with user"]
    click node9 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:174:176"
    node9 --> node10{"Is registration approved?"}
    click node10 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:177:179"
    node10 -->|"Yes"| node11["Send welcome message and sign in user"]
    click node11 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:179:186"
    node10 -->|"No"| node12{"User registration type?"}
    click node12 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:188:202"
    node12 -->|"EmailValidation"| node13["Send email validation message and redirect"]
    click node13 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:191:196"
    node12 -->|"AdminApproval"| node14["Redirect to admin approval page"]
    click node14 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:199:201"
    node12 -->|"Other"| node15["Return generic registration error"]
    click node15 openCode "src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs:202:203"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/External/ExternalAuthenticationService.cs" line="137">

---

In `RegisterNewUserAsync`, we check if the email is already registered to avoid duplicates. Then we prepare a registration request with generated password and approval status based on settings. We call CustomerRegistrationService to create the user, then trigger events and notifications. Finally, we link the external account and sign in if approved.

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

After registering a new user in `RegisterNewUserAsync`, we handle post-registration states: email validation, admin approval, or error. We send validation emails or redirect accordingly, or return an error if something went wrong.

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

## Finalizing customer retrieval from claims

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Are usernames enabled?"}
    click node1 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:112:113"
    node1 -->|"Yes"| node2["Try to get customer by username"]
    click node2 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:114:118"
    node1 -->|"No"| node3["Try to get customer by email"]
    click node3 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:122:126"
    node2 --> node4{"Is customer found and valid?"}
    click node4 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:130:131"
    node3 --> node4
    node4 -->|"Yes"| node5["Cache and return authenticated customer"]
    click node5 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:134:136"
    node4 -->|"No"| node6["Return null"]
    click node6 openCode "src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs:131:131"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/Libraries/Nop.Services/Authentication/CookieAuthenticationService.cs" line="111">

---

Back in `GetAuthenticatedCustomerAsync`, after external authentication, we check if usernames are enabled to decide whether to get the customer by username or email claim. We verify the claim issuer matches nopCommerce's expected issuer. If the customer is inactive, deleted, or requires re-login, we return null. Otherwise, we cache and return the customer.

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
