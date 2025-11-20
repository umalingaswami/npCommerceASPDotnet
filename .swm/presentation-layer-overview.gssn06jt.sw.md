---
title: Presentation Layer Overview
---
# Overview of the Presentation Layer

The Presentation Layer in nopCommerce is the component responsible for managing the user interface and handling user interactions. It determines how data is displayed to users and how users provide input to the system.

This layer acts as a bridge between the user and the underlying business logic and services, retrieving data to present and sending user input back for processing.

# Implementation with [ASP.NET](http://ASP.NET) Core 5

Built using [ASP.NET](http://ASP.NET) Core 5, the Presentation Layer leverages a modern, cross-platform web framework. This choice enables nopCommerce to run on multiple operating systems while supporting contemporary web development practices such as dependency injection, middleware, and Razor Pages or MVC.

# Separation of Concerns

By isolating the user interface logic within the Presentation Layer, nopCommerce achieves a clear separation of concerns. This separation enhances maintainability and scalability by keeping UI code distinct from business logic and data access layers.

# Extensibility via Themes and Plugins

The Presentation Layer supports extensibility through themes and plugins. Themes allow developers to customize the visual appearance of the storefront, including layout, colors, and styles, without modifying core code. Plugins can introduce new UI features or modify existing ones, such as adding widgets or promotional banners, enabling flexible customization.

These extensibility mechanisms ensure that the core business logic remains unaffected while allowing rich customization of the user experience.

# Example Use Case

For example, a plugin might add a new promotional banner widget to the storefront, enhancing marketing capabilities. Alternatively, a theme could change the entire layout and color scheme to align with a brand's identity. Both changes are handled within the Presentation Layer, demonstrating its role in managing UI customization.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBbnBDb21tZXJjZUFTUERvdG5ldCUzQSUzQXVtYWxpbmdhc3dhbWk=" repo-name="npCommerceASPDotnet"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
