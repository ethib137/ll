<p align="center">
  <img height="60" src="/images/metal_loop_circle.png" width="60" />
</p>

# Guidelines for Metal Component Developers

Some guidelines for writing maintainable and modular components.

## Metal

### State (and Config)

All attributes that a component may be passed should be listed in it's `STATE` configuration. While they are also available as `this.config[name]`, avoid referencing them there. Instead prefer `this[name]`, since props must be declared in `STATE` to be available, ensuring they are documented.

~~The exception to this rule is `children`, which is OK to reference from `this.config`.~~
`children` will be referenced with `this.children`([issue #118](https://github.com/metal/metal.js/issues/118))

```js
import Component from 'metal-jsx';
import Types from 'metal-state-validators';

class LabeledButton extends Component {
	renderLabel() {
		/**
		 * BAD: We should be able to see all of the accepted attributes in STATE,
		 * but we cannot here since config exposes any attribute passed to the component.
		 * In other words, don't expect anything other than children on the config object.
		 */
		return <span>{this.config.label}</span>;
	}

	render() {
		return (
			<div class="labeled-button">
				{this.renderLabel()}

				/**
				* GOOD: We know that buttonText is a something we can pass
				* to LabeledButton and that it should be a string.
				*/
				<Button onClick="onClick">{this.buttonText}</Button>
			</div>
		);
	}
}

LabeledButton.STATE = {
	buttonText: {
		validator: Types.string
	},

	onClick: {
		validator: Types.func
	}
};

export default LabeledButton;

```

`STATE` is self-documenting, and should be viewed as the component's public API. Metal makes no distinction between **public** and **private** attributes, in the way that React does with `props` and `state`.

#### Naming STATE Attributes

For our project specifically, we will declare **public** and **private** like

```js
LabeledButton.STATE = {
	privateStateAttribute_: {
		value: 0
	},

	publicStateAttribute: {
		validator: Types.string
	}
};
```

Where private attributes are suffixed with an underscore and should be sorted along with public attributes. We also do not need to specifically declare any validators for private attributes since they are all internal.

#### Naming Event Handlers

Event Handlers that are declared in the STATE as part of the public API should be named `on[descriptor][eventName]`.

```js
PostHeader.STATE = {
	onDateClick: {
		validator: Types.func
	},

	onLocationClick: {
		validator: Types.func
	}
}
```

In the case of a component having only one instance of a given event type (ie. button or checkbox), the descriptor can be left out.

```js
Button.STATE = {
	onClick: {
		validator: Types.func
	}
}
```

```js
Checkbox.STATE = {
	onChange: {
		validator: Types.func
	}
}
```

Functions that are declared on the component object and will be passed into an element or component attribute should follow a different naming convention. In order to keep them from being confused with event handlers that are passed into the component they will not begin with `on` instead they will follow the following pattern:

`handle[descriptor][event]`

```js
class Post extends Component {
	created() {
		bindAll(
			this,
			'handleDateClick',
			'handleLocationClick'
		);
	}

	handleDate() {
		// do something.
	}

	handleLocationClick(event) {
		// do something with the event object.
	}

	render() {
		const {handleDateClick, handleLocationClick} = this;

		return (
			<div>
				<PostHeader onDateClick={handleDateClick} onLocationClick={handleLocationClick} />
			</div>
		);
	}
}
```
In the case of `handle` type events, an `event` name is not always required. Event names are only required if the method uses the `event` object such as `handleLocationClick` which is seen above.


If using a method in multiple places, it is best that the descriptor would describe the action

```js
class Form extends Component {
	created() {
		this.handleMinimize = this.handleMinimize.bind(this);
	}

	handleMinimize() {
		// do something.
	}

	render() {
		const {handleMinimize} = this;

		return (
			<div>
				<Button onClick={handleMinimize} />

				<div>
					<span onClick={handleMinimize} />
				</div>
			</div>
		);
	}
}
```


### Destructuring Functions on the Component Object

Metal does not distinguish between functions declared on the component object and attributes passed into the component. This allows us to destructure functions alongside attributes. This is fine to do as long as the function that has been destructured is not being called in the current component but rather being passed throught to another component or element.

For example, `handleDateClick` and `handleLocationClick` may be destructured, but `renderPostContent()` should be called direclty off `this`.

```js
class Post extends Component {
	created() {
		bindAll(
			this,
			'handleDateClick',
			'handleLocationClick',
			'renderPostContent'
		);
	}

	handleDateClick() {
		// do something.
	}

	handleLocationClick() {
		// do something.
	}

	renderPostContent() {
		// do something.
	}

	render() {
		const {handleDateClick, handleLocationClick} = this;

		return (
			<div>
				<PostHeader onDateClick={handleDateClick} onLocationClick={handleLocationClick} />

				{this.renderPostContent()}
			</div>
		);
	}
}
```

### When to create a folder for component organization.

In the instance were a group of components are so related and interdependent that you would want to put them in the same file, create a folder and add them there. This is preferred over having multiple components in a single file because it keeps the line count of files from growing too large.

One instance where this should be implemented is Radio and Radio Group. These two components will always be used together, so it makes sense to let their file structure reflect that.

```
- Components/
	- radio-group/
		- __tests__/
			- Option.js
			- RadioGroup.js
		- index.js
		- Option.js
		- RadioGroup.js
```

We will use an index in order to export all of the related components. We are also able to namespace the components as we export them. This allows us to avoid adding a namespace within the radio folder, while still namespacing it where it is used.

`/index.js`
```js
import Option from './Option';
import RadioGroup from './RadioGroup';

RadioGroup.Option = Option;

export default RadioGroup;
```

When we implement RadioGroup it will look like this:

```js
import RadioGroup from '../radio-group';

<RadioGroup checked={checked} name="testradio" onChange={handleRadioChange}>
	<RadioGroup.Option label="Option 1" value={0} />
	<RadioGroup.Option label="Option 2" value={1} />
	<RadioGroup.Option label="Option 3" value={2} />
</RadioGroup>
```