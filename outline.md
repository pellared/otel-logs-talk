# OpenTelemetry Logs Driving a Major Shift: Events, Richer Data, and Smarter Semantics

Welcome to my talk about how OpenTelemetry Logs evolved from the least mature signal to potentially become the foundation of major new enhancements.

## whoami

I've traveled from the beautiful city of Kraków (Cracow).

You can find me lurking around GitHub as pellared, probably creating some issue or opening yet another pull request.

I'm a Software Engineer at Splunk (a company aquired by Cisco), which means I get paid for contributing to open source.

I am an OpenTelemetry Go maintainer.
I'm also an OpenTelemetry Specification sponsor.
Fun fact: I'm apparently the 3rd top OpenTelemetry Contributor according to Linux Foundation Insights.
I'm not sure if that's impressive or just means I need better hobbies.

I volunteered to design OpenTelemetry Go Logs and improve the OpenTelemetry Logs Specification in November 2023. 
Apparently, someone thought I actually knew what I was doing.

I am not an English native speaker.
But if something is not clear, don't blame my English.
It simply means I have no idea whay I am talking about.

## uname OpenTelemetry

For those who might be new to OpenTelemetry, let me give you a quick overview.

OpenTelemetry is a framework for generating, processing, and exporting telemetry data.
It provides a unified approach to observability across different signal such as traces, metrics, logs, profiles.

The beauty of OpenTelemetry is that it gives you one standard way to instrument your applications, regardless of the language or backend you're using.
Whether you're sending data to Jaeger, Prometheus, Splunk, or any other observability platform, OpenTelemetry provides the common foundation.

## init()

People usually talk about the past, present, and future.

But honestly, who cares about the past?
We're here to talk about what's happening NOW and what's coming NEXT.
We're not just talking about logs in isolation.
We're talking about how logs and how they're driving major improvements across the entire OpenTelemetry ecosystem.

During the talk are are going to explore the following areas in the context of OpenTelemetry:

the syntax (or data model) of logs,

the semantics of logs and events,

the logging APIs.

We'll dive into the design decisions, and emerging patterns that are turning logs from "legacy baggage" into the foundation for the next generation of OpenTelemetry-powered insights.
By the end of this talk, you'll understand why logs are no longer an afterthought, but a first-class citizen among other OpenTelemetry signals.

## trash logs

Show hands: Who has seen logs like this?
Keep your hands up!
Don't worry, nobody will judge you.

*Say what part of the audience have their hands up.*

I've written logs like this too.
We've ALL been there.
"LOGGING HERE" with dashes?
Classic debugging technique.
Passwords in debug logs?
Academic excellence right there.

But seriously, this is what we're working with in production systems everywhere.
Note that wasted logs cost money.
There's nothing like a free meal, and every useless "It works" message is burning through your observability budget.
Plus, it slows down your application.
Moreover, it adds noise to the captured telemetry.
THIS is why we need better logging standards.

## syntax

What does a proper log record look like in OpenTelemetry? Let's talk about the Logs Data Model.

In OpenTelemetry, a log record isn't just a string with a timestamp. It's a structured piece of telemetry with:

- the timestamps when it happened and when it was captured
- the trace and span IDs for correlation magic
- not just "INFO/WARN/ERROR" but proper severity levels
- the actual log message or payload that can be complex
- the key-value pairs for structured metadata
- the event name identifies the type of event being recorded - we are going to discuss it later
- the information what application and which library or component generated this log

This isn't just making your regular logging more complicated.
It is structured logging that actually integrate with your traces.

Here you can see the timestamp when the something happened and when it was recorded in OpenTelemetry.

As you can see the log record contains the trace context.

The severity is a level which is a number, but it is also a text so that the names from logging libraries can be preserved.

Body is typically the log message.

Attributes are any additional metadata or log fields.

Resource contains the data boostrapped by OpenTelemetry SDK that allows identifying the software that emited the log record.

Instrumentation scope contains information about the source code that emitted the log record.

## complex data

The Body and Attributes fields in OpenTelemetry logs can contain complex data structures.
Not just simple strings and numbers that make you cry at 3 AM.

You can have:
- **Nested objects** with multiple levels of structure (like a Russian doll, but useful)
- **Arrays** of values or objects (finally, proper lists!)  
- **Mixed data types** within the same attribute (chaos, but organized chaos)

This means you can log entire payloads, complex error objects, or structured events without flattening them into strings.
Remember the dark days of parsing "user=alice,role=admin,lastLogin=2023-01-01"?
Yeah, we don't talk about those anymore.
No more trying to reconstruct object hierarchies from flat attributes like some kind of archaeological dig through your own code.

