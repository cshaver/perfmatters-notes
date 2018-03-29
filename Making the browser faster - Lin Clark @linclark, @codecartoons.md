# Making the browser faster - Lin Clark @linclark, @codecartoons
#PerfMatters

- Dropdown menu craziness -> lots of repaints became important, ideally 60fps
- Content authors pushing boundaries further
	- not just making pages more interactive, but new classes of apps to the web (PC games, photoshop)
	- Macbook: 16ms 4.5 million pixels
	- new iPad: 8ms 3.5 million pixels
	- VR: 11ms 16.5 million pixels
- Browsers preparing to make these leaps to keep up with these changes
	- parallelism

## What does the browser do?
- Parse
	- create DOM tree: data structure for elements on the page and how they relate to each other
	- doesn’t tell us what it should look like
- Style
- Layout
	- dimensions of browser window -> relative values (e.g. 50% width)
	- breaks up bits of say paragraphs, how many lines high they’ll be
	- this is what tells us exactly what the page should look like
- Painting
	- splitting up what needs to be displayed into layers
	- like old cartoons
- Composite & Render
	- put the layers together and put that on the screen

Browser chrome, can also use this rendering engine
Tabs are where you start to see this parallelism
Chrome did it first
multi-process architecture

## Computer Architecture crash course
- brain parts:
	- thinking - arithmetic and logic unit (ALU)
	- short-term memory - registers
	- long-term memory - RAM
	- ALU + registers = CPU
- need instruction
- each box of short-term memory has a label
- instructions, e.g. `ADD R4 1`
- one thing at a time, think one thought at a time
- true for computers until mid-2000s, but they were still getting faster (Moore’s law)
- limitations becoming apparent
- doot doot cores
- data race - parallel code with same memory
- one way to get around is to choose tasks independent of each other so they don’t have to share memory
- course-grained parallelism

## firefox
> You must be this tall to write multi-threaded code.  

we can’t all be wizards
data races & memory bugs cause some of the worst security bugs
how to make it safe?
sponsoring work on rust programming language

Parse -> JS engine
- a lot of calculations to figure out what changes needed into DOM tree
- only so much the JS engine can do to speed up this application code
- what if app code could run in parallel?
	- take advantage of multiple cores the same way the browser code can?
	- web workers
		- js code which runs of different cores
	- shared array buffers


- course-grained parallelism - implemented in all browsers
- app code parallelism - standards being adapted by all browsers
- fine-grained parallelism - ?
