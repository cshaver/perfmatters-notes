# Accelerated Mobile Pages: The good, the bad and the ugly - Vitor Taliaia
#PerfMatters

## What is AMP anyway?
- Accelerated Mobile Pages

### The problems that AMP is trying to solve
- average mobile page takes 19s to load
	- 53% of users will abandon the page after 3s
- frozen navigation/scrolling
- flash of unstyled content

### How it works
- AMP HTML Spec
	- superset of HTML
	- HTML that includes custom AMP features through web components
- AMP JS library
	- a script to implement APM perf best practices
- MAP Cache
	- servers that deliver AMP content
	- they fetch the AMP pages, cache, optimize, and validate them

- (code sample)
	- `<amp-img>`
- link to non-AMP version of document from AMP doc, and v/v

## The Good
- 3rd party JS runs in a sandbox (in an iframe)
- prioritize the content of the first viewport
- static layout sizing
	- helps to avoid content shifting
	- amp enforces all assets to have their size defined
- Add more functionality as you need it
	- `<script custom-element="amp-audio" ...>` `<amp-audio>`

### Not AMP-only
> It’s not so much about what AMP does; it’s more about what it doesn’t allow.  
> – Jeremy Keith  

## The Bad
- Where’s my page being served from?
	- `google.com/…` what? 
- Constrained design
- “Google seems to be more worriedd about chasing facebook/instagram features than speeding things up”

## The Ugly
- “Even more centralization of content on huge companies like Google and Facebook”
- Google kept saying that AMP wouldn’t affect search ratings…
	- AMP is the only way to appear on the top search carousel and receive the lightning bold sign
- http://ampletter.org/
- Fake news can get away without branding - since AMP layouts all look the same
	- it can be hard for readers to discern between credible and less-than-credible sources

## The “seems to be” Good
- “Improving URLs for AMP pages”
- “Standardizing lessons learned from AMP”

> Let’s incentivize the goal, not the tool.  
> – Tim Kadlec, “The Two Faces of AMP”  