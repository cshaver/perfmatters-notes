# Progressive Web Apps: Show Up for Offline First, but Stay for the Killer Performance Boost - Jason Lengstorf @jlengstorf
 #PerfMatters

git.io/v9tEr

## How much does your website **cost**?
- We measure cost in seconds
- Time isn’t the only cost
- $2.05/MB AT&T data travelling
- Websites are getting larger
- Also: loading huge websites on mobile just SUCKS
- What if the connection is flaky?
	- unstable connections are framed as emerging markets
	- but jfc, getting on BART is a flaky connection

## WE CAN  FIX IT!
(and it doesn’t mean starting from scratch!)

## The solution: progressive web apps
We can:
- significantly decrease the amount of data transferred on each page load
- improve the experience for users on unreliable connections
- “that’s not my target market”
	- high end devices, metropolitan areas, blah blah
	- booooring, like being told to eat your vegetables
	- can’t we just ignore PWAs? no, smh

### Here’s why
- PWAs decrease load times on all connections
- perceived load times improve
- stability improves for mobile users everywhere
- user experience is better all around

## real-world examples
- lengstorf.com
	- cold: ~4s 477kb
	- warm: ~1s 0.5kb
- csswizardry.com
	- cold: 1.5 339kb
	- warm: 1s 1.5kb

### What makes a PWA?
- web-based application
- web manifest
	- JSON file
	- tells the phone / chromebook / whatever that it’s web app capable and can be installed on a phone screen
	- “hey you come here a lot do you wanna install this” 
- service worker
	- installed locally on the users browser, acts as a proxy between the web app and the internet, catches things along the way
	- when you first install, it downloads critical assets and puts them in the users cache
		- recommended to cache the app shell (header, footer, shared scripts)
	- pre-cache non-critical assets
		- do work in the background that doesn’t affect the user experience
	- acts as a proxy for all outgoing requests
		- e.g. listen for images, if image isn’t available, you can put a little “hey that image is missing” image in ts place
			- also nefarious things, don’t be a dick
		- also if cached, you can skip the server for this entirely
			- this is *instant*
			- this allows PWAs to load offline on subsequent visits (!!) (this works on csswizardry.com)

some bad news:
- service works alone are not enough to solve all the problems of all the perf
	- you have to download all the content all at once at first
	- only useful on repeat visits
- what can we do to make first visit really fast?
	- implement route-based code splitting
	- use http/2
	- set up background prefetching
	- lazy load any non-critical assets

- “PRPL Pattern” - bit.ly/prpl-pattern
	- push (http/2 push)
	- render
	- precache (anything possible)
	- lazyload (anything not critical)

+ let’s add two to PRPL
	+ eliminate all unused code
	+ use server-side rendering to generate and serve static files

You don’t have to do all or any of these! but it’s useful to know how this works and to weigh the trade-offs of doing it yourself vs tools

### route-based code-splitting
- creates smaller bundles for specific routes
- creates shared bundle for common code
	- e.g. react, etc. on every page
- allows loading only what’s _really_ needed
- code splitting with webpack (probably the most common way of doing this)
- use `import()` to identify chunks
- in Webpack 4, use `optimization.splitChunks`
	- 3 or earlier, use `CommonsChunkPlugin`

### serving assets with http/2
- multiplex your downloads
- provides push capabilities to minimize requests

### asset prefetching
- assume which routes the user will load next
	- Gatsby, react static site generator playing with using ML and markov chains to experiment with this
- loads the assets for those routes in the background
- loading the next route is pretty much instantaneous

### lazy loading assets
- prevents images from blocking initial load
- only requests them once they’re in the viewport
- “won’t lazy loading make the page jump?”
	- we can fix it!
	- loading animation: `responsize-lazyload.js`
	- medium uses a blur-up technique
	- lazy loading with traced SVGs (sharp plugin with gatsby.js)
	- simplified polygons (gatsby.js)

### eliminating unused code
- reduces the overall size of the codebase
- helps avoid module bloat
	- figuring out where the dead code is is really hard at places like IBM
	- garbage dump people from Labyrinth movie
- use Uglify.js to remove dead code
- enable tree-shaking with ES modules
	- e.g. only using deep merge from lodash, throw away the rest of it
- Webpack Bundle Analyzer
	- accidentally shipping all locales of moment.js
	- find lots of little nonsense pieces

### Serving only Static Assets
- Removes the need for runtime server rendering
- Eliminates an extra HTTP round-trip
- Improves cacheability
- “but it’s haaaaard”
	- trade-offs: high effort / but user has to deal with that crap
	- loading 1MB of JS before anything cad be displayed :( :( :(
- SSR so no JS is required to initially display the page
	- original intent was to make SPAs seo-crawlable
- “but static files won’t work for my app!
	- yes they will. (probably.)
		- pretty much anything now can be built client-side
		- that means you don’t actually need a server
		- you can do that with static files!
		- you’re not giving anything up by doing this

## Overwhelming?
- Performance is hard
- we can get most of these performance improvements FOR FREE
	- open source is AWESOME
- how to improve web perf for free:
	- sw-precache and/or sw-toolbox
		- helper libs from Google team
		- sets up sane defaults for Service Workers
		- requires manual configuration
	- Next.js + Now
		- Next.js
			- framework for server-side rendering React
			- Automatic code splitting, optimizations for routing
			- Requires a server
		- Now
			- hosting for Node, static, and Docker applications
			- purpose-built for Next.js apps
			- amazingly simple cli tooling
	- Gatsby.js + Netlify
		- Gatsby.js
			- framework for building React apps
			- compiles to static files
			- Add Service Worker + Web Manifest with plugins
			- Generated apps are unbelievably fast
		- Netlify
			- deployment and hosting for static sites
			- powerful git workflow integrations
			- automatic, free SSL generation

## Let’s Recap
- PWAs are the future
	- moving into a world where we’re starting to see companies like Apple stay the leaders at innovation
	- Microsoft is coming back, no longer a roomful of Macs
	- not the monoculture, we need to be ready for that
	- not plausible to expect every company and developer to build a native and web app etc
	- support shipping on surface books, rumors of Windows, Apple working with SWs
	- we’re gonna get there to where you can ship PWAs to every device
	- web as a universal platform!
- blazing fast apps need *fine tuning*
	- we can’t just hope and pray that people like at mozilla solve our shitty code problems for us
	- we need to pay attention to these techniques

### PWA perf checklist:
- [ ] add a Service Worker
- [ ] create a Web Manifest
- [ ] Follow the PRPL Pattern
- [ ] Remove unused code
- [ ] Generate static files

#### Use frameworks to automate perf best practices
- no reason to reinvent these wheels
- OPEN SOURCE IS DOPE
- extreme edge case? open a PR!

#### Spending time on perf is good for everyone
- lower cost to pay-per-MB users
- better experience on mobile and low-powered devices
- improved reliability on spotty connections
- apps feel more “solid” on all devices
- repeat visitor see damn-near instant page loads

## When we focus on performance, everybody wins!
