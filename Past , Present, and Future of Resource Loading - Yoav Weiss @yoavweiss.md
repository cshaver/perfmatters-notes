# Past , Present, and Future of Resource Loading - Yoav Weiss @yoavweiss
#PerfMatters

## Past
- `time != (bytes / BW)`
- Latency - dominated overall performance more than bandwidth
- Things got better
	- better fast, computers
	- also more slow computers with  mobile devices
	- speed of light hasn’t changed
- content of web evolved significantly over the years
	- full-fledge application development on the web
	- Images, styles, scripts, video, audio, etc etc

## Problems
## Why is loading slow?
### Connection Establishment
- First thing browser does is 
- inherently latency-bound
- hostname -> ip address (DNS request/response)
- TCP connection established between client/server, 3-way connection
- SSL does its own thing
- then finally HTTP GET request

### Server-side processing

### Slow start
- server doesn’t know how much bandwidth it can send without overwhelming the network
- mechanism that enables server to gradually send more and more data to feel the network out
- also limits the amount of data we can send initially
- 14kb, ramping up

### Discovery
- Finding which other resources it needs to download
- tokenization, preloaded, figure out which resources it needs
	- doesn’t cover all the resources, some come after calculating styles and executing scripts
- critical rendering path -> download HTML to create DOM
- blocking scripts prevent the DOM from being created
- CSSOM together with the DOM to create render tree, which creates layout, and then paint can happen
- Browsers have priority for different types of resources
	- HTML > CSS > JS > Fonts > Images
	- In practices, browsers have implemented more complex logic
	- HTML -> CSS (-> CSS), JS (-> JS), IMG resources 
	- Dependencies!
	- Results in crazy-ass bandwidth graphs with holes in them
		- browser doesn’t know it needs to download more resources because it hasn’t discovered it yet

### Queueing
- HTTP/1.1 - each connection can only handle a single request at a time
- multiple resources to download - only some (2-6) per RTT
	- this is why many requests is an anti-pattern with HTTP/1.1
- analogy with terrible security line, someone not prepared blocks everyone

### Contention
- lower-priority resources slowing down higher-priority resources
- happens in HTTP/1.1 by definition
- not solved in HTTP/2 either

### Bloat
- Resource size definitely still matters


## Today’s Solutions
### Queuing
- HTTP/2 to the rescue! (aka “H2”)
	- enables you to send multiple strings, corresponding to request/response pair
	- multiple strings on a single connection, many requests in parallel
	- airport security line analogy
		- multiple parallel stations, first one ready goes ahead
	- doesn’t magically fix everything
		- can’t just download everything in parallel
		- still network bandwidth-bound
- H2 priorities
- H2 doesn’t fix everything
	- Priority responsiveness
		- responsiveness to priorities on the server-side
		- effectively having two queues
			- priority h2 queue
			- TCP queue in the kernel
		- hard for server to respond to priority changes or resource cancellations
			- results in lower priority resources coming down before higher priority ones
	- Packet loss HOL
		- more vulnerable to this than h1
		- TCP connection, reliable protocol
			- any loss stalls all communication until that loss is recovered
			- push down head of line blocking problem from application layer to transport layer
			- h2 performs worse than h1 in some cases
		- x-resource compression
			- cannot compress multiple files together as if they are one
			- bundling resources is a best practice for hhtp1/1 to reduce number of requests
			- hoping to get rid of that for h2 but lack of shared compression contexts means theres still some benefit to bundling resources together

## Tomorrow’s

- how to fix? QUIC protocol, by Google
	- moving the entire stack from transport to security to http layer to the user space 
		- implementations can have a tighter contnrol over what they send and when they send it
	- handles packet losses differently, providing differently reliability guarantees than TCP
		- each one of the strings delivered in order
		- other responses not impacted if one of the packets were to go missing
	- being standardized as part of the ITF, to provide a reliable stable version
- h2 compression dictionaries
	- compress multiple resources together as if they were one resources
	- some security hurdles to making that happen
- alternative to that is better bundling => Web packaging porposal
	- enables us to bundle resources together but also maintain them as individual resources that can be processed as such
	- significant performance benefits
- server side processing
	- traditional advice: flush early, send something to the browser as soon as you can
		- this is problematic in some stacks because if your DB call fails, the entire doc is impacted. not always trivial to switch document once you started sending data down
	- we can use the server side thinking time to send other resources down => **h2 push**
		- put network connection to good use while we wait for the server
		- turn bandwidth graph from hot mess to `…/`
		- also enables us to ramp up the 14k limit with slow start before we get the HTML
		- ramp up the connection early with other assets
	- “http/2 push is tougher than I thought”
		- may be pushing resources unnecessarily
		- cache digests proposal to fix that
			- giving the server a condensed list of resources in cache, server can decide which resources need to be pushed
- resource discovery
	- preload
		- `<link rel="preload" ...>` - decouples download from execution
		- browser doesn’t need to run js and css in order to find the resources it would have found later
		- important part of that is priorities
			- `<link rel="preload" as="script">` -> lets the browser figure out prorities
	- priority hints - early proposal
		- influence the browser’s heuristics regarding a resource’s priority
		- a script may not be a critical resource, 
- connection establishment
	- preconnect
		- `<link rel="preconnect">` - tell the browser to connect ahead of time to 3rd-party domain, get that connection out of the way of the critical path for that resource
	- connection coalescing
		- h2 allows this
		- requires TLS certificate will cover both hosts, DNS both resolve to the same IP (?)
		- avoid multiple competing connections at the same time
	- other ways to improve?
		- TCP fast open
		- TLS 1.3
		- QUIC
		- Alt-Src over DNS
- slow start & contention
	- same solution - reduce the number of connections we need to use
	- Unshard your domains - H2 we don’t need this
	- non-credentialed connections
		- big contributor to multiple connections
		- font downloads, es6 modules, etc
		- causes a lot of bandwidth contention issues
	- secondary certificates
		- enables server to coalesce connections onto with a shared certificate
- bloat
	- Load _only_ what you need
	- Use responsive images
	- Brotli
		- advancement in compression, supported in all modern browsers
		- 30% smaller files with same compression speed as gzip
	- Future: Compression API
		- native browser compression/decompression abilities to create smarter compression schemes in the browser

## What do I actually need to do?
- HTTP/2
	- significant improvement
- Use push, preload, and preconnect
	- make sure your bandwidth is always utilized
	- use them carefully
- Unshard your domains and coalesce connections as much as possible
- Load only the resources you actually need
- Future improvements underway!
	
