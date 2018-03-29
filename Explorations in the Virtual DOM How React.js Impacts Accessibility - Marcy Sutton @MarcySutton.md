# Explorations in the Virtual DOM: How React.js Impacts Accessibility - Marcy Sutton @MarcySutton

## New JavaScript Hotness syndrome
## What made React.js different
- Virtual DOM
- Component lifecycle
- One-way data flow
- Timing in framework land
- Developer ergonomics

## Accessibility in React Apps
### Virtual DOM

#### axe-core & Virtual DOM <3
github.com/dequeuelabs/axe-core/blob/develop/doc/developer-guide.md
- APIs for virtual DOM, for testing a11y

#### Component Lyfe
- `componentDidMount`, etc. lifecycle hooks super useful

### HTML in JS
- github.com/davidtheclark/react-aria-menubutton (accessible react components)
- a little controversial

#### POUR me a coffee
w3.org/WAI/WCAG21/Understanding
- Perceivable
- Operable
- Understandable
- Robust

#### Plugging WAI-ARIA Authoring Practices 1.1 again
- JS developers don’t need to reinvent the wheel

### User-input events
- Virtual DOM
- Dynamic HTML w/ JS
- Multiple contexts

#### That’s a job for: Synthetic Event Delegation
- `SyntheticEvent` is a wrapper around React’s Event System

#### Binding Events in React
- bind inline `onClick={this.handleClick}`

#### `SyntheticEvent` things to watch out for:
- `SyntheticEvent` is a wrapper utility
- Bind to keyboard-accessible widgets
- Capturing & non-React events

### Focus management
- Probably the biggest thing to benefit from learning when working with JS apps, so many scenarios not being handled for focus
- smashingmagazine.com/2015/05/client-rendered-accessibility
- Layers/Modals
- Sidenavs
- Tabs
- Menus

#### There are multiple focus management strategies
- User input event + focus on refs
- assign focus via component state callback
- focus on component mount/update* (carefully)
- Use context (recent API changes)
- Focus layer system (smarter)

#### Code Examples
- facebook.github.io/react/docs/refs-and-the-dom.html
- github.com/facebook/react/issues/1791
- Attest DevTools Extension

### Testing and Accessibility
- Unit tests
- Integration tests
- Tools

#### React Unit Tests
- Rendering with props/state
- Interaction APIs
- Snapshot testing - look at state tree and see if things change

##### Shallow Rendering
Lets you render a component “one level deep” and test without worrying about child components. Does not require a DOM.
github.com/davidtheclark/react-aria-menubutton

##### Tests requiring more than Shallow DOM
- Computed name and role
- Color contrast
- Visible focus state
- focus management with refs
- ARIA support

##### Mounted rendering
-attaching code to the DOM
- This is where `enzyme` comes into play
- bit.ly/a11yHelper

##### Other Unit Test Considerations
- Test individual units
- JSDOM platform gaps
- Events on inaccesible elements

###### The dreaded `documentElement` error
- Testing for a11y in a shallow test environment, no real browser
- bit.ly/jsdom-helper
	- gives you some hooks to tell JSDOM “here’s `window`, here’s `document`”

#### Integration Tests
> Frontend bugs have occurred with boundary between components or layout of our app – for example between the API and the data layer, the layout and our components, or between a parent and child component.

##### Testing with axe-webdriverjs
bit.ly/a11y-demo-app

##### A11y Testing APIs
- axe-core
- axe-webdriverjs
- React-axe
- eslint-plugin-jsx-a11y
- React-a11y
