# LL CSS Guidelines

- JS Component = .scss file
- Each .scss file should have one container only.
``` css
  .component-container {
    // All styles should be within this container.
  }
```
- Components are only allowed to style the containers of other components. They should not style elements inside of the container.
  - These should only be layout related styles.
- Use includes for all styles that bourbon has mixins.
- Component variants should be styled within that component by passing in a classname from the parent component.
- Uses sass variables to consistently apply design patterns across multiple selectors.
  - i.e. $cardPadding: 24px;
  -      $theme: #BB66CC;
  -      $cardBoxShadow: 0 2px 8px #FFF;
  - Variables specific to a component should be declared within a component.
  - Variables that may be used by multiple components should be declared in _variables.scss.
