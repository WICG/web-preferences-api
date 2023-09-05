# Web Preferences API

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

The **Web Preferences API** aims to solve this by providing a way for sites to indicate a user preference for a given pre-defined setting.

It is intended for this override to apply permanently and be scoped per origin. 
The override should be passed down to sub-resource where possible, see privacy section for details. This explainer refers to "site" but it should be read to mean origin.

### Goals

- Provide a way for sites to override a given user preference in a way that fully integrates with existing browser APIs
- Increase usage of these preferences, leading to a more accessible web

### Non-Goals

- Provide a way for sites to store arbitrary site-specific preferences -- local storage or other storage APIs should be used instead
- Provide a way for sites to determine the source of a user preference, beyond User Agent VS site (e.g. a site won't be able to determine if a setting comes from the OS or browser)
- Force browsers to provide a UI for overriding OS level user preferences (although this would be nice)
- Force browsers to provide a UI for overriding user preferences per site (although this would be nice)

## Use Cases

### Color scheme toggle switch

Currently, sites can use a variety of UI components to implement a per site configuration of color scheme preference.
An example of a custom element library for this is [dark-mode-toggle](https://www.webcomponents.org/element/dark-mode-toggle).

This library currently requires users to use a class to indicate the dark mode preference, rather than being able to use the preference media query.
It also contains a "hack" to allow the media attribute on `<link>` elements to work.

With the **Web Preferences API**, this library could be updated to remove the "hack" for `<link>` elements, and all limitations such as requiring a dark mode class would be removed.

### Syncing preferences across devices

Like with the previous use case, if a site wanted to sync a user's animation preference across devices, they'd currently have to remove any usages of media queries and swap to using the DOM and a CSS selector (e.g. class, or attribute).

With the **Web Preference API**, this would no longer be the case and sites could use a simple sync function on page load to ensure the server and client preference matches.

This would have the added effect of the site benefiting from any potential future UA stylesheet to reduce animations for users who have indicated a preference for reduced motion.

### Fully Themed Browser UI

Currently, if a site decides not to use `prefers-color-scheme`, they're likely also not using the `color-scheme` property to declare support for dark mode.

This likely results in having to manually theme all browser provided UI (e.g. form controls, scrollbars) for dark mode. This is a lot of work and is likely to be missed in some places.

With the **Web Preferences API**, sites could simply use the `color-scheme` property and rely on the browser to theme all browser provided UI.

## Proposed Solution

### The `navigator.preferences` object

A new `navigator.preferences` object will be added to the platform. This object will be the entry point to this API.

#### WebIDL

```webidl
[Exposed=Window]
partial interface Navigator {
  readonly attribute PreferenceManager preferences;
};

[Exposed=Window]
interface PreferenceManager {
  // null means the preference is not overridden
  attribute ColorSchemePref?  colorScheme;
  attribute ContrastPref?  contrast;
  attribute ReducedMotionPref?  reducedMotion;
  attribute ReducedTransparencyPref?  reducedTransparency;
  attribute ReducedDataPref? reducedData;
  // Future preferences can be added here, the exact properties will be down to the browser support.

  sequence<PreferenceSupportData> getSupported();
};

enum ColorSchemePref { "light", "dark" };
enum ContrastPref { "no-preference", "more", "less" };
enum ReducedMotionPref { "no-preference", "reduce" };
enum ReducedTransparencyPref { "no-preference", "reduce" };
enum ReducedDataPref { "no-preference", "reduce" };

[Exposed=Window]
interface PreferenceSupportData {
  readonly attribute DOMString name;
  readonly attribute FrozenArray<DOMString> values;
};
```

#### TypeScript

```ts
interface Navigator {
  readonly preferences: PreferenceManager;
}

interface PreferenceManager {
  // null means the preference is not overriden
  colorScheme: string | null; // "light" | "dark" | null
  contrast: string | null; // "no-preference" | "more" | "less" | null
  reducedMotion: string | null; // "no-preference" | "reduce" | null
  reducedTransparency: string | null; // "no-preference" | "reduce" | null
  reducedData: string | null; // "no-preference" | "reduce" | null
  // Future preferences can be added here, the exact properties will be down to the browser support.

  getSupported(): PreferenceSupportData[];
}

interface PreferenceSupportData {
  readonly name: string;
  readonly values: string[];
}
```

### The `navigator.preferences.[preferenceName]` properties

Each preference the browser supports will be exposed as a property on the `navigator.preferences` object.

Feature detection for a given preference is as simple as:

```js
const colorSchemeSupported = 'colorScheme' in navigator.preferences;
```

Each property will return null if not overridden or a string value indicating the preference.

```js
const colorScheme = navigator.preferences.colorScheme; // "light" | "dark" | null
```

To clear an override and return the preference to the browser default, the property can be set to null.

```js
navigator.preferences.colorScheme = null;
```

To set a preference override, the property can be set to a valid value for the preference.
If an invalid value is set then this will be a no-op.
```js
navigator.preferences.colorScheme = 'dark';
```

### The `navigator.preferences.getSupported` method

This method allows a site to get the preferences supported by the browser. This is useful for sites that want to dynamically generate UI for overriding preferences.

It also allows sites to determine if a preference value is supported before attempting to set it.

```js
const preferenceSupportData = navigator.preferences.getSupported();
console.log(preferenceSupportData); // [ { name: 'contrast', values: ['more', 'less', 'no-preference'] }, ... ]
```

## Privacy and Security Considerations

### Avoiding fingerprinting

This API exposes no new fingerprinting surfaces beyond that which already exist in the platform.


### Permissions & User Activation

This API does not require any permissions or user activation. It exposes no new capabilities to a site beyond that which already exists in the platform.

A site can already store a value in local storage and read into the DOM to override a preference. This API simply makes the ergonomics of this better.

### Iframes etc

See [#8](https://github.com/lukewarlow/web-preferences-api/issues/8) for discussion regarding this.

For the spec we can probably find an existing definition to reference, but for the purposes of this explainer:

- Any same-origin subresource (e.g. iframes) should get the overridden value.
- Any cross-origin subresource that already has communication with the parent (e.g. `postMessage`) should get the override value.
- Any cross-origin subresource with no external communication (e.g. an SVG loaded as an image) should get the override value.
- Any cross-origin subresource that has no communication with parent but can communicate externally should **NOT** get the override value.

Wherever the override value is passed down it should be done so in an opaque manner.

e.g. if the parent frame sets `colorScheme` to `dark` then the iframe should see `prefers-color-scheme` as dark but shouldn't read `navigator.preferences.colorScheme` as `dark`.

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

- Should getSupported be asynchronous? It was originally but has been changed to synchronous because it is unlikely to be a performance bottleneck and it makes the API easier to use.
- Where should the PreferenceManager interface be exposed?
    - It is currently exposed on the `navigator` object, is this best?
    - It is currently exposed even in insecure contexts, is this okay? (For the same reasons it's not user activation locked I think this is fine)
    - It is currently only exposed to Window, should it also be exposed to Service and/or Web Workers?

## Acknowledgements

Special thanks to [Ryan Christian](https://github.com/rschristian) for his help in reviewing the original explainer and providing feedback.

