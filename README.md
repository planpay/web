# @planpay/web

The `@planpay/web` module provides methods for interacting the the planpay JavaScript SDK. This greatly simplifies integration with sites using TypeScript or ES6+. This includes loading the SDK into React, Angular and vanilla JS sites. It automates the following:

- the loading of the planpay JavaScript SDK
- provides auto-complete support for your code to simplify integration
- helps to keep your integration up to date with the latest versions for the planpay widgets

> **Important:** this module is in BETA and is subject to change.

## Installation

Using NPM:

```
npm i @planpay/web
```

Using Yarn:

```
yarn add @planpay/web
```

## Basic usage

To use any of the methods, we must first initialize the planpay web components library by calling the `planpay.init({...})` method. It is then possible to call the `.refresh({...})` methods of `planpay.pricePreview` and `planpay.checkout` objects to refresh/load the price-preview or checkout widgets.

### Initializing the module

Use the following code with the provided options to initialize the widget. This will manage the load of the JavaScript SDK on to the current page.

```ts
import { planpay } from '@planpay/web';

planpay.init();
// Loads `prod` (production) SDK by default.
```

If no arguments are provided, the production SDK will be loaded. If we want to target the sandbox (`sbx`) environment, we would need to provide the following:

```ts
planpay.init({
  environment: 'sbx',
  // OR explicitly provide 'prod' for the production environment.
});
```

> **Note**: the `showDebug` property can be set to `true` to provide verbose debug logs in the browser console, which can be useful during development.

### Loading the price-preview

The price-preview widget allows merchants to include a preview of the price in search results etc. similar to the following:

> **planpay** it for A$30.00 per week

The library is agnostic of view engine and requires only that an HTML element with certain properties be included on the page. Multiple elements can be included on the same page.

```html
<div
  data-planpay-data-type="price-preview"
  data-planpay-total-cost="200.00"
  data-planpay-currency-code="AUD"
  data-planpay-redemption-date="2023-03-30"
  data-planpay-payment-deadline="60"
  data-planpay-total-minimum-deposit="20.00"
  data-planpay-merchant-id="dhj987l"
>
  Loading...
  <!-- Optional "loading" content -->
</div>
```

All values are fully configurable, other than `data-planpay-data-type="price-preview"` which must exactly match this value. The `data-planpay-merchant-id` value should match your merchant ID.

The following method must then be called after the content has loaded.

```ts
planpay.init(); // See "Initializing the module"

planpay.pricePreview.refresh();
```

This will (re-)render a planpay price-preview for all applicable markup on the page.

This method also accepts an arguments object where we set the `targetElement` value to a valid CSS selector or HTML element for the **parent** element of the price-preview markup.

```ts
planpay.init(); // See "Initializing the module"

planpay.pricePreview.refresh({
  targetElement: '.search-results',
});
```

The target element should contain all of the markup as child elements. For example, if the price-preview markup is included in each search-result item on a search-results page, then the CSS selector should be for the element which contains all of the search-result items. When this is not provided, the whole `document` will be target for updates. This option can be useful to allow selective price-preview updates after AJAX load etc.

### Loading the checkout

This widget works much like the price-preview widget. However the markup to be included is simpler.

```html
<div data-planpay-checkout-id="n1083c79a35b">
  Loading...
  <!-- Optional "loading" content -->
</div>
```

The only variable here is the `data-planpay-checkout-id`. The value of this attribute should be set with the result of the create-checkout (back-channel) API call that your server performs to the planpay API.

> **Important:** the API call to create the checkout(ID) must originate from your website's server - and not the front-end of the website - in order to protect your API key and prevent customers from creating fraudulent checkouts.

Next, call the `planpay.checkout.refresh()` method.

```ts
planpay.init(); // See "Initializing the module"

planpay.checkout.refresh();
```

This will (re-)load a planpay checkout into any elements with the `data-planpay-checkout-id` attribute.

> **Note:** we don't recommend loading multiple widgets per page as this may result in unexpected results.

## Useful integration tips

The library loads the planpay JavaScript SDK from the planpay servers on to every target page when the `planpay.init()` method is called. This helps to ensure that widget integrations keep up to date with the latest changes. However, it can create challenges with certain front-end frameworks like React.

In order to call the method only after the content has rendered, lookup the appropriate component or page event for your library. Here are some examples:

### React

Add the planpay method calls via the [useEffect(...)](https://reactjs.org/docs/hooks-reference.html) hook.

```ts
useEffect(() => {
  // planpay init and other method calls here...
}, []); // <-- empty array means 'run once'
```

### Angular

Add the planpay method calls via the [ngAfterContentInit(...)](https://angular.io/api/core/AfterContentInit) hook.

```ts
ngAfterContentInit() {
  // planpay init and other method calls here...
}
```