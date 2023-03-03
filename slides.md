---
theme: slidev-theme-nearform
highlighter: prism
lineNumbers: false
layout: intro
---

# Otimizando a performance do Node.js
## dicas, truques e práticas recomendadas

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/rafaelgss" target="_blank" alt="GitHub"
    class="text-xl icon-btn text-white !border-none">
    <carbon-logo-github />
    @rafaelgss
  </a>
  <a href="https://twitter.com/_rafaelgss" target="_blank" alt="Twitter"
    class="text-xl icon-btn text-white !border-none">
    <carbon-logo-twitter />
    @_rafaelgss
  </a>
</div>

<!--
What can make a Node.js application slow? This talk will cover interesting stuffs
to improve your app and save your company money
-->

---
layout: two-cols
---

# Rafael Gonzaga

- Staff Engineer at **NearForm**
- Made in Brazil 🇧🇷

**Open Source**

- `@nodejs` TSC member
- `@fastify` core member
- `@clinicjs` core member

::right::

![](/images/qrcode_rafaelgss.dev.png)

<div class="abs-bl m-6 flex gap-2">
  <a href="https://github.com/rafaelgss" target="_blank" alt="GitHub"
    class="text-xl icon-btn text-white !border-none">
    <carbon-logo-github />
    @rafaelgss
  </a>
  <a href="https://twitter.com/_rafaelgss" target="_blank" alt="Twitter"
    class="text-xl icon-btn text-white !border-none">
    <carbon-logo-twitter />
    @_rafaelgss
  </a>
</div>

<style>
li {
  line-height: 1.3em !important;
}
</style>

<!--
Please, follow me on Twitter
-->

---
layout: statement
---

# Common performance issues

<!--
So, let's talk about some common issues I've seen during my journey as a consultor and as an employee solving
performance issues
-->

---
layout: two-cols
---

# Bad ISP

![](https://res.cloudinary.com/rafaelgss/image/upload/v1632097391/blog/network-performance/WhatsApp_Image_2021-09-19_at_9.09.19_PM_ynclsu.jpg)

There is nothing to do from server side in that situations

::right::

# Bad server region

![](https://res.cloudinary.com/rafaelgss/image/upload/v1632143773/blog/network-performance/Drawing-10.sketchpad_2_g67lqp.png)

It's a bad architectural decision 

<style>
img {
  border: 3px solid #000;
  width: 90%;
}
</style>

<!--
The first one is network.

Most of people tends to believe that the reason of delay is often the client bandwidth,
in fact, it helps for sure, but, it’s not a determinant factor of improvement.

Once the client reaches 5Mbps approximately, they may not see too much improvement in page load time,
on other hand, reducing latency will always improve the loading time.
This happens because the majority of latency is caused by Three-Way Handshake in HTTP/1.1.

The first image shows a ISP that does not provide the best path to reach the end server,
in our case, my personal blog. In fact, is not uncommon to see gamers angry
because of their latency in the game even with a good bandwidth contract.

The second one seems obvious, however, I’ve seen a lot of architects ignoring it.
When the server is far away from client it needs more hops to reach and consequently latency.

So, assure you to understand where most of your clients are accessing your site. In the case of an international website, it should be
configured in proxy

-->

---
class: light
---

# Overengineering

<img src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183994/its-not-fast-enough/over-1_btmnug.png">

<style>
img {
  width: 85%;
}
</style>

<!--

The next one is Overengineering, yeah, I know, everyone has worn this hat at one time.

Let's imagine the following situation:

You have two applications using the same DB, let's call it as DB1 and it's working relatively well.
1% of the users are experiencing 900ms of delay and someone joins to the project with the following concern
-->

---
class: light
---

# Overengineering

<img src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183995/its-not-fast-enough/over-2_qtuprz.png">

<style>
img {
  width: 85%;
}
</style>

---
class: light
---

# Overengineering

<img style="height: 90%" src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183995/its-not-fast-enough/over-3_e7bndh.png">

<!--
Now, you have a "needed BFF" that calls a lambda in someplace and then you have decoupled databases, so,
you need to use a message broker to guarantee the availablity and those stuffs.

In the end of the day, you are spending way more money and the P99 is worse. So, let's avoid it please.
-->

---
class: light
---

# Overengineering

<img style="height: 90%" src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183994/its-not-fast-enough/over-4_xmkfmh.png">

---

# Bad platform usage

```js
for (let i = 0; i < n; ++i) {
  // The CPU will be idle until the I/O operation is done
  // THAT'S BAD
  await longIOOperation();
}
```

| | |
|-|-|
|![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183995/its-not-fast-enough/bad-platform-usage-1_zbmir1.png)| CPU Usage with spikes, meaning a idle time
|![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183995/its-not-fast-enough/bad-platform-usage-2_a9kgti.png)| Low event loop utilization, wasting utilizable time

---
layout: section
---

# Ok, now what?

<div v-click>
  <h2>How do we start a optimization work?</h2>
</div>

---
layout: statement
---

# Establish a Baseline

- Get the current application metrics
- Perform a benchmark

<!--
Before any work, you must establish a baseline, you should get all the current metrics
and benchmarks for further comparisson
-->

---

# Isolating the bottleneck

- Q: You are a soccer coach, which player would you recruit for your team?

<div class="two-columns">
  <div style="border-right: 1px solid #000" class="col-left">
    <h2> Player A</h2>
    <img src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183993/its-not-fast-enough/isolating-op1_zq3dx9.png" />
  </div>
  <div style="margin-left: 20px;" class="col-right">
    <h2> Player B</h2>
    <img src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183993/its-not-fast-enough/isolating-op2_ywl8fy.png" />
  </div>
</div>

<div v-click>

- Answer: **A**, because you just need to teach **A** to kick to the left

> Reducing variability (isolating the problem) is essential.
</div>

<style>
img {
  width: 70%;
}
</style>

<!--
Let's talk about a very important part when you are optimizing an application, that's isolating the bottleneck.

You have the player A who kicked all the balls to the same wrong location or the player B who kicks randomly
but eventually, makes the goal.
-->

---
layout: intro
---

# Tools

---
class: light
layout: center
---

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183992/its-not-fast-enough/autocannon_vy4f8a.png)

---
class: light
---

# Clinic.js

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183992/its-not-fast-enough/clinic_aejezm.png)

