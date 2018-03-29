# Making Pinterest Fast - Jess, Michelle, Sarah

## First Dedicated Perf Team
- 200M global monthly active users, 1M+ requests per second
- _TIme Is Money_, Tammy Events
	- 2s expected page load time
	- 25% decrease in Google searches after 200ms slowdown
	- 28% permanent abandonment rate for slow sites
- Solid engagement lifts, but ?

> If you can’t measure something you can’t understand it…
> - H James Harrington

## Data Confidence
- **Pinner Wait Time (PWT)**
- E.g. Close-up Page - what are the important parts of the page?
	- composite metric of both image and metadata
- For home page, originally had a metric similar to “visually complete”
	- improving this metric didn’t really affect engagement
	- Time to interactive was a more accurate metric

### Fighting doubt - 4 steps to build confidence:
1. Set Baselines
	- Confidence tests
	- Graphs reflect real user experience
	- Need graphs to be absolutely correct
	- Teams understand their metrics - what we’re measuring and why we’re measuring it
2. Tie Perf Metrics to Business Goals
	- in process of running experiments to see what correlations they have
	- Need a clear link between performance and engagement
	- Helps teams use perf as a level, a feature they can ship to reach their goals in engagement for their product
	- Better trust in performance because those metrics mean something
	- teams can budget more time for perf based on impact
3. Run PR Campaign
	- be the friendly face to help spread the perf culture
	- building things we can show to more people and get them excited about performance
	- all company demos, get people excited, they can come to us for the same kinds of gains
	- other teams know about the performance team
4. Fight Regressions
	- probably the most important
	- regression protection can be more impactful than optimization
	- previously couldn’t convince teams there was a real regression, only had production
	- builds on the confidence, better trust in performance
	- regressions caught before they reach production, prevent them from creeping slowly into the product and impacting performance

## Regressions
- In-house regression testing framework, **Perf Watch**
	0. Merge code to master
	1. run perf build
	2. deploy to perf server
	3. run tests for each critical page
	4. run integration test for surface areas several hundred times
	5. calculate the 90th percentile of PWT, report to realtime dashboard

### Detecting a Regression
- E.g. exceeding a threshold by 0.5s
- Identify commit that causes the regression, create a fix
- Sometimes it’s not clear which commit caused it
- Supplementary framework **Perf Detective**
	- runs a binary search to determine the offending commit
	- i.e. like a git bisect
	- does **Perf Watch** and reports the offending commit

## Optimization Strategy
1. Analysis and brainstorming
	- waterfalls, perf data
	- met with teams to brainstorm
2. Line of Sight
	- Prioritize potential optimizations, estimate time and impact
3. Prototyping
	- Sprint, early version or hack of optimization to give us better estimates for Line of Sight document
	- Reveals certain issues with optimization that saves a lot of time down the line
4. Experimenting
	- Ran each optimization with A/B testing
	- 1% increase in a lot of the engagement metrics
	- filter data by casual (most important metric), core, dormant, marginal, new, resurrected users
	- filter by country - brazil has higher network latency, so optimizations have bigger impacts on performance and engagement
5. Rejoicing
	- reduced PWT by 20% (goal was 14%)

## Vision
- What’s the next step?
- Accomplishments need to be scalable to entire company

### Scalability
- Not just us making these optimizations, especially on surfaces we’re not experts at
### Ownership
- Encourage ownership of surfaces, can’t be the only ones jumping on regressions
### Knowledge
- No silver bullet, or general solution for every problem
- Fun to talk shop with other teams, but not something people can repeat over and over
- Need centralized documentation to know the history and learnings of perf at Pinterest
### Tools
- Built tools not necessarily developer friendly
- Make perf fun to tackle
### Strategy
- Always be on the forefront, learning, directing the strategy of perf at Pinterest

Distribute the work, **Multiply the gains.**
