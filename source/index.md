---
title: itembase API Developer Documentation
 
language_tabs:
  - shell
  - java
  - php


toc_footers:
  - <a href='mailto:rk@itembase.biz'>Sign Up for a Developer Key by email</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Welcome

Welcome to the itembase API developer documentation! Our API is designed to make it easier for e-commerce players to connect to each other. This first version of our API gives you unified, cross-platform access to the e-commerce data of your client applications. This means that instead of developing separate clients for each e-commerce platform you need access to, you can simply connect to itembase's access layer in order to retrieve the data. In other words, itembase deals with anything platform-specific leaving you free to focus on what you know best: your own service.

Our API and access layer are built using modern web-standards such as REST, OAuth 2.0 and HTTPS. The guides below provide you with all the information you need to get started and begin developing. If you are just looking around, you can always take a look at our sandbox to get an intuitive feel for what the API can offer. If something is unclear or you need additional information, just fire a mail to our support team. We're always looking for ways to make developing with itembase easier and hassle-free and are always ready to help. Happy coding!


# Steps to get started
## 1. Register your application by email. 
Before you can begin developing your client, you have to tell us who you are. Just send our support team a short email saying that you would like to integrate your application with itembase. We will need to know the re-direct URLs for your application. These are the URLs that itembase will re-direct to after we authenticate users of your application. You can find out more about the authentication  and authorization flows in the sections below.
  

When you register your application itembase will send you:

  * credentials for your client application to allow it to connect to itembase's access layer, (a client id and secret), 
  * access credentials for a sample user so you can play around in our sandbox environment. 

In addition, registering your application allows our product and dev teams to start planning when and how it will be offered on the itembase solutions platform. 

## 2. Use our sandbox environment for development and testing of your client application.

Using the credentials for your client application along with the sample user access tokens we send you once you register with us, you can start playing with the sample data in our sandbox environment. We have prepared some Swagger 2.0 API documentation where you can try out sample API requests in your browser. You can take a look at the data entities section for short descriptions of the different types of e-commerce data available from the itembase API.


## 3. Implement an OAuth flow in your application in order to enable your service in a production environment. 

Your client has to be able to handle an OAuth flow before it is ready to be released in a production environment. This is a must have, as it will be necessary for users to grant your application access to their data via the itembase API.


# Authentication and authorization



## Outline: Integrating an OAuth flow in your client.