<style>
img {
  width: 50%;
  margin: auto;
}
</style>

---
layout: fact
---

# Let's go to the code

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183993/its-not-fast-enough/shut-up-show-code_cg57q4.jpg)

<style>
img {
  width: 40%;
}
</style>

<!--
I said too much already
-->

---

# Simple Express application

```js {all|13-16|7-11|3-5|all}
const server = express()

const currentClients = Array(4).fill(0).map((_, idx) => {
  return { name: `Person ${idx}` }
})

async function updateAllClientsScore () {
  for (const client of currentClients) {
    await updateClientScore(client)
  }
}

server.post('/', async (req, res) => {
  await updateAllClientsScore()
  res.send({ hello: 'world' })
})
```

<div v-click>

```console
autocannon -d 20 -c 100 -m POST http://localhost:3000

> 10k requests in 20.03s, 2.47 MB read
```
</div>

---

# Clinic Doctor

```console
clinic doctor --autocannon [ -d 20 -c 100 -m POST / ] -- node cpu-idle.js
```

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183992/its-not-fast-enough/cpu-idle-doctor_i87j1e.png)

<!--
The graph generated by Clinic Doctor shows the CPU with some spikes, but, frequently idle.

When you are performing a load test using any tool (autocannon in our case), we are expecting to see the CPU
under high usage, and looking at this graph, we can see that it's not the case, it becomes idle (0%) from time to time.

This is due to bad platform usage, the code is iterating and waiting for a Promise resolution before firing the next one 

We can be smarter in this case.
-->

---

# Promise.all

```diff
async function updateAllClientsScore () {
-  for (const client of currentClients) {
-    await updateClientScore(client)
-  }
+  await Promise.all(currentClients.map(updateClientScore))
}
```

Running `autocannon` again:

```console
autocannon -d 20 -c 100 -m POST http://localhost:3000

> 30k requests in 20.05s, 7.62 MB read
```

### Handling **200%** more requests

<style>
h3 {
  margin-top: 40px;
  text-align: center;
  text-shadow: 0 0 1px #fff;
}
</style>

<!--
The `Promise.all()` is pretty simple and very known by Javascript developers

Now, the code is firing all the Promises at the same time and consequently, it will be faster.
-->

---

# Be careful with `Promise.all`

```diff
- const currentClients = Array(4).fill(0).map((_, idx) => {
+ const currentClients = Array(1000).fill(0).map((_, idx) => {
  return { name: `Person ${idx}` }
})
```

<div v-click>

```console
autocannon -d 20 -c 100 -m POST http://localhost:3000

> 4k requests in 20.04s, 876 kB read
```

<hr />

- Don't use `Promise.all` when:
  1. The promises length is unlimited
  2. The promises length is too large - the amount limit really depends on the work

