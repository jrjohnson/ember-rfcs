---
stage: released # FIXME: This may be recommended
start-date: 2020-08-25T00:00:00.000Z
release-date: 2022-05-02T00:00:00.000Z
release-versions:
  ember-source: v4.4.0

teams:
  - framework
prs:
  accepted: https://github.com/emberjs/rfcs/pull/659
project-link:
---

# {{unique-id}} helper

## Summary

Add a new built-in template helper `{{unique-id}}` for generating unique IDs.

See [pre-RFC issue #612](https://github.com/emberjs/rfcs/issues/612)

## Motivation

When working with HTML it is very common to need to create and reference [DOM IDs that are unique within the HTML document](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id). Classic Ember components provide the `elementId` attribute which can be used to construct unique ids within classic components, but `elementId` is not available within Glimmer components or route templates.

There are several common use cases where a developer may need to generate a unique ID for use in a template:
1. Associating `label` and `input` elements using the label's `for` attribute and the input's `id` attribute.
2. Using WAI-ARIA attributes to improve accessibility (eg. `aria-labelledby`, `aria-controls`)
3. Integrating 3rd party libraries that attach themselves to DOM elements using DOM IDs (eg. maps, datepickers, jquery plugins, etc)

Since providing some faculty for generating unique IDs for DOM elements can reasonably be considered a requirement for most Ember apps wishing to implement an accessible UI (via labelled inputs and/or WAI-ARIA), it is reasonable for Ember to provide this functionality at the framework level. Ember already provides the `guidFor` utility in javascript, so it is reasonable for Ember to provide similar functionality within templates.

## Detailed design

Add built-in `{{unique-id}}` template helper.

### `{{unique-id}}`

The `unique-id` helper can be invoked with no arguments. When invoked this way, the `unique-id` helper will return a new unique id string for every invocation.

In practice this invocation style would usually be paired with a `let` block to enable re-use of the unique id generated by `{{unique-id}}`.

```hbs
{{#let (unique-id) as |emailId|}}
  <label for={{emailId}}>Email address</label>
  <input id={{emailId}} type="email" />
{{/let}}

{{#let (unique-id) as |passwordId|}}
  <label for={{passwordId}}>password</label>
  <input id={{passwordId}} type="password" />
{{/let}}
```

In the future, an inline or template version of `let` could enable a single invocation of `{{unique-id}}` for re-use of the id within a template.

### Implementation

The `{{unique-id}}` helper can be implemented using the existing mechanisms Ember uses for generating unique ids. An implementation should ideally be designed to prevent id collisions among other ids generated by Ember.

## How we teach this

### Ember API docs: Ember.templates.helpers

The Ember API docs can be updated to include the `unique-id` helper on the page for [Ember.templates.helpers](https://api.emberjs.com/ember/release/classes/Ember.Templates.helpers)

> Use the `{{unique-id}}` helper to generate a unique ID string suitable for use as an ID attribute in the DOM.
>
> ```hbs
> <input id={{unique-id}} type="email" />
> ```
>
> Each invocation of `{{unique-id}}` will return a new, unique ID string. You can use the `let` helper to create an ID that can be reused within a template.
> ```hbs
> {{#let (unique-id) as |emailId|}}
>   <label for={{emailId}}>Email address</label>
>   <input id={{emailId}} type="email" />
> {{/let}}
> ```


### Ember Guides: Associating labels and inputs
The Ember guides currently include a section on associating labels and inputs. This section can be updated to use this new `{{unique-id}}` helper.

[Guides: associating labels and inputs](https://guides.emberjs.com/release/components/built-in-components/#toc_ways-to-associate-labels-and-inputs)

> Every input should be associated with a label. Within HTML, there are several different ways to do this. In this section, we will show how to apply those strategies for Ember inputs.
>
> You can nest the input inside the label:
> ```hbs
> <label>
>   Ask a question about Ember:
>   <Input type="text" @value={{this.val}} />
> </label>
> ```
> You can associate the label using for and id:
> ```hbs
> <label for={{this.myUniqueId}}>
>     Ask a question about Ember:
> </label>
> <Input id={{this.myUniqueId}} type="text" @value={{this.val}} />
> ```
>
> In HTML, each element's id attribute must be a value that is unique within the HTML document. Ember provides the built-in `{{unique-id}}` helper to assist you with generating unique IDs.

> ```hbs
> {{#let (unique-id) as |myId|}}
>   <label for={{myId}}>
>     Ask a question about Ember:
>   </label>
>   <Input id={{myId}} type="text" @value={{this. val}} />
> {{/let}}
> ```
>
> The `aria-label` attribute enables developers to label an input element with a string that is not visually rendered, but still available to assistive technology.
> ```hbs
> <Input id="site" @value="How do text fields work?" aria-label="Ember Question"/>
> ```
>
> While it is more appropriate to use a <label> element, the `aria-label` attribute can be used in instances where visible text content is not possible.

### Accessibility guides
This helper will be an important part of Ember's out-of-the-box accessibility story. Future improvements to Ember's accessibility guides will be able to use this helper when discussing how to build forms and how to work with WAI-ARIA attributes such as `aria-controls` or `aria-describedby`.


## Drawbacks

Adding new helpers increases the surface area of the framework and the code the core team commits to support long term.

There is nothing about this proposal that could not be instead implemented in an add-on.

## Alternatives

1. Do nothing; developers can use backing classes for templates that require an ID, and either use `elementId` in classic components or import `guidFor` in glimmer components or via a hand-rolled helper.

2. Introduce a keyword-style syntax that leverages a build-time AST transform to convert this:
```hbs
<label for="{{unique-id}}-toggle">Toggle</label>
<input id="{{unique-id}}-toggle" type="checkbox">
```
into
```hbs
{{#let (unique-id) as |_id|}}
  <label for="{{_id}}-toggle">Toggle</label>
  <input id="{{_id}}-toggle" type="checkbox">
{{/let}}
```
This approach is implementable in userland or in add-ons, so it may not be appropriate to consider this alternative for the core primitive introduced into Ember.js itself.

This approach may be slightly more ergonomic but relies on a more magical, non-standard keyword-like API that will need to be specifically taught, and will be a larger maintenance burden.

## Unresolved questions

none
