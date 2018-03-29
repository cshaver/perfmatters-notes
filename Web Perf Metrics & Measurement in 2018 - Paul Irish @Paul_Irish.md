# Web Perf Metrics & Measurement in 2018 - Paul Irish @Paul_Irish
#PerfMatters

## When is load?
- shrug/
- Page load cannot be captured by a single metric
	- is it happening?
	- is it useful?
	- is it usable?
	- is it done?
	- is it delightful?

## let’s meet the family of metrics!
### visual metrics
lets take all the frames of load and break them apart

#### first paint
- the first paint after navigation
- includes basically any paint
- includes background paints
- it can fire when `body { background: #fff }` is painted

#### first contentful paint
- first paint of content
- content includes
	- text, images (includes background images), canvas, SVG

#### first meaningful paint
- the primary content is visible 
- kind of hard to define
- marks the paint that follows most significant change to layout
- # of layout objects added
	- we chart these, cumulatively
	- we look at the largest jump and call that FMP
- defined in bit.ly/ttfmp-doc
- another trick: handling webfonts
	- fuckin’ invisible text
	- if there are pending web fonts (and text is currently invisible) FMP waits for those web fonts to finish loading
	- excludes any web fonts that paint <200 characters (icon fonts)
- consideration: FMP is sensitive to small changes

#### visually complete
- visually, the page is done loading
- considerations: modal pops and image carousels wreak havok

#### speed index
- how quickly does the page approach visually complete?
- two film strips:
	- A `...xxxxXXXX [visually complete]`
	- B `........... [visually complete]`
	- defined at sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index
	- oftentimes a unitless number
		- speed index is essentially the time it took for the average pixel to paint on the screen
		- basically millisecond values
- but how do we determine the % of visual completion
- speed index (compare via color palette)
- perceptual SI (compare via luminance, contrast, structure (SSIM), math)
- SI & pSI analysis
	- highly correlated
	- pSI is a slightly tougher critic
	- pSI punishes for elements moving around (within the viewport)
		- e.g. an image comes in and pushes all the content down
	- considerations:
		- both unfairly reward pages with non-white backgrounds
		- pSI does moreso
		- both are completely thrown by modals & carousels

#### hero rendering times
- blog post from speedcurve with 3 different metrics, and another with a composite metric
- “last painted hero”
- the last paint of critical content
	- largest `<h1>` or largest text in viewport using final webfont
	- largest `<img>`, if none, use largest `background-image`
	- LPH is max of those two

### interactivity metrics
- if we can’t measure something we can’t improve it

#### Estimated Queuing Time
- summarize the main thread impact on user IO during that time
- RAIL - respond in under 50ms

#### Time to Interactive
- the load is finished. the main thread work is done
- aka “consistently interactive”
- requires
	- network to be fairly quiet
	- long tasks complete
- defined in Time to Interactive Explainer and Time to Interactive v1.0 (which also defines Network Quietness)

#### First CPU Idle
- main thread’s first snooze after FMP
- aka “first interactive”, naming is hard
- Defined in Time to First Interactive - Task Cluster Definition

#### First Input Delay
- queuing time of the first input of the page
- designed for collection in a RUM scenario
- defined at First Input Delay. RUM impl at tdresser/first-input-delay
- Protip: focus on the 99th percentile

## Top priority metrics
1. Time to Interactive
2. Speed Index
3. First Contentful Paint

## Measuring
In the lab and in the field

### Lab data
- dive in see what’s happening
- not loading the page under the same conditions
- devtools, webpagetest, Lighthouse

### Field Data / RUM
- real world
- happened in the past
- hard to diagnose
- not a lot of detail
- Longtask API
	- `PerformanceObserver`
	- measure input delay & latency: `performance.now() - event.timeStamp`

write code -> test in the lab -> release to users -> validate via RUM -> write code …