</div>

<style>
hr {
  margin-top: 20px;
  margin-bottom: 20px;
}
</style>

<!--
However, be careful with `Promise.all` when you don't have a fixed limit or the limit is too high,
this is crucial for your application

And the benchmarks show that it in fact is dangerous, it reduced the performance significantly.
-->

---

# But, why?

CPU is fine

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183992/its-not-fast-enough/cpu-throttling-cpu_tpyhwu.png)

---

# Memory isn't

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183994/its-not-fast-enough/cpu-throttling-memory_ilotsl.png)

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183992/its-not-fast-enough/cpu-throttling-eld_vvhr6c.png)

- Garbadge Collection is synchronous and block the event-loop when it runs

<!--
When you run Promise.all with no boundaries,
it will allocate all the promises function in the heap at the same time, which leads to potentially
block of event-loop to run the garbadge collection
-->

---
class: light
---

# Event Loop Delay is critical

```
       ┌───────────────────────────┐
    ┌─>│           timers          │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    │  │     pending callbacks     │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    │  │       idle, prepare       │
Lag │  └─────────────┬─────────────┘      ┌───────────────┐
    │  ┌─────────────┴─────────────┐      │   incoming:   │
    │  │           poll            │<─────┤  connections, │
    │  └─────────────┬─────────────┘      │   data, etc.  │
    │  ┌─────────────┴─────────────┐      └───────────────┘
    │  │           check           │
    │  └─────────────┬─────────────┘
    │  ┌─────────────┴─────────────┐
    └──┤      close callbacks      │
       └───────────────────────────┘
```

<!--
You may have seen this diagram a lot in several talks, and in fact, it's pretty important. You must understand
how your code is executed by the platform to be able to optimize it. In our case, the GC is blocking the event loop
for some time (increasing the event-loop delay), due to allocating and freeing promises memory.
-->

---

# Instead, use `p-limit`

```js
const limit = require('p-limit')(300)

async function updateAllClientsScore () {
  await Promise.all(
    currentClients.map((client) => limit(() => updateClientScore(client)))
  )
}
```

<!--
There are several others libraries for Promise limitation, it's completely up to you to find the best fit.
-->

---

# Don't block the event-loop

( ) - No
```js
const fs = require('fs')
const server = express()

server.get('/', (req, res) => {
  const content = fs.readFileSync('largeFile', 'utf8')
  // do something with content
  res.send({ hello: 'world' })
})
```

(X) - Yes

```js
const fs = require('fs')
const server = express()

server.get('/', (req, res) => {
  fs.readFile('largeFile', 'utf8', (err, content) => {
    // do something with content
    res.send({ hello: 'world' })
  })
})
```

<!--
Ok, one more strict rule before going to fun things.

Don't block the event-loop, this sentence is widely shared but, I needed to add it in my presentation as well
just to make sure that we are aligned

When you block the event-loop all the further requests will wait until the first request is complete to start processing it
-->

---
layout: intro
---

# That's said. Let's talk about _"fun"_ things

---

# Profiling

- Imagine a large codebase

- It might have **several functions** doing CPU work

- How do we see what's the function spending more time in the CPU?

- **Profilling**🌟

<!--
Sometimes you don't know exactly which function is spending more time in the CPU, in that case,
profiling is an excelent approach
-->

---

# Clinic Flame

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183996/its-not-fast-enough/clinic-flame_y16nlm.png)

<!--
In Flamegraph the X-axis represents the combined percentage of time a function spends on the CPU

The Y (uai) axis represents the call stack

This visualization makes the evil function easier to find, this is pretty good. Let's look better at it
-->

---

# Clinic Flame

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/call-stack-flame_vq7wxv.png)

Callstack

---

# Clinic Flame

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/wider-block-flame_e49ceb.png)

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/wider-block-flame-2_gajwl9.png)

<!--
Wider blocks mean more time spent in the CPU
-->

---

# Clinic Flame

```js {all|3-6|8-14|16-20|all}
const server = express()

let obj = { a: 1 }
for (var i = 0; i < 15; i++) {
  obj = { obj1: obj, obj2: obj };
}

function sumValues () {
  let sum = 0;
  for (let i = 0; i < 9999990; i++) {
    sum += i;
  }
  return sum;
}

server.get('/', (req, res) => {
  const sum = sumValues()
  const rawData = JSON.stringify(obj)
  res.send(rawData + sum);
})
```

```console
clinic flame --autocannon [ -c 100 -d 10 / ] -- node server.js
```

<!--
So, let's look at a feasible example

