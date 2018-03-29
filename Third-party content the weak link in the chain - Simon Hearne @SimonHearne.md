# Third-party content: the weak link in the chain - Simon Hearne @SimonHearne
#PerfMatters

simonhearne.github.io/weak-links

AKA Bad people doing horrible things to good sites

## Modern web workflow 101
1. make something awesome
2. test it
3. ship it
4. …
5. put tags on it

- we seem to have less control than ever over 3rd party content
- biggest issue is “the business”
- tags serve business goals
	- measurement & analytics
	- ads & retargeting
	- “optimization” & testing
		- optimizely, etc
	- comments & live chat
	- tag management
- the _money_
	- we know that optimizely slows down the site, but it will get us $750k increased revenue this year
- the _its not my job_
	- we suspect it slows the site down, we haven’t tested it. marketing says its critical to their latest tv campaign so theres no font arguing
- the _tag manager_
	- all the tags go through the tag manager, so they should be fine

- but what about the _risk_?
	- ps: availability heuristic 101
	- remember when facebook went down?
	- remember when disqus went down?
	- remember when maximizer went down?
	- remember when Dyn went down
	- left-pad :(
- risk 1: malicious code injection
	- Snyk: how much of your code has vulnerabilities?
	- cryptojacking
	- it happens to the biggest players
	- internet “service” providers
	- content delivery networks
	- unintentional data collection
		- website tracking is a “security disaster waiting to happen”
- risk 2: single point of failure
	- do they fail gracefully
	- are they using a can? (& is it as good as yours?)
	- what is their SLA for availability? (& is it as good as yours?)
	- availability to the user
		- government / geo blocking
		- ad blocking
		- tracker blocking
		- random crap
- risk 3: code quality
	- XSS vulnerabilities
	- different release schedules
		- how do you know when it changes?
	- just plain thoughtless
- risk 4: performance
	- self-policing isn’t good enough
		- …the X Web Reference snippet was available… and the download time over https did not exceed 500ms.  Snippet is sampled every minute from a variety of U.S. locations.
	- tools aren’t equal
	- the web is variable
	- resource timing is the hero we need
		- not without Timing-Allow-Origin (smh)
	- resource timing won’t save us
		- no redirect information
		- limited data on 72% of third-party content
		- only the first 150 requests
		- no data on old iDevices
		- no data for cross-origin iframes
- risk 4: performance (for real this time)
	- cpu is our biggest bottleneck
	- who’s policing the third-parties
	- variable CPU time (on synthetic agents!)
	- the most frustrating perf bug, ever
		- 1600ms between click and navigation
	- who watches the watchmen?
- risk 4: performance (part iii)
	- delaying onload
	- keeping the radio awake
		- using a website and your phone gets hot…

## we have little control over which are used
but there are things we can do…

### stage 1: find out what’s there
- synthetic testing
	- webpagetest.com
	- https://requestmap.this/
	- bonus: third-party categorization github.com/simonhearne/thirdpartydb
- RUM
	- Akamai mPulse

### stage 2: determined the impact
- synthetic testing
	- webpagetest with “block” tab
	- github.com/simonhearne/third-party-impact
- resource impact from synthetics
	- blog.catchpoint.com/2018/01/10/using-catchpoint-to-analyze-third-party-impact/
	- speedcurve.com/blog/ux-focus-for-waterfalls-and-third-parties/
- resource impact from RUM
	- tool demo: page group conversion impact & analysis ::(what is this?)::
- advertising partners
	- partner 1 = ~400ms slower than partner 2
	- migrating all ads = 220ms faster page load
	- additional revenue ~= $12,000 per month
- longtasks API
- bonus: determine the value!
	- blog.sumall.com/journal/optimizely-got-me-fired.html

> Everything should have a value, because everything has a cost.  
> – Tim Kadlec  

### stage 3: measure them and report on them
- content security policy (report-only)
	- header you can set that says “this page, you can only execute scripts, download CSS from this domain”
	- report-uri.com
- synthetic testing
- RUM
	- the best way to monitor resources, gives you the breadth of data
	- waterfalls, Akamai Impulse

### stage 4: defending ourselves
- Content Security Policy
	- CSP Strict-Dynamic
- Sub-resource integrity
	- github.com/mikewest/signature-based-sri
- Service worker
- self-hosting / proxying
	- google-webfonts-helper.herokuapp.com/fonts - self-hosting google fonts
	- wwo.com/knowledge/host-wwo-javascript-files-on-your-server

### stage 5: have a third-party policy
- what does it do?
- who uses it?
- whats the risk to the site?
- how do you remove it?

### share with other teams!
- open source dashboard tools