You will have to integrate an OAuth flow into your client application before you can release it in a production state. This flow is required as it allows a user to authorize your application to access their data using the itembase API. In order to do this properly, you will need to follow the steps below. The guides below assume that you follow the `authorization_code` grant type / flow. Other flows work according to the [OAuth 2.0 RFC specification](https://tools.ietf.org/html/rfc6749).
:

   1. Select the scope of your client application (that is the data access level you require).
   2. Initiate authorization in your front-end via a button.
   3. Redirect the user to the itembase accounts server to complete the authorization on our side.
   4. Request access and refresh tokens from itembase.

## Step 1: Select the scope of your client application.

A scope describes which resources your client application will be able to access and in what way. During the authorization process whatever scopes you have chosen for your application will be shown to the user in a readable form. The table below shows which scopes are currently supported by the itembase API.

|Scope item|Readable form|Description|
|---|---|---|
|user.minimal|Read general user information|Information includes the user id, their email address etc. This scope is needed for the /me call after authorization and should always be included|
|connection.transaction|Read transactions from connected platforms|Get access to transaction data of connected shops|
|connection.product|Read product data from connected platforms|Get access to product data of connected shops|
|connection.profile|Read profile data from connected platforms|Get access to specific account data of a connected shop|
|connection.buyer|Read customer information from connected platforms|Get access to customer data of a connected shop|


You can select more than one `scope item` to include in the scope of your application. To allow you can provide a space separated list of scope items. The example below would allow your client application access to the itembase APIs that return basic user nformation and transaction data for user-defined shop connections:

``user.minimal connection.transaction``

## Step 2: Initiate authorization in your front-end via a button.

You need to integrate a button in the front-end of your system that the user can click in order to connect their itembase account with your client application. When the user clicks this button, you will need to redirect them to the authentication URL of our accounts server. You will need to include the following as parameters in this URL:

   * The client id you received from itembase when you registered your application.
   * The scope of your application.
   * A re-direct URI to which itembase will re-direct the browser after authentication on our server.

## Step 3: Re-direct the user to the itembase accounts server

Your front-end button should re-direct to the itembase accounts server in order to complete the authorization. The complete URL should include the parameter information from Step 2, above. 

``https://accounts.itembase.com/oauth/v2/auth?response_type=code&client_id=your_client_id&scope=user.minimal&redirect_uri=http://your.service/connect/itembase&grant_type=authorization_code'
``  

<aside class="notice">
The above contains placeholders. You must replace them with real parameter data.
</aside>

After the redirect to the itembase accounts server, the itembase user will be authenticated. Depending on the login status of the user, they may need to enter their itembase username and password. If they do not yet have an itembase account they may be asked to create a new account. Account creation and login happens completely on our side.


Once the user is authenticated successfully as an itembase user, they will see a website that asks them to grant or refuse your client application access to their user data on itembase. The dialogue they see lists all scopes you have requested.

If the authorization is not successful (e.g. the user does not accept your request or perhaps an error occurred), the itembase accounts server redirects to your redirect_uri with an error response:

``http://your.service/connect/itembase?error=access_denied``

If the user grants access to client, the itembase accounts server re-directs to the  ``redirect_uri`` you have specified and will include an authorization code (parameter code):

``http://your.service/connect/itembase?code=1234someauthcode``

This code remains valid for some seconds and can only be used once. You will need to provide this code in order to obtain access and refresh tokens from itembase, as outlined in Step 4 below.

## Step 4. Obtaining tokens.

### Initial token request

With the authorization code you can request an access token and a refresh token the first time. This request goes to the accounts server's token endpoint and must include the secret of your client application:

``https://accounts.itembase.com/oauth/v2/token?client_id=your_client_id&client_secret=your_client_secret&grant_type=authorization_code&code=1234someauthcode&redirect_uri=http://your.service/connect/``

<aside class="notice">
The above contains placeholders. You must replace them with real parameter data.
</aside>

The ``redirect_uri`` will be ignored in the authorization_code flow, but **still has to be given**. The ``redirect_uri`` parameter must be one that is allowed by itembase (e.g. the one from the authorization call). The accounts server will return the access and refresh tokens to you as a JSON response.

>Sample JSON response containing access and refresh tokens.


```json
{
    "access_token": "MjU5MGM0YjJkZmIyZDZmZmE3NGQwZGZkMzYxNTBhYjA2M2Vj",
    "expires_in": 2592000,
    "token_type": "bearer",
    "scope": "user.minimal",
    "refresh_token": "ODk3YjA5MjM3YzNjMzQ1NjY5NDZiZGZjMDI2ODQ1NazZ2Vj"
}
```

The access and refresh tokens are associated with the single specific user for whom you requested data access. You need to store these tokens and use them when requesting data for that user from the itembase data access layer. Both the access and refresh tokens expire after a certain time:

|Token type | Expires after |
|---|---|
|Access token | 1 day |
|Refresh token | ~ 34 days |in the response bodyYou will receice a JSON response that returns your tokens:


### Using the refresh token to request a new access token

Every time the access token for a user expires, you will need to request a new one using the refresh token. This process happens in a similar way to the initial request above:

``https://accounts.itembase.com/oauth/v2/token?client_id=your_client_id&client_secret=your_client_secret&grant_type=refresh_token&refresh_token=your_refresh_token#``


<aside class="notice">
The above contains placeholders. You must replace them with real parameter data.
</aside>

The accounts server will return a JSON response containing your new tokens that replace the old ones.

>Sample JSON response containing new access and refresh tokens.

```json

{
    "access_token": "MjU5MGM0YjJkZmIyZDZmZmE3NGQwZGZkMzYxNTBhYjA2M2Vj",
    "expires_in": 2592000,
    "token_type": "bearer",
    "scope": "user.minimal",
    "refresh_token": "ODk3YjA5MjM3YzNjMzQ1NjY5NDZiZGZjMDI2ODQ1NazZ2Vj"
}

```

**IMPORTANT** You need to obtain a new access token before the corresponding refresh token expires. If the refresh token expires, you will need to repeat the authorization / authentication process.

### Obtaining a UUID for your user

Most calls to the itembase API are user specific (e.g. your client will request data for a specific user via our API). In order to make these calls, you will need to provide the itembase user id for that user. To get the UUID for a user associated with a particular access token, you can make a request to the following endpoint including the accesstoken in the request header. Of course, you must have included ``user.minimal`` as part of your scope for the request to be successful:

``Request (GET):
https//users.itembase.com/v1/me

Header:
Authorization: Bearer your_access_token_here
`` 

>The response has basic information for the user, including the user uuid.


```json

{
    "uuid": "a4b91ee7-ec1a-49b9-afce-371dc8797749",
    "username": "thommy",
    "email": "tb@itembase.biz",
    "first_name": "Thomas",
    "middle_name": null,
    "last_name": "Bretzke",
    "name_format": "first middle last",
    "locale": "en",
    "preferred_currency": "EUR"
}

```

We strongly recommend that you store the access token, refresh token and the uuid associated with a particular user securely on your server. You can create a corresponding user on your side with matching information if it is appropriate for your purposes.


# Environments, API URLs, message formats and rate limits

## Environments
Versions of all our services exist in two environments: a sandbox for exploration, development and testing; and a production environment for live systems. The documentation always primarily refers to the production environment although it is usually generally applicable to the sanbox as well. 

## API Urls

|Type|Production|Sandbox|
|---|---|---|
|OAuth2 auth URL|[https://accounts.itembase.com/oauth/v2/auth](https://accounts.itembase.com/oauth/v2/auth)|http://sandbox.accounts.itembase.io/oauth/v2/auth|
|OAuth2 token URL|https://accounts.itembase.com/oauth/v2/token|http://sandbox.accounts.itembase.io/oauth/v2/token|
|User information endpoint|https://users.itembase.com/v1/me|http://sandbox.users.itembase.io/v1/me|
|Activation endpoint|https://solutionservice.itembase.com|http://sandbox.solutionservice.itembase.io|
|API endpoint|	https://api.itembase.io (coming soon)|	http://sandbox.api.itembase.io|

## Transport and message formats

Transport in the production system is over HTTPS. We use JSON as the message format in all cases. 

## API Rate Limits

In the production environment we enforce rate limits on a per client_id basis. You can contact our support team if you need more information about this or require a greater limit. At present API calls to the sandbox API are unlimited. 


# Data Entities

The itembase API is designed to provide a unified, cross-platfrom access layer that lets you fetch, query and process e-commerce data using a single client application. We distinguish between four different types of e-commerce data, which we term data entities, listed below.

## Transaction
 
The **Transaction** entity models an e-commerce purchase and contains the following fields:

 * `id` The unique identifier for the transaction in the itembase data storage.
 * `source_id` The unique identifier for the transaction on the source e-commerce platform.
 * `original_reference` The retailer's order reference for the transaction.
 * `created_at` DateTime when the transaction was created.
 * `updated_at` DateTime when the transaction was updated (if at all).
 * `currency` The currency of the transaction (ISO-4217 currency code).
 * `total_price` The total cost of the transaction, including any taxes and charges.
 * `total_price_net` The total cost of the purchases in the transaction, excluding taxes and charges.
 * `total_tax` The amount of sales tax or VAT incurred.
 * `status` The status of the transaction (e.g. shipped).
 * `products` An array of itembase Product entities (see below), that is the item or items purchased.
 * `shipping` The shipping address and recipient.
 * `billing` The billing address.

## Product

The **Product** entity models any item that can be purchased from an online retailer. Product entities contain the following fields: 
       
 * `id` The unique identifier for the product in the itembase data storage.
 * `source_id` The identifier for the product on the source platform. 
 * `original_reference` The retailer's original reference for the product. 
 * `url` The URL to the product listing on the e-commerce platform. 
 * `created_at` DateTime when the product listing was created. 
 * `name` The title of the product. Contains sub-field indicating the language the title is written in.
 * `active` Whether the listing for the product is active or not. 
 * `condition` E.g. new, second-hand etc.
 * `currency` The currency the product is listed under (ISO-4217 currency code).
 * `price_per_unit` The price of a single product unit.
 * `tax_rate` The percentage tax that applies to the price_per_unit.
 * `tax` The calculated tax for the product.
 * `identifier` A list of identifiers e.g. SKU, ASIN.
 * `stock_information` Relevant stock information (e.g. in stock, the inventory level etc).
 * `description` Textual description of the product.
 * `categories` Array of categories to which the product belongs (e.g. Home and Garden, Outdoors).
 * `picture_urls` Array of URLs linking to images of the product.

## Buyer

This entity models a person, an organisation or a company capable of making a transaction from a retailer. Buyer entities contain the following fields: 

 * `id` The unique identifier for the entity in itembase data storage. 
 * `source_id` The identifier for the entity on the source e-commerce platform. 
 * `original_reference` 
 * `status` The status of the buyer (e.g. confirmed).
 * `first_name` The given name of the buyer.
 * `last_name` The family name of the buyer.
 * `language` The language associated with the buyer (e.g. DE).
 * `locale` The default language associated with the buyer (e.g. de_DE, en_EN).
 * `currency` The default currency the buyer usually buys in (ISO-4217 currency code).
 * `contact` Contact details for the buyer (contains the lists: `addresses` and `emails`).
 * `created_at` DateTime for when the buyer entry was created on the platform.

## **Profile**

This entity models the profile of a retailer on some e-commerce platform. Profile entities contain these fields:
 
 * `id` The unique identifier for a profile in the itembase data storage. 
 * `source_id` The unique identifier for the profile on the source e-commerce platform. 
 * `platform_id` The e-commerce platform identifier. 
 * `platform_name` The name of the e-commerce platform.
 * `status` The status of the profile on the e-commerce platform (e.g. confirmed, pending, etc). 
 * `display_name` The name of the retailer displayed on the e-commerce platform. 
 * `language` The language of the profile (e.g. DE / EN / ES).
 * `locale` The default locale for the retailer (e.g. de_DE).
 * `currency` The default currency the retailer sells in (ISO-4127 currency code).
 * `contact` Contact details for the retailer. 
 * `created_at` DateTime when the retailer profile was created. 