The following code does two operations: sumValues and stringify a big object.

Running it using clinic flame + autocannon will produce the following flamegraph
-->

---

# Clinic Flame

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/sum-values-flame_ymoayb.png)

- What about the missing data inside (anonymous)?
  - It gets inlined

<!--
As you can see in the above image, `sumValues` is one of the functions spending more time in the CPU,
then you have the etag generation, which is a default plugin in express.

also note that anonymous (our source code) has a blank part, what is that? Let's see that in the next slides
-->

---

# Kernel Tracing

It will retrieve C++ symbols in V8/Node.js from kernel execution 

* Linux
  * Linux perf events (mostly available)

_Sorry, Windows/MacOS users :)_


```console
clinic flame --kernel-tracing -- node index.js
```

In another terminal:

```console
autocannon -c 100 http://localhost:3000
```

<!--
Clinic Flame provide a way to profile your Node.js application using kernel tracing (through linux_perf)

Basically, it gets all the source C++ symbols from V8 at kernel level

Well, running our application with `--kernel-tracing` flag and performing the benchmark we'll be able to see an quite
interesting stack trace
-->

---

# Kernel Tracing

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/sum-values-kernel_fpru9p.png)

<!--
The flamegraph generated showed the missing part, the `JSON.stringify`, which in fact,
is consuming an important piece of our CPU

Note that, I was able to get this stacktrace enabling V8 in the Clinic Flame.

A very important mention is that JSON.stringify and JSON.parse are synchronous functions, which means if you are dealing
with large objects, it will hurt your application performance
-->

---
layout: intro
---

# V8 Tricks

<!--
Let's talk about some V8 tricks
-->

---
class: 'light'
---

# Inlining

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/inlining_hfz1a7.png)

<!--
Firstly, inlining...

This is a powerful compiler approach that affects directly the profling analysis.

When a code/function is inlined, it's compiled into another function, normally, its caller.

However, since you don't have the function anymore, you may lose some stacktrace in the profiler as you have seen in
JSON.stringify example.

Fortunately, Clinic Flame does some tricks to track it down, though it can fail at times.
-->
---

# What to expect from inlining?

- Making a single function inline in a hot path yields a 5-10% improvement in throughput

- V8 has a limit to how many functions it can inline, because the risks of deopts are too high

---
layout: two-cols
class: two-cols-bordered
---

# -Inlining

Try to run the application code without inlined functions

```console
node --no-turbo-inlining index.js
```

```console
Running 20s test @ http://localhost:3000/a
100 connections

Req/Bytes counts sampled once per second.

343k requests in 20.05s, 75.9 MB read
```

> You will notice that it decreases the performance significantly.

::right::

# +Inlining

The original version `node index.js` shows the comparisson

```console
Running 20s test @ http://localhost:3000/a
100 connections

430k requests in 20.05s, 95.2 MB read
```

<style>
blockquote {
  background: #1b4592 !important;
  border-left: 10px solid #5c8de7 !important;
  padding: 0.5em 10px !important;
}
</style>

<!--
If you want compare your code with no inlined function, you can pass the flag `--no-turbo-inlining`
-->

---

# Hidden Classes

- Accessing an object's property is the most used operation in any Javascript code
  ```js
  myObj.key1 // 'test'
  ```
- Making it very fast is critical for JS engines

> In order to efficiently store and access object's properties, all Javascript engines introduced the concept of _shapes_ (also referred as **"Hidden Classes"**).

<!--
Hidden Classes is a very important part of any Javascript engine, in our case, V8.
-->

---

# Hidden Classes

- A _shape_ (hidden class) is a dictionary of an object's properties, with the exception of the value, which is stored separately
  - Instead of values, the shape contains the **memory offset** at which the value is found

- When two objects are created with the same properties, the engine assigns to them the same shape.
  - It **improves the memory management** significantly

<!--
- Values can then be stored and accessed sequentially in memory, which results in better performance.
-->

---
class: light
layout: two-cols
---

# Hidden Classes

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183990/its-not-fast-enough/shape-obj-1_lmlmpm.png)

::right::

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183990/its-not-fast-enough/shape-1_vrk7vt.png)

<!--
In this example, Object A and Object B use the same shape, with different memory offets, but continuous in the memory, so it's fast!
-->

---

# Property order is relevant

- Being based on memory offsets, property order is relevant.

  ```js
  const a = {x: 1, y: 2}
  const b = {y: 2, x: 1}
  ```

  The two objects above **will not** have the same shape.

- Adding properties to an object changes its shape.

