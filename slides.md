---
theme: default
title: OpenTelemetry Logs Driving a Major Shift
favicon: ./favicon-32x32.png
class: text-center
transition: slide-left
mdc: true
---

<style>
/* Ensure presenter notes are visible. */
.dark .prose {
  color: white;
}
.light .prose {
  color: black;
}
</style>

<h1>
<strong>
<span class="text-orange">Open</span>
<span class="text-blue">Telemetry</span>
Logs
</strong><br>
<u>Driving a Major Shift</u>
</h1>

## Events, Richer Data, and Smarter Semantics

<div class="pt-12">
  by Robert Pająk
</div>

<div>
  pellared @
  <a href="https://github.com/pellared" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<https://otel-logs-talk.netlify.app>

<!--
Welcome to my talk about how OpenTelemetry Logs evolved from the least mature signal to potentially the foundation of new major enhancements.
-->

---
layout: center
---

# `whoami`

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">
<v-clicks>

🏠 [Kraków, Poland](https://krakow.travel)

‍💻 [@pellared](https://github.com/pellared) on GitHub

💼 Software Engineer at [Splunk a Cisco company](https://www.splunk.com)

🔧 [OpenTelemetry Go](https://github.com/open-telemetry/opentelemetry-go) maintainer and [Specification](https://github.com/open-telemetry/opentelemetry-specification) sponsor

🌴 Contributing to OTel Logs since Nov 2023

<div class="text-lg mt-8 opacity-80">
  <strong>Disclaimer:</strong> Non-native English speaker.<br/>
</div>
</v-clicks>
</div>

<!--

[click] I've traveled from the beautiful city of Kraków (Cracow).

[click] You can find me lurking around GitHub as pellared, probably creating some issue or opening yet another pull request.

[click] I'm a Software Engineer at Splunk (a company aquired by Cisco), which means I get paid for contributing to open source.

[click] I am an OpenTelemetry Go maintainer.
I'm also an OpenTelemetry Specification sponsor.
Fun fact: I'm apparently the 3rd top OpenTelemetry Contributor according to Linux Foundation Insights.
I'm not sure if that's impressive or just means I need better hobbies.

[click] I volunteered to design OpenTelemetry Go Logs and improve the OpenTelemetry Logs Specification in November 2023. 
Apparently, someone thought I knew what I was doing.

[click] I am not an English native speaker.
But if something is not clear, don't blame my English.
It simply means I have no idea whay I am talking about.

-->

---
layout: center
---

<h1 class="flex gap-2">
<code>uname</code>
<span class="flex gap-2">
  <img src="./opentelemetry-logo-nav.png" alt="OpenTelemetry" class="h-10" />
  <strong>
    <span class="text-orange">Open</span>
    <span class="text-blue">Telemetry</span>
  </strong>
</span>
</h1>

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">

<v-clicks>

A **framework** for generating, processing, and exporting telemetry data:

📊 **traces** - distributed request flows

📈 **metrics** - numerical measurements over time

📝 **logs** - timestamped records

🔍 **profiles** - performance profiling data

<div class="text-xl mt-8 text-orange">
  One standard to observe them all 💍
</div>

</v-clicks>

</div>

<!--

For those who might be new to OpenTelemetry, let me give you a quick overview.

[click] OpenTelemetry is a framework for generating, processing, and exporting telemetry data.
It provides a unified approach to observability across different signal such as

[click] traces

[click] metrics

[click] logs

[click] profiles

[click] The beauty of OpenTelemetry is that it gives you one standard way to instrument your applications, regardless of the language or backend you're using.
Whether you're sending data to Jaeger, Prometheus, Splunk, or any other observability platform, OpenTelemetry provides the common foundation.

-->

---
layout: center
class: text-center
---

<div class="text-5xl text-left mx-auto space-y-18">

<span v-click.hide>🕰️ Past (Deprecated)</span>

📍 Present (Stable)

🚀 Future (Development)

</div>

<!--

People usually talk about the past, present, and future.

[click] But honestly, who cares about the past?
We're here to talk about what's happening NOW and what's coming NEXT.
We're not just talking about logs in isolation.
We're talking about how logs and how they're driving major improvements across the entire OpenTelemetry ecosystem.

-->

---
layout: center
---

# `init()`

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">

<v-clicks>

🔧 syntax

📚 semantics

⚡  api

</v-clicks>

</div>

<!--
During the talk are are going to explore the following areas in the context of OpenTelemetry:

[click] the syntax (or data model) of logs,

[click] the semantics of logs and events,

[click] the logging APIs.

We'll dive into the design decisions, and emerging patterns that are turning logs from "legacy baggage" into the foundation for the next generation of OpenTelemetry-powered insights.

By the end of this talk, you'll understand why logs are no longer an afterthought, but a first-class citizen among other OpenTelemetry signals.

-->

---
layout: center
class: text-center
---

# trash logs

<div class="text-left">

```log
[2025-09-10 14:22:01] INFO: It works
[2025-09-10 14:22:02] INFO: Still works
[2025-09-10 14:22:03] INFO: Yep, still working
[2025-09-10 14:23:44] ERROR: Failed
[2025-09-10 14:25:10] DEBUG: User payload: {"user":"alice","password":"hunter2"}
[2025-09-10 14:26:30] WARN: Something went wrong!!!
StackTrace: java.lang.Exception: oh no
at com.company.module.Class.method(Class.java:42)
at com.company.module.Other.method(Other.java:99)
at com.company...
(more 800 lines)
LOGGING HERE ----------------------------------
value=1
LOGGING HERE ----------------------------------
value=2
LOGGING HERE ----------------------------------
value=3
```

</div>

<div class="text-2xl">
Who has seen logs like this? 🙋‍♂️
</div>

<!--

Show hands: Who has seen logs like this?
Keep your hands up!
Don't worry, I won't judge you.

*Say what part of the audience have their hands up.*

I've written logs like this too.
We've ALL been there.
"LOGGING HERE" with dashes?
Classic debugging technique.
Passwords in debug logs?
Academic excellence right there.
-->

---
layout: statement
---

# THIS is why<br>we need better logging standards

<!--
But seriously, this is what we're working with in production systems everywhere.

Note that wasted logs cost money.

There's nothing like a free meal, and every useless "It works" message is burning through your observability budget.

Plus, it slows down your application.

Moreover, it adds noise to the captured telemetry.

THIS is why we need better logging standards.
-->

---
layout: section
---

# syntax

<!--
What does a proper log record look like in OpenTelemetry? Let's talk about the Logs Data Model.
-->

---
layout: center
class: text-center
---

<h1>
<strong>
<span class="text-orange">Open</span>
<span class="text-blue">Telemetry</span>
Logs
</strong>
<br>
Data Model
</h1>

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">
<v-clicks>

⏰ Timestamp & Observed Timestamp

🔗 Trace Context

📊 Severity

📝 Body

🏷️ Attributes

🎯 Event Name

📍 Resource & Instrumentation Scope

<div class="text-xl mb-6 text-orange">
  Not just strings with timestamps.
</div>
<div class="mt-8 text-xl font-bold text-blue">
  Logs that carry context. Logs that make sense! ✨
</div>

</v-clicks>
</div>

<!--
In OpenTelemetry, a log record isn't just a string with a timestamp. It's a structured piece of telemetry with:

[click] the timestamps when it happened and when it was captured

[click] the trace and span IDs for correlation magic

[click] not just "INFO/WARN/ERROR" but proper severity levels

[click] the actual log message or payload that can be complex

[click] the key-value pairs for structured metadata

[click] the event name identifies the type of event being recorded - we are going to discuss it later

[click] the information what application and which library or component generated this log

[click]
This isn't just regular logging made more complicated.

[click]
It is structured logging that actually integrate with your traces.

-->

---
layout: center
---

# Log Record

```json {1-2|3-5|6-7|8|9-14|15-19|20-22}
"Timestamp": "2023-11-15T09:15:20.250Z",
"ObservedTimestamp": "2023-11-15T09:15:20.251Z", 
"TraceId": "b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3",
"SpanId": "c1d2e3f4a5b6c7d8",
"TraceFlags": 1,
"SeverityText": "ERROR",
"SeverityNumber": 17,
"Body": "Exception occurred while fetching user data",
"Attributes": {
  "exception.type": "java.sql.SQLTransientConnectionException",
  "exception.message": "Connection timeout after 3000ms",
  "user.id": "alice",
  "db.operation": "select"
},
"Resource": {
  "service.name": "user-service",
  "service.version": "1.5.2", 
  "deployment.environment": "staging"
},
"InstrumentationScope": {
  "Name": "org.example.user.controller.UserController"
}
```

<!--

Here you can see the timestamp when the something happened and when it was recorded in OpenTelemetry.

[click] As you can see the log record contains the trace context.

[click] The severity is a level which is a number, but it is also a text so that the names from logging libraries can be preserved.

[click] Body is typically the log message.

[click] Attributes are any additional metadata or log fields.

[click] Resource contains the data boostrapped by OpenTelemetry SDK that allows identifying the software that emited the log record.

[click] Instrumentation scope contains the information about the source code that emited the log record.

-->

---
layout: center
---

# complex data

<div class="mb-8">
  <div class="text-2xl text-blue mb-4">
    <code class="text-orange">Attribute</code> and <code class="text-orange">Body</code> values can be complex
    (<code class="text-green">AnyValue</code>).
  </div>
  
  <div class="text-2xl">
    <code class="text-green">AnyValue</code> can be one of:
  </div>
</div>

<div class="grid grid-cols-1 gap-4 max-w-4xl mx-auto text-xl">
<v-clicks>

<div class="flex items-center gap-4 p-4 bg-gray-800 rounded-lg border-l-4 border-blue">
  🔤 <span class="font-mono text-blue">primitive value</span> <span class="text-gray">(<code>string</code>, <code>int</code>, <code>bool</code>, <code>double</code>)</span>
</div>

<div class="flex items-center gap-4 p-4 bg-gray-800 rounded-lg border-l-4 border-green">
  📋 <span class="font-mono text-green">array of AnyValue</span>
</div>

<div class="flex items-center gap-4 p-4 bg-gray-800 rounded-lg border-l-4 border-purple">
  🗂️ <span class="font-mono text-purple">map of string to AnyValue</span>
</div>

<div class="flex items-center gap-4 p-4 bg-gray-800 rounded-lg border-l-4 border-yellow">
  💾 <span class="font-mono text-yellow">byte array (BLOB)</span>
</div>

<div class="flex items-center gap-4 p-4 bg-gray-800 rounded-lg border-l-4 border-gray">
  ⚪ <span class="font-mono text-gray">empty value</span>
</div>

</v-clicks>
</div>

<!--
The Body and Attributes fields in OpenTelemetry logs can contain complex data structures.
Not just simple strings and numbers that make you cry at 3 AM.
The value can be:

[click] primitive value like text, number or boolean value,

[click] an array of any value, meaning that the first item in an array can be a string and the second item can be an integer

[click] a map of string to any value, meaning you can compose any nested structure like a Matryoshka doll,

[click] or even a binary object,

[click] at last in can be nothing.

You can approxiamte it to something like BSON format.

-->

---
layout: center
---

# Complex values

```json
"gen_ai.request.model": "gpt-4-turbo",
"gen_ai.usage.input_tokens": 110,
"gen_ai.usage.output_tokens": 45,
"gen_ai.prompt": [
  {
    "role": "system",
    "content": "You are a helpful and knowledgeable AI assistant from Splunk."
  },
  {
    "role": "user",
    "content": "What is OpenTelemetry?"
  }
],
"gen_ai.completion": [
  {
    "role": "assistant",
    "content": "OpenTelemetry is an open-source observability framework for collecting telemetry data."
  }
]
```

<!--
Here are some examples of complex attributes.
This means you can log entire payloads, complex error objects, or structured events without flattening them into strings.
No more trying to reconstruct object hierarchies from flat attributes like some kind of archaeological dig through your own code.
-->

<div class="mt-6 text-lg">
  <span class="text-green">✨</span> Rich, structured, meaningful data!
</div>

---
layout: statement
---

# Complex values<br>are coming<br>to ALL signals!

OTel is dropping the attributes values restriction

<!--
But wait, there's more! These complex values are coming to ALL OpenTelemetry signals: traces, metrics, and profiles.

This won't be just a logs feature.

It's like getting a free upgrade to business class for your entire observability stack.

Imagine spans with complex attributes that can hold entire request/response objects. We're talking about a unified approach to complex data across all telemetry signals.

Getting agreement that this is even acceptable took about a year of proposals, bikeshedding, and "is this not breaking?" discussions.

It's like finally getting your whole family to agree on pizza toppings: miraculous and life-changing.
-->

---
layout: section
---

# semantics

<!--
We've got structure and complex data.
But how do we make sure everybody calls things the same way so we can actually search, correlate, and NOT cry when we merge data from five services written by five teams?
-->

---
layout: statement
---

<h1>
<span class="text-orange">Open</span>
<span class="text-blue">Telemetry</span>
<br>
Semantic Conventions
</h1>

Making logs searchable & comparable

<div class="text-2xl font-bold text-green mb-6">
  Same semantics = Better queries and dashbaords
</div>

<!--
That's where semantic conventions come in.

Think of them as telemetry guidelines with a giant shared dictionary of attribute names that all SDKs, libraries, instrumentation, and software agree to use.

The semantic conventions registry is organized into groups.
-->

---
layout: center
---

# Cross-cutting attributes

- `code.line_number`
- `exception.message`
- `service.name`, `deployment.environment`

<!--
We have cross-cutting attribute groups like code or exception groups.
They apply across many signals and use cases. For example we have:

code.line_number – because knowing where it broke is half the battle

exception.message – the actual error

service.name, deployment.environment – so you know what application broke and if it is on production or just staging
-->

---
layout: center
---

# Signal-specific attributes

- `log.record.uid`
- `log.iostream` 

<!--
There are also attributes that make sense only for log records.

log.record.uid – stable unique ID for the log record

log.iostream – stdout, stderr, or that custom log stream you regret creating
-->

---
layout: center
---

# Domain-specific attributes

- `http.method`, `http.request.body.size`
- `db.system`, `db.statement`
- `gen_ai.provider.name`, `gen_ai.response.id`

<!--
At last, there are attributes ocused on protocols, systems, and shiny new tech.

There are attributes for the web stuff, database things, and even AI.
-->

---
layout: section
---

# event vs log

<!--
OpenTelemetry distinguishes between Log Records and Event Records.

Log Record is for general-purpose logging, human-readable, "something happened".
Event Record is a well-defined occurrence with stronger semantics, "this specific thing happened".
-->

---
layout: statement
---

<h1>
An Event Record<br>
is a Log Record<br>
with an <span class="text-orange">event name</span><br>
and a <span class="text-blue">well-known structure</span>.
</h1>

<!--
An Event Record is a Log Record with an event name and a well-known structure.
-->

---
layout: center
---

<div class="grid grid-cols-2 gap-8 text-lg">

<div class="text-left">

### Log

```json
{
  "Body": "User logged in successfully",
  "SeverityNumber": 9,
  "SeverityText": "INFO", 
  "Attributes": {
    "user.id": "octobot"
  }
}
```

</div>

<div class="text-left">

### Event

```json
{
  "EventName": "user.login",
  "SeverityNumber": 9,
  "Attributes": {
    "user.id": "octobot"
  }
}
```

</div>

</div>

<div class="text-center mt-8 text-xl text-green">
  Same data model, different semantics.
</div>

<!--
Why the distinction?

Because not every line your app spits out deserves to be treated like a meaningful business event.

Events are intentional, instrumented with purpose, not sprinkled around like debugging salt.

This matters because platforms can now treat Events as first-class citizens.

You can index them differently, correlate them intelligently, and maybe even generate useful alerts instead of just noise.
-->

---
layout: statement
---

# spans & metrics from events

<v-clicks>

<div class="text-left mb-8">

```sh
time="23:00:00.000Z" eventName="span.start" traceID=T01 spanID=S01 parentID=    spanName=requestHandler
time="23:00:00.010Z" eventName="span.start" traceID=T01 spanID=S02 parentID=S01 spanName=storeJob
time="23:00:00.150Z" eventName="span.start" traceID=T02 spanID=S03 parentID=    spanName=requestHandler
time="23:00:00.160Z" eventName="span.start" traceID=T02 spanID=S04 parentID=S03 spanName=storeJob
time="23:00:00.250Z" eventName="job.stored" traceID=T02 spanID=S04 parentID=S03
time="23:00:00.486Z" eventName="span.end"   traceID=T02 spanID=S04 parentID=S03 spanName=storeJob       duration=326ms
time="23:00:00.490Z" eventName="job.done"   traceID=T02 spanID=S04 parentID=S03
time="23:00:00.500Z" eventName="span.end"   traceID=T02 spanID=S03 parentID=    spanName=requestHandler duration=230ms
time="23:00:00.900Z" eventName="job.stored" traceID=T01 spanID=S02 parentID=S01
time="23:00:01.010Z" eventName="span.end"   traceID=T01 spanID=S02 parentID=S01 spanName=storeJob       duration=1s
time="23:00:01.020Z" eventName="job.done"   traceID=T01 spanID=S02 parentID=S01
time="23:00:01.123Z" eventName="span.end"   traceID=T01 spanID=S01 parentID=    spanName=requestHandler duration=1123ms
```  

</div>

<div v-click class="text-2xl mb-8 text-red">
⚠️ There are trade-offs ⚠️
</div>

<div>
Read more about: <code><a href="https://brandur.org/canonical-log-lines">Canonical Log Lines</a></code> and <code><a href="https://jeremymorrell.dev/blog/a-practitioners-guide-to-wide-events">Wide Events</a></code>.
</div>

</v-clicks>

<!--

Jeremy Morrell wrote a nice blog post called "A Practitioner's Guide to Wide Events" that basically says: "What if we just logged everything we need in really rich, wide events?"

The idea is simple but powerful: instead of having traces, metrics, and logs as separate things, you emit super-detailed log events that contain ALL the context you need.
Then you can derive spans and metrics from these logs with proper semantics.

Think about it:
- Want a span?
  Extract the start/end timestamps and duration from your events.
  Follow the correlation IDs and build the trace graph.
- Need metrics?
  Aggregate the numerical values in your event attributes.

[click]
Now, I have to admit something: I was actually doing this back in 2019.
Not because I was some visionary genius, but because I only had access to an efficient logs backend.
When you're stuck with one tool, you get creative.
Turns out, constraints sometimes lead to good architectural decisions. Who knew?

[click]
Of course, there are huge trade-offs.
Storage costs and processing overhead on the backend, bigger network traffic, the existential crisis of "am I doing observability wrong?".
But when it works, it's like having X-ray vision into your systems.
Everything is connected, everything has context, and you can slice and dice your data however you want without wondering if you forgot to add that one crucial attribute to your metric.
And always remember, there is no silver bulet.

[click]
You can read about it more online.
-->

---
layout: section
---

# api

<!--
Until recently, OpenTelemetry logs were mostly about getting logs from file streams or collecting (bridging) logs from existing logging frameworks (like log4j, Serilog, or Python's logging module) and adding correlation context.

But what if you want to log directly using OpenTelemetry APIs?

What if you want to emit those beautiful structured events we've been talking about?
-->

---
layout: center
---

# User-facing logging API

```go
// Emit an event record.
logger.WarnEvent(ctx, "rate_limit.approached",
    semconv.ClientIP("192.168.1.100"),            // Reuse attribute from semantic conventions.
    attribute.Int("rate_limit.max_per_sec", 100), // Custom application-specific attribute.
) 
```

<div class="text-xl text-left mx-auto space-y-4 max-w-4xl mb-8">
<v-clicks>

🚀 No format conversion

✨ Full feature access

🎯 Consistency

</v-clicks>
</div>

<div class="grid grid-cols-2 gap-8 text-xl max-w-4xl mx-auto">
  <div v-click class="text-center p-6 bg-blue-900 rounded-xl border-2 border-blue-400">
    <div class="text-4xl mb-4">🔧</div>
    <div class="text-blue-400 font-bold mb-2">Instrumentation Authors</div>
    <div class="text-sm text-gray-300">Library & framework developers</div>
  </div>
  
  <div v-click class="text-center p-6 bg-green-900 rounded-xl border-2 border-green-400">
    <div class="text-4xl mb-4">👩‍💻</div>
    <div class="text-green-400 font-bold mb-2">Application Developers</div>
    <div class="text-sm text-gray-300">Business logic & features</div>
  </div>
</div>

<!--
Now here's the big one – the User-facing OpenTelemetry Logging API.
This is the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

[click] There is no need to go through traditional logging libraries and have the bridge overhead.

[click] You can use complex attributes and 

[click] You can also reuse the attributes accross signals.

[click] It is critical for libraries and frameworks adding OpenTelemetry instrumentation to make sure to not couple it to any other library like log4j.

[click] But it is all the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

The best part? It plays nicely with existing logging.
You can gradually migrate, use both approaches, or pick the right tool for each job.

It's not about replacing everything. It's about giving you options.

And yes, it takes forever to get right.
Designing APIs that developers will actually want to use is hard.
Making them consistent across 10+ programming languages?
Even harder.
But we're getting there.
-->

---
layout: center
---

# `Enabled` API

<div class="grid grid-cols-2 gap-8 text-lg">

<div v-click class="text-left">

Before ❌

```go
// Always does the work
logger.Info(ctx, "User operation",
  formatMessage(),             // CPU cycles!
  buildComplexAttributes()...) // Memory allocs!
```

<div class="text-red mt-4">
  🐌 Wasted CPU cycles<br/>
  📈 Memory allocations<br/>
</div>

</div>

<div v-click class="text-left">

After ✅

```go
// Check first, work later
if logger.InfoEnabled(ctx) {
  logger.Info(ctx, "User operation",
    formatMessage(),          // Smart!
    buildComplexAttributes()) // Only when needed!
}
```

<div class="text-green mt-4">
  ⚡ Zero overhead when disabled<br/>
  🎯 Work only when necessary
</div>

</div>

</div>

<!--
Now, let's talk about performance.

[click]
Because what good is amazing structured logging if it kills your application's performance?

[click]
OpenTelemetry recently added Enabled API.
It is a way to check if logging is actually needed before you do all the expensive work of building log records.
Think of it as "measure twice, cut once" but for telemetry.

The idea is simple: before you spend CPU cycles formatting messages, serializing complex objects, or building those beautiful structured attributes, you ask: "Hey, is anyone actually going to process this log?".
If the answer is no, you skip all the work.

-->

---
layout: center
---

# `Enabled` functionality

<div class="text-left mx-auto space-y-6">

<v-clicks>

⚡ Check before you emit records to skip expensive work

🚀 A door for better integrations

<Youtube id="Ej-z2WwWWak" />



</v-clicks>

</div>

<!--

But here's where it gets really exciting.

[click]
This Enabled functionality doesn't just make the OpenTelemetry Logs faster.

[click]
It opens the door to hooking into extremely performant, OS-native tracing systems like Linux user_events and 
ETW (Event Tracing for Windows).
These systems are so fast they make regular logging look like writing with a pen.

[click]
If you want the deep dive, check out the presentation by Cijo Thomas & Chris Gray: "Beyond OTLP: Unlocking the Potential of OS-native Tracing".
-->

---
layout: center
---

# `tl;dr;`

### syntax

📝 Logs are more than strings

⚡ Structure unlocks superpowers  

### semantics

🎯 Semantic conventions = consistency

✨ Events are logs with well-known structure

### api

🔧 Direct usage

🚀 Performance and integration improvements

<div class="text-xl mt-8">
  <span class="text-orange">Performance</span> + 
  <span class="text-blue">Rich telemetry</span> +
  <span class="text-green">Integrations</span> = 💖
</div>

<!--
What did we learn today?
Let me summarize this journey through the chaotic world of logs.

We've all been writing trash logs. "LOGGING HERE" with dashes?
We've ALL been there. But trash logs cost money, slow down apps, and make observability engineers cry into their coffee.

In OpenTelemetry, logs are structured and can contain complex data. Moreover, complex attributes are going to also arrive to other signals. OpenTelemetry has semantic conventions that not only define the attributes, but also distinguish between log and event records. Logs are getting faster too.

That's the kind of evolution that should make everyone happy: developers get better observability, operations get cleaner data.
-->

---
layout: center
class: text-center
---

# Questions?

<div class="text-6xl mb-8">🙋‍♀️🙋‍♂️</div>

<!--
Alright, that was a lot of information about logs, events, complex data.

I apparently spend too much time talking.

Time for questions! Fire away!
-->

---
layout: center
---

# Thank You!

This presentation: <https://otel-logs-talk.netlify.app>

<div class="text-lg mx-auto space-y-2">

<span v-click class="text-2xl">Special thanks 🙏</span>

<v-clicks>

- Liudmila Molkova, Trask Stalnaker, Ted Young, Austin Parker
- Tyler Yahn, Cijo Thomas
- OpenTelemetry Go and Specification SIG
- Everyone who raised their hands
- GenAI (sic!)

</v-clicks>
</div>

<div v-click>
<div class="mt-8 text-xl text-red">
  Don't let your logs be trash 🗑️
</div>
<div class="text-2xl mb-2 text-green">
  May your logs be structured! 🎉
</div>
<div class="flex text-2xl gap-2">
  <img src="./opentelemetry-logo-nav.png" alt="OpenTelemetry" class="h-8" />
  <strong>
    <span class="text-orange">Open</span>
    <span class="text-blue">Telemetry</span>
  </strong>
  ❤️
</div>
</div>

<br>

<div v-click class="text-xl">

**GitHub**: [@pellared](https://github.com/pellared)

[**CNCF Slack**](https://slack.cncf.io/): Robert Pajak
</div>
