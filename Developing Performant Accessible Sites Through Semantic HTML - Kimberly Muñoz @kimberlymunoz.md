# Developing Performant Accessible Sites Through Semantic HTML - Kimberly Muñoz @kimberlymunoz

## What is semantic HTML?
- Markup that reinforces the meaning of the page
- Machines, screenreader, humans can parse/understand/read it
- `<main>, <nav>, <heading>, <button>, <input>, <select>`

## Why accessibility?
An accessible site creates a better experience for **everyone**.

## Semantic HTML = Better Focus Management
- Makes your site navigable by keyboard in a way that makes sense.
- Excellent focus management is good UX for everyone
	- Example: skip links, visible focus

### Have your visual layout match your DOM order
- Retrofitting this via `.blur`, `.focus` is difficult
	- what happens when someone clicks then starts tabbing?
	- when someone tabs backward?
	- does your tab order code work on mobile/desktop/other breakpoints?

### Consider the `<button>`
- w3.org/TR/wai-aria-practices-1.1/examples/button/button.html
	- shows the amount of code needed just to make a `button` via `div`

### Semantic elements are flexible
- https://cfpb.github.io/capital-framework/

### Still works on slow browsers

## Offload validation and other work to the browser
### The benefits of custom inputs
#### Validation
- the browser handles validation
- require with `required`
- disable with `novalidate` on `form`
#### Custom input
- on mobile, you get a custom keyboard for the input type
- for `type="date"` you can get a browser-provided datepicker
### The downsides of custom inputs
- implementation varies from browser to browser
- not supported on older browsers. always use backup validation!
- on long forms, you might need a more robust solution
### In the future: `inputmode`

## Maintainable Modular Code
### Have you ever been in a codebase with…
- mystery classes?
- `div` bloat?

- How we bundle CSS today
	- One large site bundle
- How we might bundle tomorrow
	- smaller bundles in HTTP/2 Era

## Resources
- _Designing for Performance_ by Lara Callendar Hogan
- WAI-ARIA Authoring Practices 1.1