<!--
- If the object is modified too often, a transition chain visit can be very expensive.

- Adding properties to an object, add a transition chain to the object. Look at the next diagram
-->

---
layout: two-cols
class: light
---

# Transitions

<img src="https://res.cloudinary.com/rafaelgss/image/upload/v1651183990/its-not-fast-enough/shape-obj-2_gipd4n.png" style="width: 90%;" />

::right::

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/shape-2_upv5ia.png)

<!--
The `z` property inside `b` object is added after the object creation, meaning a new shape chain.

It means, that object B contains two shapes and the source code is allocating memory for 3 shapes
-->

---

# Don't use `delete`

- When a property is _deleted_ from an object, the engine does not use shapes anymore.
  - Property access becomes much slower.

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/delete-property-bench_ortxgo.png)

<!--
The benchmark was retrieved from a repository I maintain to benchmark some tricks in Node.js, check it out in my Github profile.

If you are using delete property for some reason, I strongly recommend you to just assign undefined instead. It can save some headaches
-->

---

# `express` is not a good choice nowadays

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/express-benchmark_ocoy9j.png)

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183990/its-not-fast-enough/koa-benchmark_nvyrl6.png)

![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183990/its-not-fast-enough/fastify-benchmark_oohl1z.png)

<!--
- **Express** (⭐56k):
  - v5.0.0 (beta) -> just dependencies update
  - No features since 2019
  - Stagnant
- **Fastify** (⭐22.5k):
  - v4.0.0 (alpha)
    - **Better** Typescript support
    - DX Updates
    - New error handling composition
  - Still the fastest web framework in the Node.js globe
-->

<style>
img {
  width: 70%;
}
</style>

<!--
Express is not directly maintained since 2020, there are no new features or new improvements coming and you must not follow the herd behavior.

Fastify, instead is getting improvements every day reaching the v4 pretty soon, check it out and I swear you won't regret it.
-->

---

# Don't use private properties on <= v16

```console
Manipulating private properties using #             x   2,121,767 ops/sec ±0.82% (90 runs sampled)
Manipulating private properties using underscore(_) x 567,016,249 ops/sec ±0.39% (95 runs sampled)
Manipulating private properties using Symbol        x 557,787,596 ops/sec ±1.00% (91 runs sampled)
```

Rather, use:

- `_underscore` convention
  ```js
  class A {
    constructor() { this._privateProp = 0 }
  }
  ```
- `Symbol`
  ```js
  const kPrivateProp = Symbol('private-prop')
  class A {
    constructor() { this[kPrivateProp] = 0 }
  }
  ```

<!--
The last tip is don't use private properties in your application, it's very slow. Instead, use Symbol for libraries or just the underscore convention.

Important to mention that V8 solved this performance issue, but it's not available in Node.js yet, so, for those who are using Node v17 and below, don't use that, trust me.
-->

---

# Is X faster than Y on Node.js vX.Y.Z?

![](/images/qrcode_github.com.png)

<style>
img {
  width: 40%;
  margin: 0 auto;
}
</style>

---

```js
const number = parseInt('10', 10)
```

```js
const number = +'10'
```

![](/images/parsing-integer.png)

<style>
table, th, td {
  border: 2px #000 solid;
}

th {
  font-weight: bold;
}
</style>

---

# Date.toISOString regression

* Node.js v18

![](/images/date-to-iso-string-v18.png)

* Node.js v19

![](/images/date-to-iso-string-v19.png)

<style>
img { width: 70% }
</style>

---

# How do we avoid regressions in a codebase?

- [<carbon-logo-github />RafaelGSS/autobench](https://github.com/RafaelGSS/autobench)

autobench.yml
```yml
name: 'Autobench Example'
benchFolder: 'bench'
url: 'http://localhost:3000'
benchmarks:
  - name: 'request slow promise'
    path: '/slow'
```

- `autobench compare`

<!--
I wrote a CLI that reads autocannon output and compare the throughput in PR's. It's very useful to avoid performance regression in your repository. Check it out, it's completely Open Source.

Basically, you just need an autobench.yml file that exports all the routes available for benchmark and a Github Action for that.

See the next example
-->

---
class: light
---

# Autobench

**Regression:**
![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/autobench-regression_c5s9xu.png)

<hr>

**Success:**
![](https://res.cloudinary.com/rafaelgss/image/upload/v1651183991/its-not-fast-enough/autobench-success_hatofj.png)

<style>
img { width: 60% }
</style>

<!--
Those messages are completely adjusted in the autobench file
-->

---
layout: statement
---

# Thanks <3
