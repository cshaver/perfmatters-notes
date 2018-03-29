# There’s more to performance than meets the eye - Léonie Watson @LeonieWatson
#PerfMatters

## Screen readers
- do cool stuff
	- e.g. pronunciation with and without correct lang attribute
	- Late 90s, screen readers consumed HTML directly from the browser and create a virtual model of the content
		- two user agents, caused problems
	- early 00s, screen readers using accessibility APIs to query information
		- E.g. disclosure widget, `summary` element
			- not implemented in Edge, just text
			- Get visual and keyboard interactivity with Firefox
				- screen reader can query a11y tree, current state communicated
	- late 10s
		- all screen readers use a11y APIs to query info, but not all create virtual model of the content
- single process model vs multi-process model
	- in-process
		- screen reader injects itself into the application process and communicates directly
	- out-of-process communication
		- IE is relaxed in how it approaches - screen reader injects itself into application process and has direct access to the content process
			- API calls made directly into the content process
			- these APIs are really chatty
			- Fast TTI and subsequent perf, but prone to instability
	- multi-process: chrome & firefox
		- let the screen reader exist inside the application process, but does not have direct access to the content process
			- Chrome: initial approach
				- proxied a11y api calls from the application process into the content process
				- thousands of calls needed to create virtual buffer, distinct perf hit
			- Chrome: current approach
				- caches a copy of the a11y tree in the application process
				- possible slow TTI, but fast subsequent performance
				- once cache has happened, perf is superb. noticeable delay with particularly large page
				- video: 4s delay on w3.org wai-aria 1.1
			- Firefox: approach
				- proxies a11y api calls from the application process into the content process, but uses intelligent caching
				- Firefox makes guesses about what other info might be needed based on that initial call
				- good TTI and subsequent perf, but a11y perf issues introduced with Quantum remain unresolved (intelligent caching not quite right)
	- multi-process: edge
		- screen readers cannot inject into the application process, and do not have access to the content process
		- no caching, all a11y API requests are handled via the OS
		- fast TTI, but possible slow subsequent performance
		- out-of-process calls are 12 to 50 times slower than in-process calls, making a performant virtual model impossible
	- Edge: Narrator
		- Narrator is integrated into the OS, and does not use a virtual model, so is more performant
	- Safari: VoiceOver
		- VO is also integrated into the OS, and does not use a virtual model, so is performant

## Performance: drop the virtual model?
- screen readers depend on the virtual model to improve human performance
- demo: selecting text with and without virtual model
	- shift + down arrow until everything selected (ugh)
	- dropping a marker at beginning and end of desired content
- dropping virtual model means we need to make this more efficient

## Performance: improve accessibility APIs
- a11y APIs need the same features that screen readers have in the virtual model
- need better ways of bundling information, e.g. “give me this container and everything in it”
- demo: searching with UIA (newer of APIs on Windows platform)
	- doesn’t allow you to search for content within an iframe

- demo: how it really is
	- enabling human performance
	- when it all comes together, capable screen reader users are pretty damn fast
