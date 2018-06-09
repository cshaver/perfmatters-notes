# Demystifying Performance Timers - Jeremy Wagner @malchata

## Regarding Testing Methods
### Synthetic Testing (or _lab data_)

- E.g. Lighthouse in Chrome, looking around in the Network panel, sightspeed.io if you’re fancy
- It’s still synthetic!

#### For
	- it’s easy!
	- great for spot-checking perf in dev (with network throttling)
	- easy to measure the effect of code changes
#### Against
	- doesn’t reflect what real users are experiencing
	- narrow view of performance for our application

### RUM (Real User Metrics)
#### For:
	- metrics from real users using your site/app
#### Against
	- not as convenient

## Navigation and Resources Timing
- **Navigation Timing:** Gathers page metrics
- Resources Timing: Gathering resources metrics (page-dependent resources, CSS, JS images, etc.)

### Processing model
[w3.org/TR/navigation-timing-2/](w3.org/TR/navigation-timing-2)

#### Unload
- any time you kick off navigation, the current doc has to be unloaded
- it could be cancelled (hit escape, close the browser, “are you sure you want to leave this page?!”)
- metrics are pretty simple:
	- `startTime` - beginning of the entire nav-timing phase: always 0
	- `unloadEventStart` - maybe 0 if no previous doc, or if previous is not from same origin (timing allows origin headers)
	- `unloadEventEnd` - can overlap with other events (e.g. connect, redirect), because browser’s trying to do things as efficiently as possible

#### Redirect
- When a navigation encounters one or more HTTP 30x status codes
- chase our tail if too many redirects
- insecure context -> secure context -> another redirect -> insecure -> secure
- phases
	- `redirectStart`
	- `redirectEnd`
	- resource timing API processing model _begins_ here
	- both 0 if no redirects occur
	- redirects from and to other origin s can still come as 0 if `Timing-Allow-Origin` header isn’t set (allows other origins to collect resource and timing metrics)

#### App Cache
- when the browser begins to fetch an item
- whether a browser needs to fetch something from the cache
- `fetchStart` - regardless of caching, this begins about the same time
- if an item is cached, metrics that come after (e.g. `connectStart`, `responseEnd`) will occur very nearly after

#### DNS
- when a domain name is resolved to an IP address, can take some time
- `domainLookupStart`
- `domainLookupEnd`
- DNS lookups can be cached at multiple levels:
	- Browser
	- System
	- Router
	- ISP
	- _et al._
- Extremely small or 0 DNS lookup times could be from a cache

#### Connection
- When TCP handshake occurs, TLS negotiation time
- `connectStart`
- `secureConnectionStart` - when your TLS negotiation is occurring
- `connectEnd`
- connections can be reused (`Keep Alive`, HTTP/2 which streams resources over one connection)
- `secureConnectionStart` will be 0 for non-secure sites
	- can also be 0 if TLS sessions are resumed

#### Request/Response
- when requests are made and responses occur
- TTFB (time to first byte): content-download time
- `requestStart` - denotes the start of the request
- `responseStart` - start of response
- `responseEnd` - end of response
- no request end, not as significant as the entire process
- Resource timing API processing model _ends_ here

#### Processing
- DOM is being processed; takes time to break down the document, tokenize HTML tags, discover other resources, populate the DOM
- `domInteractive`
- `domContentLoadedEventStart`
- `domContentLoadedEventEnd`
- `domComplete` - everything is loaded, all page-dependent resources
- interactive and domContentLoadedEventStart occur around the same time
- depending on frontend-architectural complexity; domComplete often occurs after domContentLoadedEventEnd

#### Load
- when the page has loaded everything
- `loadEventStart` - usually occurs briefly after domComplete
- `loadEventEnd` - page has utterly and completely finished loading

#### little bits
- `name` - URL of a document or resource
- `transferSize` - size of resource _including headers_
	- a little bit to a lot of data, depending on cookies or other headers
	- size on disk is size of potential response body, compression notwithstanding
- `encodedBodySize` - compressed size of a resource (without headers)
- `decodedBodySize` - the decompressed size of a resource (without headers)
	- 200kb JS compressed could be like a damn MB, browser has to compile and parse before it can execute. Maybe this is something you want to look at, CSS resources as well
- `nextHopProtocol`  - protocol used to transmit a resource
	- `h2` (HTTP/2)
	- `hq` (QUIC)
	- `http/1.1`
	- _this is probably not exhaustive!_

#### Timing recipes!
- (pseudocode)
- Fun, simple arithmetic, but useful!
- TTFB (ms) - `responseStart - requestStart`
- Resource fetch time (ms) - `responseEnd  - fetchStart`
- Fetch speed (Mbps) might not be entirely accurate; rough idea of throughput - `((transferSize / (2**20)) / ((responseEnd - responseStart / 1000)) * 8`
- Header size (bytes) - `transferSize - encodedBodySize`
- DNS lookup time (ms) - `domainLookupEndStart - domainLookupEnd`
- TCP time (ms) - `connectEnd - connectStart` inclusive of SSL time
- SSL time (ms) `connectEnd - secureConnectionStart` when secureConnectionStart > 0
- DOM-processing time - `domInteractive - domComplete`

#### Bonus materials!
- Server Timing
	- passes server-side metrics to navigation timing
	- [w3c.github.io/server-timing](w3c.github.io/server-timing)
	- what server-timing headers look like:
		- [name of metric]; dur=[duration; desc=“[description]”, [etc.]
		- server-timing headers are visualized in Chrome when present in headers, with human-readable labels
		- NEAT
- Paint Timing API
	- [w3.org/TR/paint-timing](w3.org/TR/paint-timing)
	- Try it in the console* (*only in chrome)
		- `performance.getEntriesByType('paint')`
		- `first-paint`, meaningful or not, `first-contentful-paint` (usually the same, still in dev, kind of new)
- Long Tasks API
	- activity that blocks the main thread for a long period of time
	- “Long Task refers to an event loop that exceeds 50ms”
	- observer has to be initialized first
	- `entryTypes: ['longTask']`

#### Quick Tips!
- Don’t record computed metrics _(unless you know what you’re doing)_
- Always feature-check this stuff
	- `if ('performance' in window) { /* ... */ }`
- gather performance metrics with `PerformanceObserver`
	- long loops could be blocking the main thread
