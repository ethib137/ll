## CSS

### One File Per Component

In the same way that we should try to keep our javascript logic modular by organizing it into components, we should do the same for the related styles:

```
- Components/
	- Button.js
	- Avatar.js
- Styles/
	- Button.scss
	- Avatar.scss
```

### Namespace Component Styles

Component styles should always be namespaced under a selector with the suffix `-container`:

```scss
.image-viewer-container {
	/* Styles for the image-viewer-component */
}
```

Make sure to declare variables, mixins, or any other type inside of this selector so that they are scoped to that component.

### Component Variations

To provide different styles for a component, you can define classes that can be added in addition to the `-container` class. These class names should always be prefixed by the component name:

```html
<div class="avatar-container avatar-large"></div>
```

### Nested components

Only a component's `-container` may be referenced from other component style declarations. If you feel the need to reach into and target any sub-elements of a component, you should stop and re-evaluate. Perhaps you need to instead add another variation to that sub-component:

```scss
.comment-container {
	/* This is fine */
	.avatar-container {
		margin: 24px;
	}

	/* BAD: Don't do this */
	.avatar-container .name {
		color: red;
	}
}
```

Styles applied in this way will most commonly be things related to layout (i.e. margin, display).

### Vendor Prefixes

Use [Bourbon](http://bourbon.io/docs/) as a guide for deciding whether or not you need to use a mixin for the style you are applying. If there is a mixin for a style, use it:

```scss
.avatar-container {
	@include transition(background-color 0.4s ease);

	border-radius: 50%; /* Were good since they don't provide a mixin */
}
```

### Use Variables

If two styles are meant to have the same value, make sure to extract it into a variable. [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) still applies here, as usual. However, don't do this just because two styles happen to have the same value.

```scss
.comment-container {
	$contentMargin: 0 0 24px;

	/* The next two elements should always have the same margin */
	.name {
		color: green;
		margin: $contentMargin;
	}

	.text {
		margin: $contentMargin;
	}

	/* This element just happens to have the same value, no semantic relation */
	.avatar-container {
		margin: 0 0 24px;
	}
}
```

If you have global values shared between multiple components, make sure they are named well to avoid conflicts:

```scss
/* _globals.scss */

$baseBoxShadow: 0 0 4px 4px rgba(0, 0, 0, 0.2);
$cardTitleHeight: 24px;

/* BAD: This name is easily clobbered by later declarations */
$padding: 4px;
```