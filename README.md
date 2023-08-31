# Web Preference API (name TBD)

Authors: [Luke Warlow](https://github.com/lukewarlow)

## Status of this Document

This document is intended as a starting point to engage standards bodies in developing a solution to the problem listed below.

## Introduction

Currently, website authors have a choice when wishing to honour a user's preference for a given setting:

They can choose to "use the platform" where the user must indicate their preference via their OS or, if lucky, they can override in the browser. This comes with a number of issues:
- Relies on the user's OS or browser offering the ability to change the setting
- Relies on the user knowing how to change the setting in their OS or browser
- No ability to override the setting for a specific site
- No ability to sync preferences for a site across devices

Alternatively, sites can and do offer site-level settings, but this currently comes with a number of issues:
- No integration with CSS preference media queries
- No integration with conditional resource loading (e.g. using `<source media="(prefers-contrast: more)">`)
- No integration with JS APIs for retrieval of these preferences (e.g. `matchMedia`)
- No integration with [User Preference Client Hints](https://web.dev/user-preference-media-features-headers/)
- No integration with the `color-scheme` CSS property
- The various client storage mechanisms that could store these preferences can be cleared in a number of scenarios

The **Web Preference API** aims to solve this by providing a way for sites to indicate a user preference for a given pre-defined setting.

It is intended for this override to apply permanently and be scoped per origin. (This explainer refers to "site" but it should be read to mean origin).

### Goals

- Provide a way for sites to override a given user preference in a way that fully integrates with existing browser APIs
- Provide a way for sites to determine the user's preference for a given setting, without having to resort to `matchMedia`
- Increase usage of these preferences, leading to a more accessible web

### Non-Goals

- Provide a way for sites to store arbitrary site-specific preferences -- local storage or other storage APIs should be used instead
- Provide a way for sites to determine the origin of a user preference, beyond User Agent VS site (e.g. a site won't be able to determinate if a setting comes from the OS or browser)
- Force browsers to provide a UI for overriding OS level user preferences (although this would be nice)
- Force browsers to provide a UI for overriding user preferences per site (although this would be nice)

## Use Cases

### Color scheme toggle switch

Currently, sites can use a variety of UI components to implement a per site configuration of color scheme preference.
An example of a custom element library for this is [dark-mode-toggle](https://www.webcomponents.org/element/dark-mode-toggle).

This library currently requires users to use a class to indicate the dark mode preference, rather than being able to use the preference media query.
It also contains a "hack" to allow the media attribute on `<link>` elements to work.

With the **Web Preference API**, this library could be updated to remove the "hack" for `<link>` elements, and all limitations such as requiring a dark mode class would be removed.

### Syncing preferences across devices

Like with the previous use case, if a site wanted to sync a user's animation preference across devices, they'd currently have to remove any usages of media queries and swap to using the DOM and a CSS selector (e.g. class, or attribute).

With the **Web Preference API**, this would no longer be the case and sites could use a simple sync function on page load to ensure the server and client preference matches.

This would have the added effect of the site benefiting from any potential UA stylesheet to reduce animations for users who have indicated a preference for reduced motion.

### Fully Themed Browser UI

Currently, if a site decides not to use `prefers-color-scheme`, they're likely also not using the `color-scheme` property to declare support for dark mode.

This likely results in having to manually theme all browser provided UI (e.g. form controls, scrollbars) for dark mode. This is a lot of work and is likely to be missed in some places.

With the **Web Preference API**, sites could simply use the `color-scheme` property and rely on the browser to theme all browser provided UI.

## Proposed Solution

### The `navigator.preference` object

A new `navigator.preference` object will be added to the platform. This object will be the entry point to this API.

```ts
interface Navigator {
  readonly preference: PreferenceManager;
}

interface PreferenceManager {
  setOverride(name: string, value: string): Promise<void>;
  clearOverride(name: string): Promise<void>;
  get(name: string): Promise<PreferenceResult>;
  getSupported(): Promise<PreferenceSupportData[]>;
}

interface PreferenceResult extends EventTarget {
  readonly value: string;
  readonly isOverride: boolean;
  onchange: ((this: PreferenceResult, ev: Event) => any) | null;
}

interface PreferenceSupportData {
  readonly name: string;
  readonly values: string[];
}
```

### The `navigator.preference.setOverride` method

This method allows a site to override a given user preference. This method will:
- Resolve when the preference has been successfully overridden.
- Be rejected if the operation is not successful. It'll reject with a [`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException) value of:
  - `NotSupportedError`: If the preference is not supported by the browser.
  - `ValidationError`: If the provided value is not valid for the given preference.
  - `OperationError`: If the operation failed for any other reason.

```js
await navigator.preference.setOverride('prefers-contrast', 'more');
```

### The `navigator.preference.clearOverride` method

This method allows a site to clear an override for a given user preference. This method will:
- Resolve when the preference has been successfully cleared.
- Be rejected if the operation is not successful. It'll reject with a [`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException) value of:
  - `NotSupportedError`: If the preference is not supported by the browser.
  - `OperationError`: If the operation failed for any other reason.

```js
await navigator.preference.clearOverride('prefers-contrast');
```

### The `navigator.preference.get` method

This method allows a site to get the current value of a given user preference. This method will:
- Resolve with an object containing the current value of the preference, along with whether this is a site override.
- Be rejected if the operation is not successful. It'll reject with a [`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException) value of:
  - `NotSupportedError`: If the preference is not supported by the browser.
  - `OperationError`: If the operation failed for any other reason.

```js
const preferenceResult = await navigator.preference.get('prefers-contrast');
console.log(preferenceResult.value); // 'more'
console.log(preferenceResult.isOverride); // true
preferenceResult.addEventListener('change', () => {
  console.log(preferenceResult.value); // 'less'
});
```

### The `navigator.preference.getSupported` method

This method allows a site to get the preferences supported by the browser.

```js
const preferenceSupportData = await navigator.preference.getSupported();
console.log(preferenceSupportData); // [ { name: 'prefers-contrast', values: ['more', 'less', 'no-preference'] }, ... ]
```

## Privacy and Security Considerations

### Avoiding fingerprinting

This API exposes no new fingerprinting surfaces beyond that which already exist in the platform.

## Alternative Solutions

### Use a custom media query

A site could hypothetically use the as yet unimplemented [Custom Media Queries](https://drafts.csswg.org/mediaqueries-5/#script-custom-mq)
this way they could define custom media queries for each preference they wish to override.

This spec is currently in the early stages of development, and it is unclear if it will ever be implemented or in what shape it will take.
For example, would this allow a site to use a custom media query inside the media attribute of a `<link>` element?

This also doesn't solve the key issue of third party libraries being aware of this preference override.

### Request that browsers provide a UI for overriding preferences per site

While this would be a nice solution, the lack of any such UI in any browser makes it unlikely that this will happen any time soon.

This also doesn't fix the (relatively minor) issue of preference syncing across devices.

## Open Questions

- Do we need a clearOverride method? Could we just use setOverride with a value of `null` or `undefined`?
- Do we need a clearAllOverrides method?
- Do we need a way to get the preference value or is using `matchMedia` sufficient? (I think we need at least a getOverride method)
- Do we need an API method to indicate the accepted values for a given preference?
- Do we need a user activation requirement for set and clear? (e.g. clicking a button)
  - This would remove the ability to automatically sync preferences.
- Do we need a permission grant? Or at least integrate with permissions policy?
- Do we need configuration for the scope of these overrides?
- Do we need configuration for choosing session vs permanent override?

## Acknowledgements

Special thanks to [Ryan Christian](https://github.com/rschristian) for his help in reviewing this explainer and providing feedback.