But wait, there's more! These complex values are coming to ALL OpenTelemetry signals: traces, metrics, and profiles.
This won't be just a logs feature.
It's like getting a free upgrade to business class for your entire observability stack.
Imagine spans with complex attributes that can hold entire request/response objects.
We're talking about a unified approach to complex data across all telemetry signals.
Getting agreement that this is even acceptable took about a year of proposals, bikeshedding, and "is this not breaking?" discussions.
It's like finally getting your whole family to agree on pizza toppings: miraculous and life-changing.

## semantics

We've got structure and complex data.
But how do we make sure everybody calls things the *same* way so we can actually search, correlate, and NOT cry when we merge data from five services written by five teams?
That's where **semantic conventions** come in.

Think of them as a giant shared dictionary of attribute names that all SDKs, libraries, and instrumentation agree to use.
Without them, we'd have `userId`, `user_id`, `uid`, `userid`, and `id` all meaning the same thing.

The **semantic conventions registry** is organized into groups:

**Cross-cutting attributes** – apply across many signals and use cases:
- `code.line_number` – because knowing where it broke is half the battle
- `exception.message` – the actual error
- `service.name`, `deployment.environment` – so you know if you broke production or just staging

**Signal-specific attributes** – e.g. here are ones that make sense only for log records:
- `log.record.uid` – stable unique ID for the log record
- `log.iostream` – stdout, stderr, or that custom log stream you regret creating

**Domain-specific attributes** – focused on protocols, systems, and shiny new tech:
- `http.method`, `http.request.body.size` – web stuff
- `db.system`, `db.statement` – database things
- `gen_ai.provider.name`, `gen_ai.response.id` – yes, even AI has semantics now

## event vs log

OpenTelemetry distinguishes between **Log Records** and **Event Records**.

Log Record is for general-purpose logging, human-readable, "something happened."
Event Record is a well-defined occurrence with stronger semantics, "this specific thing happened."

An Event Record is a Log Record with an event name and a well-known structure.

Why the distinction?
Because not every line your app spits out deserves to be treated like a meaningful business event.
Events are intentional, instrumented with purpose, not sprinkled around like debugging salt.

This matters because platforms can now treat Events as first-class citizens.
You can index them differently, correlate them intelligently, and maybe even generate useful alerts instead of just noise.

## spans & metrics from events

Jeremy Morrell wrote a nice blog post called "A Practitioner's Guide to Wide Events" that basically says: "What if we just logged everything we need in really rich, wide events?"

The idea is simple but powerful: instead of having traces, metrics, and logs as separate things, you emit super-detailed log events that contain ALL the context you need.
Then you can derive spans and metrics from these logs with proper semantics.

Think about it:
- Want a span?
  Extract the start/end timestamps and duration from your events.
  Follow the correlation IDs and build the trace graph.
- Need metrics?
  Aggregate the numerical values in your event attributes.


Now, I have to admit something: I was actually doing this back in 2019.
Not because I was some visionary genius, but because I only had access to an efficient logs backend.
When you're stuck with one tool, you get creative.
Turns out, constraints sometimes lead to good architectural decisions. Who knew?

Of course, there are huge trade-offs.
Storage costs and processing overhead on the backend, bigger network traffic, the existential crisis of "am I doing observability wrong?"
But when it works, it's like having X-ray vision into your systems.
Everything is connected, everything has context, and you can slice and dice your data however you want without wondering if you forgot to add that one crucial attribute to your metric.
And always remember, there is no silver bulet.

## user-facing api

