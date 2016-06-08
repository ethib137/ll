# Migrating to Metal from React

A few hints for migrating your React components to Metal.

## Lifecycle

|  React                      |  Metal           |  Use                                     |
| --------------------------- | ---------------- | ---------------------------------------- |
|  render                     |  render          | *Bind functions to the instance.*        |
|  componentDidMount          |  attached        | *Bind event listeners to dom nodes.*     |
|  componentWillMount         |  created         |                                          |
|                             |  detached        | *Remove event listeners from dom nodes.* |
|  componentWillUnmount       |  disposed        |                                          |
|  componentWillReceiveProps  |  sync[AttrName]  | *activeTab --> syncActiveTab*            |
|  componentDidUpdate         |  rendered        |                                          |
|  shouldComponentUpdate      |  shouldUpdate    |                                          |

## Attributes

|  React      |  Metal         |
| ----------- | -------------- |
|  onClick    |  data-onclick  |
|  className  |  class         |

Style objects work mostly the same, but you will need to explicitly pass the unit for any value whose type is a number:

```js
const styles = {
	display: 'block',
	width: 25,        // React
	width: '25px'     // Metal
};
```

## Accessing and Changing State

| React                       | Metal                       |
| --------------------------- | --------------------------- |
| this.state.x                | this.x                      |
| this.setState({x: 1, y: 2}) | this.setState({x: 1, y: 2}) |
| this.setState({x: 1})       | this.x = 1                  |

## Props and State

Both props and state will be defined on your Metal component's `STATE`. If you want to keep certain attributes private, which would be analagous to `this.state` in React, give the name a `_` suffix, like `count_`. More common practice is to put the underscore at the beggining of the name, but this is preffered as it will make certain lifecycle names cleaner (`syncCount_`).

All metal components inherit a few default state attributes. One of them is `elementClasses`, which can be used to apply a classname to the root element of a component. Use this instead of manually implementing and passing in your own className state attribute.

Configuring `defaultProps` is done on the state attribute definition in Metal. See the allowed configuration [here](https://github.com/metal/metal-state/blob/master/src/State.js#L59).

## PropTypes

Use [metal-state-validators](https://www.github.com/metal/metal-state-validators) instead:

```js
import Types from 'metal-state-validators';

...

MyComponent.STATE = {
	name: {
		validator: Types.string
	},

	count: {
		validator: Types.number,
		value: 0
   }
};
```

## Object spread syntax and passing through other props

Usually declaring every prop explicitly leads to more maintainable code, but if you are wrapping very basic components with some small extended behavior, you can use a utility to pass `other` props in Metal:

```js
import {omit} from 'lodash';

function otherProps(component) {
	return omit(
		component.config,
		component.getStateKeys(),
		'children'
	);
}

...

class MyComponent extends Component {
	render() {
		return (
			<div {...otherProps(this)}>
				...
			</div>
		);
	}
}
```