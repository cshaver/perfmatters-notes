# Getting Up to Speed with Performance with Tim Kadlec

[#PerfMatters: Getting Up To Speed](https://perfmattersconf.com/workshop/)
[Getting Up to Speed with Performance @ #perfmatters - Google Docs](http://tinyurl.com/yalfa9d9)

## Performance impacts the bottom line
- Walmart and Amazon found that -100ms => +1% revenue,
- Yahoo, Firefox, etc big perf gains => conversions
- Etsy - +126kb, and Google + 500ms => -25% searches, took a long time to return to normal
- Netflix saved 43% bandwidth bill for turning gzip on (why was it off though?!)

[WPO Stats](https://wpostats.com/) - case studies showing impact
[Time Is Money - O’Reilly Media](http://shop.oreilly.com/product/0636920041450.do)

## Where do websites come from?
- **Network** -> HTML, JS, CSS
- Needs these to build **DOM** and **CSSOM**, and JS can mess with both of these
- Takes DOM and CSSOM -> **Render Tree**, **Layout**, **Paint**

- Latency - how long it takes, ms (pipe length)
- Bandwidth - how much can get through at a time (pipe width)
- [It’s the Latency, Stupid](http://www.stuartcheshire.org/rants/latency.html) - bandwidth is cheap, but you’re stuck with bad latency
- Example RWC -> New York
	- Speed of light in a vacuum would be about 14ms
	- Best case actually fiber, round trip 43ms
	- `ping 66.111.3.74` -> anywhere from 66 to 108ms
	- Bounces matter, onto cable, etc., `traceroute 66.111.3.74`
```
traceroute to 66.111.3.74 (66.111.3.74), 64 hops max, 52 byte packets
 1  10.133.120.1 (10.133.120.1)  3.914 ms  2.180 ms  2.360 ms
 2  10.128.255.17 (10.128.255.17)  1.060 ms  2.003 ms  1.773 ms
 3  172.20.129.1 (172.20.129.1)  2.410 ms  5.850 ms  3.603 ms
 4  207.62.234.131 (207.62.234.131)  2.500 ms  2.252 ms  6.126 ms
 5  137.164.32.198 (137.164.32.198)  4.726 ms  4.236 ms  5.962 ms
```

- Bandwidth impact for a website doesn’t impact much beyond 2-3Mbsp
- Latency has a direct, linear impact on page load time

## Lets zero in on an individual connection
- DNS Lookup
	- Browser, OS, router, ISP DNS caches, recursive search
	- chrome://histograms/dns
	- chrome://dns
- TCP Connection
	- 3-way handshake to establish connection to server
		- client sends `SYN` pack
		- server sends back a `SYN ACK`
		- client sends `ACK`, acknowledge that they acknowledge
		- back and forth to ensure we can reliably send this data
- SSL Negotiation
	- Like TCP Connection, there is some back/forth
		- ClientHello
		- ServerHello, Certificate,
		- ClientKeyExchage, ChangeCipherSpec, Finished
		- ChangeCipherSpec, Finished
	- Communicating using same language, cipher levels
- Request
	- `GET /file (30kb)`
	- TCP slow-start - make sure we’ve got the bandwidth before trying to use it
	- Typically `10x TCP Segments (14600 bytes)` (this is a server configuration)
	- `10x ACK`
	- `20x TCP Segments (29200 bytes)`, `20x ACK`, etc warming up
	- Next time it picks up where it left off, now that we know the connection is there

## What can we do?
### CDN

Content duplicated on an edge server closer to the client

### Use `Keep-Alive`

- The connection is kept open so that the next request to the same server picks up where it left off, we don’t have to go through the back and forth again

### Resource Hints

#### `dns-prefetch`
- `<link rel=“dns-prefetch” href="//host_name_to_prefetch.com" />` -> I’m going to get something from this domain, do the DNS lookup ahead of time

#### `preconnect`
- `<link rel="preconnect" href="//host_name_to_prefetch.com" />`
- Same scenario as `dns-prefetch`, but on steroids. Common scenario is a font or 3rd-party resource somewhere. Does the whole connection: DNS lookup, TCP Connection, SSL Negotiation – potentially saving 300-400ms by having the browser do this upfront
- There’s no reason not to do both `dns-prefetch` and `preconnect`, progressive enhancement at its finest!
- Biggest gain is when these are for dynamically inserted assets, or assets further down in the page. If the resource is right next to the prefetch, it won’t really get you any wins
- Browser should prioritize these intelligently. Called hints for a reason - the browser still gets to decide whether it should actually open the connection if you’re on a shitty network, etc. Can prioritize the connections to ensure you’re not competing with other higher-priority connections
- Prioritizing these explicitly isn’t necessarily supported currently.

#### `preload`
- `<link rel=“preload” href=“…” />`
- 	can also be done dynamically to preload say, when user hovers something via JS
- technically not a “hint” - the browser is going to preload what you tell it to preload, you do want to be a little careful about what you’re telling the browser to bring in
- preload supersedes dns-prefetch, preconnect

#### `prefetch`
- `<link rel="prefetch" href="/style/other.css" as="style" />`
- similar to preload, but about the next page request
- we know the full path to the asset but we don’t need ti yet
- do all that stuff as before and keep it in the cache for a bit
- done at such a low priority that it doesn’t compete with the current page load, it’s possible that it won’t happen by the time the person navigates away
- `as=“style”` is needed because it can’t verify based on the extension that you’re not doing something weird here. Giving it a hint ahead of time is helpful so that the browser can make some decisions right away. e.g. JS and CSS has a higher priority in the browser, so it may initiate that preload or prefetch first. As opposed to blindly requesting everything, waiting for the server come back, so there’s no chance to do any of that smart prioritization.

####  `prerender`
- `<link rel="prerender" href="..." />`
- Pass it the full page - as if the browser opens up a hidden tab, parses everything, creates the entire page. When you actually go to the page it’s nearly instantaneous.
- In theory this is awesome; in practice, this is absolutely a foot-gun.
- Browsers have moved away from this. Some discussion on what should replace it.

## DOM, CSSOM, and JavaScript
- Creating the DOM
	- Parsing HTML, creating DOM tree. No idea what is displayed, what styled, etc., only what exists
- Creating the CSSOM
	- Selectors with different styles that apply to them
- CSS is render-blocking
	- If browser created DOM *then* applied CSS, it would be jarring and crummy
	- There are exceptions to the rule, because the browser can be clever
	- Example:
```html
<!-- this is render blocking -->
<link rel="stylesheet" href="style.css" />
<!-- this is NOT render blocking -->
<link rel="stylesheet" href="print.css" media="print" />
<!-- this might be render blocking -->
<link rel="stylesheet" href="large.css" media="(min-width: 30em)" />
```

- JavaScript is parser-blocking
	- can manipulate both DOM and CSSOM, so browser assumes the worst and blocks **everything**

```html
<link rel="stylesheet" href="/styles.css" /> // stops rendering
<script type="text/javascript" src="jquereactgular.js"></script> // won't execute the JS until CSS is downloaded, parsed, and executed
```

- CSS blocks JS execution, JS execution blocks DOM
	- If this pattern is repeated multiple times, we’re telling the browser it has to stop and repeat this process every time
	- `async`
		- IE8: [GitHub - filamentgroup/loadJS: A simple function for asynchronously loading JavaScript files](https://github.com/filamentgroup/loadJS)
		- Downloading and parsing is in parallel, so we reduce the gap a little bit
	- `defer`
		- “I don’t need you to do anything with the JS until the DOM is done”
		- Download the JS In parallel, won’t execute the JS until the DOM is complete
		- Browser creates DOM, CSSOM, moves on with its life, and JS is executing at the end
		- essentially promises not to do anything like appending to the DOM

### Let’s talk about SPAs

“[Andy & BIll’s Law](https://en.wikipedia.org/wiki/Andy_and_Bill%27s_law)”: what Andy gives, Bill takes away

> ...as bandwidth grows, and as processing power grows, as browsers get better we just keep filling everything up.
> — Jeff Veen

> …25% of new Android phones have only 512MB of RAM.
> — Jen Fitzpatrick, VP of product management for Google Maps

![](Getting%20Up%20to%20Speed%20with%20Performance%20with%20Tim%20Kadlec/A632169E-8E60-48F3-894A-385921046739.png)
[An Introduction to Speculative Optimization in V8 | Benedikt Meurer](http://benediktmeurer.de/2017/12/13/an-introduction-to-speculative-optimization-in-v8/)

> …and reduce differences in performance across browsers.
> — [Improving performance on twitter.com](https://blog.twitter.com/engineering/en_us/a/2012/improving-performance-on-twittercom.html)

- SPA architecture moves all the typical server tasks to the browser.
- Frameworks are great but can be a foot-gun, e.g. ajax response -> render on screen into DOM can have a lot of overhead
- A lot of companies using shared architecture
	- render things on first load from server
	- client can kick in and take over from there
	- this isn’t a fool-proof fix either
		- some companies are doing a useless server view, broken until client kicks in, “painting of the page”
		- React has isomorphic applications, Vue.js has nuxt, Angular rewrite for SSR
	- Netflix saw 70% faster startup times from doing this

### Render Tree
- Applies your stylesheets, user stylesheets, main default stylesheets
- Lines all this up to create a render tree
- `display: none;`  -> pull out of render tree

#### Next steps
- **Layout** - figuring out the geometry of these elements (not color!)
- **Paint**
JavaScript still manipulates these steps
- width property
	- read, being geometry related, triggers the layout step again so that the browser knows it’s accurate
	- writing that property triggers layout again
	- tips
		- batch reads/writes together
		- read once, do rest of loop

#### “Critical Path”
- We want this as low as possible
- Initial render: anything that stands in the way of initial render is critical
- Going back to what we just did
	- Necessary javascript in the head, with async attribute
	- Critical CSS
		- [GitHub - filamentgroup/criticalCSS: Finds the Above the Fold CSS for your page, and outputs it into a file](https://github.com/filamentgroup/criticalCSS)
		- certain styles apply to initial view - “above the fold” and basic styles, hopefully in that 14kb range
		- loading these inlined and minimized
		- the rest of the css can load asynchronously + set a cookie to know that it’s in cache
		- [GitHub - filamentgroup/loadCSS: A function for loading CSS asynchronously](https://github.com/filamentgroup/loadCSS)
	- Now we’re back to one critical resource, the HTML
- With just the optimization we talked about so far, content can get painted much, much earlier. Example 12.5s -> 3.9s

## Fonts
- “Worst cool thing to hit the web ever”
- Set of glyphs for anything that needs to be represented
- Size can be massive, based on:
	- complexity of glyphs
	- number of glyphs
```css
@font-face {
	font-family: "My Font";
	font-weight: normal;
	src: "...";
}
```

- The browser makes the request to pull the font into the page at the **Render Tree** step, before **Layout**
- “Use not, waste not” (unless it’s an older browser)
- You could have a custom font where it is never applied on smaller screens, and prevent the browser from requesting the font

### Two basic approaches:
	- FOUT: flash of unstyled (fallback) text
	- FOIT: flash of invisible (you see NOTHING until font is loaded)
	- [FOIT vs. FOUT](https://www.zachleat.com/foitfout/)
- Most browsers have default behavior that is mixed: invisible, but show fallback after 3 second timeout
	- IE has 0 second timeout
	- Very opinionated topic
- `font-display` property
	- Controls 3 things:
		- font block - how long to block
		- font swap - how long to swap (e.g. after 10 second delay, don’t bother switching the font out{
		- font failure - when to give up
	- Values:
		- `font-display: auto;`
		- `font-display: fallback;` _Tiny block (100ms) then show fallback. Small swap (prevents weird flicker if font is cached)._
		- `font-display: block;` _Block 3s or so, then display fallback. Infinite swap._
		- `font-display: swap;` _Like `fallback` but with no blocking period at all. Infinite swap._
		- `font-display: optional;` _Like `fallback`, but browser can decide not to load._
	- Support is minimal, but progressive enhancement for browsers that do.
- Minimize variants
	- E.g. Open Sans has 11 variants (Extrabold, Extrabold italic, Bold, Bold italic, etc.), adding up to 2.2MB
	- Browser tries to help you fake it
- Variable Fonts
	- Instead of having a glyph for every character, we have a core set of glyphs + instructions that detail how to adjust the slant, width, etc.
	- With a variable font you pull in one file (a bit larger than a non-variable font in regular) that covers every variant
	- weight, width, optical sizing, italics, slant, etc.
	- `font-variation-settings: "wght" 850, "wdth" 100, "ital" 1;` (WHO DID THESE ABBREVIATIONS?!)
- Subsetting
	- Using a subset of glyphs from the font based on what’s actually needed
	- Most 3rd party services have subsetting baked in, e.g.Google fonts can subset via query param `&text=Hello`
- Formats
	- TTF, EOT, OTF, WOFF, WOFF2
	- TTF, EOT, OTF are pretty big, not compressed by default, need gzip
		- Support was originally pretty scattered, kind of a mess
	- WOFF, WOFF2
		- Have built in compression, especially WOFF2
		- Pretty good support
	- General recommendation: use WOFF and WOFF2, don’t worry about the rest. If it’s not supported they probably don’t need extra resources
- Don’t forget about local!
	- `local("My Font")` -> check if they have a local font on the machine called “My Font”
- `unicode-range` - Paired with subsetting, this gives a lot of control over what the browser downloads in different scenarios
	- You can do some creative stuff with this - see [Combining fonts - JakeArchibald.com](https://jakearchibald.com/2017/combining-fonts/)
```css
@font-face {
  font-family: 'Just Another Hand Fixed';
  src: local('Architects Daughter'), local('ArchitectsDaughter'),
       url('https://fonts.gstatic.com/l/…') format('woff2'),
       url('https://fonts.gstatic.com/l/…') format('woff');
  unicode-range: U+2d, U+3d;
}
```

- Loading
	- `<link rel="preload" href="lato.woff2" as="font" type="font/woff2" crossorigin>`
		- always need cross origin in place for the preload, something about anonymous
	- JS API
```javascript
var font = new FontFace("Lato", "url(/fonts/lato.woff2)", { /* ... */ });
font.load();
font.ready().then(() => {
  // apply the font once it's finished downloading
});
```
	- Turn it up to 11 [Critical Web Fonts—zachleat.com](https://www.zachleat.com/web/critical-webfonts/)
		- Critical path + preload + font loading API -> font loading on steroids
		- subset of font with data URL, inline within the HTML, only with subset of characters for the initial view
		- paired with `<link rel="preload" ...>` for full font
		- Let the font loading API kick in from there, so that when `new FontFace` happens the browser’s already on it
		- JS swaps it in and does whatever it wants style-wise
		- 2.2s for webfont to appear to 970ms

[GitHub - bramstein/fontfaceobserver: Webfont loading. Simple, small, and efficient.](https://github.com/bramstein/fontfaceobserver)

## Tooling
### [WebPagetest - Website Performance and Optimization Test](https://www.webpagetest.org/)
- Advanced Testing, Simple Testing, Visual Comparison
- Come back with basic checklists, metrics
	- Load time metric not necessarily the most important
	- First byte, 0.49s not bad
	- darker orange / lighter orange for javascript - time to first byte, then content download
	- notice ssl negotiation time before requests - could potentially do `preconnect` on these
	- nowadays the CPU is the main constraint, since we’re moving all the JS execution to the client
	- bandwidth is the opposite – we’d like to see bandwidth utilized more early on
		- you don’t want to see low, dipping down, and peaking back up
		- we prefer to use it early on and get it out of the way
	- [What Does My Site Cost?](https://whatdoesmysitecost.com/)

### Chrome Dev Tools
- Suggest using a pristine profile, so that extensions and whatnot don’t show up
- Hit shift 6 times on Settings > Experiments to show experimental experiments

core tracking
- can we batch them in some way?
- `requestIdleCallback` is perfect for analytics [window.requestIdleCallback() - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)
- User Timing (“Airbnb is doing this, which is *great*”)
	- you get to say, I know what’s important to my site, when it is truly interactive, so that’s the metric I want to track
	- custom time a process for time to interactive
	- this gives a lot more information than standard metrics
	- a lot of GPU decoding, because of the graphics
	- probably not great from a memory / battery perspective? but because it’s happening on the other thread it’s probably not a dealbreaker for this scenario
- a lot of the same info in Network panel as webpagetest
- performance profiling during user interaction
	- “actually pretty good, not bad at all”
	- a few sections with issues, see fps meter dip, red blips
	- looks fairly contained
	- “actually more boring than I thought”
	- maybe on a listing page..
	- “as you’re doing it you get distracted by the setting, like I kinda wanna go there now”
	- jumping a little bit as scrolling through listing page reviews
	- looks like raven bundle wrapping most of these functions
	-