Until recently, OpenTelemetry logs were mostly about getting logs from streams or collecting (bridging) logs from existing logging frameworks (like log4j, Serilog, or Python's logging module) and adding correlation context.
But what if you want to log directly using OpenTelemetry APIs?
What if you want to emit those beautiful structured events we've been talking about?

Now here's the big one – the **User-facing OpenTelemetry Logging API**.
This is the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

Well, now you can! The OpenTelemetry Logs API lets you:

- **Emit log records directly** using OpenTelemetry's structured format
- **Create events** with proper event names and semantic attributes
- **Leverage all the goodies** – complex data types, trace correlation, resource attribution
- **Skip the middleman** – no need to go through traditional logging libraries

You can use the Logs API directly or use some more user-friendly facade.

This is huge because it means:

1. **Direct integration** – your telemetry and application logic can be tightly coupled when needed
2. **No format conversion** – you're already speaking OpenTelemetry's language
3. **Full feature access** – complex attributes, reuse of attributes accross signals, everything we've discussed
4. **Consistency** – same patterns whether you're tracing, logging, or emitting metrics

The best part?
It plays nicely with existing logging.
You can gradually migrate, use both approaches, or pick the right tool for each job.
It's not about replacing everything – it's about giving you options.

And yes, it takes forever to get right.
Designing APIs that developers will actually want to use is hard.
Making them consistent across many programming languages?
Even harder.
But we're getting there.

## enabled

Now, let's talk about performance.
Because what good is amazing structured logging if it kills your application's performance?

OpenTelemetry recently added Enabled API.
It is a way to check if logging is actually needed before you do all the expensive work of building log records.
Think of it as "measure twice, cut once" but for telemetry.

The idea is simple: before you spend CPU cycles formatting messages, serializing complex objects, or building those beautiful structured attributes, you ask: "Hey, is anyone actually going to process this log?"
If the answer is no, you skip all the work.

But here's where it gets really exciting.

It opens the door to hooking into extremely performant, OS-native tracing systems like:

- **Linux user_events** – kernel-level event tracing that's blazingly fast
- **Windows ETW (Event Tracing for Windows)** – Microsoft's high-performance event system

These systems are so fast they make regular logging look like writing with a quill pen.
We're talking about nanosecond-level overhead when events are disabled.

If you want the deep dive into this rabbit hole, check out the presentation by Cijo Thomas & Chris Gray: "Beyond OTLP: Unlocking the Potential of OS-native Tracing" (https://www.youtube.com/watch?v=Ej-z2WwWWak).
Fair warning: after watching it, you might start questioning everything you thought you knew about telemetry performance.

## tl;dr;

The bottom line? OpenTelemetry logs are not just getting smarter and more structured.
They're getting faster too.
And that's the kind of evolution that makes everyone happy: developers get better observability, ops teams get cleaner data, and performance engineers don't have to sacrifice telemetry on the altar of latency.

So what did we learn today?
Let me summarize this journey through the wonderfully chaotic world of OpenTelemetry Logs:

**The Problem:**
We've all been writing trash logs.
"LOGGING HERE" with dashes?
We've ALL been there.
But trash logs cost money, slow down apps, and make observability engineers cry into their coffee.

**The Solution:**
OpenTelemetry logs are no longer the weird cousin at the observability family dinner.
They're now:

- **Structured** – with actual fields that make sense
- **Complex** – nested objects and arrays
- **Semantic** – everyone agrees on attribute names
- **Event-aware** – distinguishing between "something happened" and "SOMETHING IMPORTANT happened"
- **Fast** – with Enabled functionality and OS-native integration
- **Developer-friendly** – finally, an API you might actually want to use

**The Punchline:**
Logs went from being the least mature OpenTelemetry signal to potentially the foundation for everything else.
It's like that awkward kid in school who suddenly became really cool and now everyone wants to hang out with them.

**The Reality Check:**
There's no silver bullet.
Storage costs, processing overhead, and existential crises about "am I doing observability wrong?" are still real.
But at least now you have options that don't involve archaeological digs through your own log files.

Remember: logs are no longer an afterthought.
They're the secret sauce for better correlation, richer context, and fewer 3 AM debugging sessions where you stare at "something went wrong" and question your life choices.

## q&a

Alright, that was a lot of information about logs, events, complex data.
I apparently spend too much time talking.

Time for questions! 

Don't be shy – I promise I won't judge your logging practices too harshly.
After all, we've all been the person who wrote `console.log("HERE")` and forgot to remove it before deploying to production.

Fire away!

## thanks

Thank you all for listening.
I hope you learned something useful today, or at least got a few laughs.

Special thanks to:

- **Liudmila Molkova, Trask Stalnaker, Ted Young, Austin Parker, Tyler Yahn, Cijo Thomas** who continously help with all the logs work
- **OpenTelemetry Go SIG** who give me a lot of feedback about the logs implementation in OTel Go
- **The entire OpenTelemetry community** for reviewing my pull requests and providing a lot of feedback
- **Splunk** for paying me to contribute to open source (best job ever!)
- **Everyone who raised their hands** during the "trash logs" section; your honesty is appreciated

And finally, thank you to **all the developers who've ever written "LOGGING HERE" with dashes**.
Without you, I wouldn't have a career in observability.

You can find me on GitHub as **@pellared** and on CNCF Slack as **Robert Pajak**, if you have questions that didn't fit into today's talk.

Now go forth and log responsibly!
