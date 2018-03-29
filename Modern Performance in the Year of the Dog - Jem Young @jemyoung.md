# Modern Performance in the Year of the Dog - Jem Young @jemyoung
#PerfMatters

- How to balance perfect engineering with perfect design?
- Balance User Experience with Developer Experience
- Internet 3x faster than it was a few years ago, are websites 300% faster?

> Engineering is about trade-offs. Good engineering is about making good trade-offs.  
> – Jacques Favreau  

## What makes up performance?
- most people would say measurement. TTR 3s we good, etc.
- performance tips and their trade-offs

thing // trade-off

### What is the best site on the internet?
- Craigslist is great
	- Fast, everything is on front page
	- HTML, CSS, minimal JS

use only html & css // limits features

## Developer Performance
- One of the most neglected topics when we talk about performance
- When’s the last time you wrote a web server in C?

### What is developer performance?
- `performance === productive`
- Can I walk into a codebase and solve a problem quickly?

#### Architecture
- `team_snake_case` vs `teamCamelCase`
- be consistent // developer freedom

#### Let’s build an app
- OMG, dogs! v1.0
- css/ js/ html/

#####  libraries
- scripting time is the slowest part
- Does it make you more productive?
- developer productivity // page weight
- choose what makes **your team** productive

### Application Performance
- OMG, dogs! v1.1
- click a button -> get a dog
- Time it, takes over 1000ms from button click to loading the next dog image

#### Network Performance
- fetch lol
- `Promise.all`, wait for all the images to load
- `Promise.race`, first one that comes back gets returned
	- it throws away the other requests
	- but the idea still holds
	- What sort of data structure should we use?
	- **Buffer**
	- Jem you work at Netflix? RXJS? 
		- no
	- Bluebird?
		- Pugsly is concerned about our use of libraries
		- “Infinite collection” -> generator
			- `while(true) {` ._.

- OMG dogs! v2.0
	- pretty fast
	- we just thought about the problem differently

maximize all connections // resource consumption

- We don’t want to forget about mobile users, they don’t have unlimited data plans
	- we can’t just blindly apply performance tips, we should think very carefully about what we’re doing

- how do we know?
	- measuring performance
	- OMG, dogs! v2.2
		- LOOKS PRETTY GOOD
	- measure FPS, TTR, PWT, … // uncertainty
		- let’s say it’s causing frames to drop, how do we know?

### Server Performance
- transfer protocols
	- http/1.1 - 2.1s
	- http/2 - 901ms
	- using http/2 can increase page speed // need to use SSL
		- what’s the problem with using SSL?
		- computing cert can be expensive
		- is it worth the server cost? yes, internet privacy is important, but it’s still important to weigh the tradeoffs
- compression
	- instead of sending every bit, we send in weird language that explains what that file is
	- gzip vs brotli
		- brotli is 3x slower on the server
	- static compression?
	- use brotli to compress static files // server overhead

## How to balance perfect engineering with perfect design?
design // engineering
you can’t - you do the best you can

https://dog.ceo
