# OpenTelemetry Logs Driving a Major Shift: Events, Richer Data, and Smarter Semantics

Welcome to my talk about how OpenTelemetry Logs evolved from the least mature signal to potentially become the foundation of major new enhancements.

## whoami

Hello, I am Robert Pająk.

I've traveled from the beautiful city of Kraków.

You can find me on GitHub as pellared, probably creating some issue or opening yet another pull request.

I'm a Software Engineer at Splunk (a company acquired by Cisco), which means I get paid for contributing to open source.

I am an OpenTelemetry Go maintainer and Specification sponsor.

I've been contributing to OpenTelemetry Logs since November 2023.

I am not an English native speaker.
But if something is not clear, don't blame my English.
It simply means I have no idea what I am talking about.
Or that I am bad at reading.

## past, present, future

People usually talk about the past, present, and future.

But honestly, who cares about the past?

We're here to talk about what's happening NOW and what's coming NEXT.

## uname OpenTelemetry

For those who might be new to OpenTelemetry, let me give you a quick overview.

OpenTelemetry is a framework for generating, processing, and exporting telemetry data.
It provides a unified approach to observability across different signals such as traces, metrics, logs, profiles.

The beauty of OpenTelemetry is that it gives you one standard way to instrument your applications, regardless of the language or backend you're using.
Whether you're sending data to Jaeger, Prometheus, Splunk, or any other observability platform, OpenTelemetry provides the common foundation.

## init()

During the talk we are going to explore the following areas in the context of OpenTelemetry:

- the data model of logs,
- the semantics of logs and events,
- the logging APIs.

We'll dive into the design decisions, and emerging patterns that are turning logs from "legacy baggage" into the foundation for the next generation of OpenTelemetry-powered insights.

By the end of this talk, you'll understand why logs are no longer an afterthought, but a first-class citizen among other OpenTelemetry signals.

## trash logs

Show hands: Who has seen logs like this?
Keep your hands up!
Don't worry, nobody will judge you.

*Say what part of the audience have their hands up.*

We've ALL been there.
"LOGGING HERE" with dashes, passwords in debug logs.

# THIS is why we need better logging standards

Wasted logs cost money, slow down applications, and add noise.
THIS is why we need better logging standards.

## syntax

What does a proper log record look like in OpenTelemetry? Let's talk about the Logs Data Model.

## OpenTelemetry Logs Data Model

In OpenTelemetry, a log record isn't just a string with a timestamp. It's a structured piece of telemetry with:

- timestamps when it happened and when it was captured
- trace context for correlation magic
- proper severity level
- actual log message or payload
- any additional metadata or log fields
- Resource allows identifying the software that emitted the log record
- Instrumentation scope containing information about the source code that emitted the log record

This isn't just making your regular logging more complicated.

It is structured logging that actually integrates with your traces.

## complex data

The Body and Attributes fields in OpenTelemetry logs can contain complex data structures.
Not just simple strings and numbers.
The value can be:

- primitive value like text, number or boolean value,
- an array of any value, meaning that the first item in an array can be a string and the second item can be an integer
- a map of string to any value, meaning you can compose any nested structure like a Matryoshka doll,
- or even a binary object,
- at last it can be nothing.

You can approximate it to something like BSON format.

## Complex values

Here are some examples of complex attributes.

This means you can log entire payloads, complex error objects, or structured events without flattening them into strings.

No more trying to reconstruct object hierarchies from flat attributes like some kind of archaeological dig through your own code.

## Complex values are coming to ALL signals!

But wait, there's more! These complex values are coming to ALL OpenTelemetry signals: traces, metrics, and profiles.

This won't be just a logging feature.

It's like getting a free upgrade to business class for your entire observability stack.

Imagine spans with complex attributes that can hold entire request/response objects.
We're talking about a unified approach to complex data across all telemetry signals.

## semantics

We've got structure.
But how do we make sure everybody calls things the same way so we can actually search, correlate, and NOT cry when we merge data from five services written by five teams?

## OpenTelemetry Semantic Conventions

That's where semantic conventions come in.

Think of them as telemetry guidelines with a giant shared dictionary of attribute names that all SDKs, libraries, instrumentation, and software agree to use.

The semantic conventions registry is organized into groups.

## Cross-cutting attributes

We have cross-cutting attribute groups like code or exception groups.
They apply across many signals and use cases. For example we have:

code.line_number – because knowing where it broke is half the battle

exception.message – the actual error

service.name, deployment.environment – so you know what application broke and if it is on production or just staging

## Signal-specific attributes

There are also attributes that make sense only for log records.

## Domain-specific attributes

At last, there are attributes focused on protocols, systems, and shiny new tech.

## log vs event

OpenTelemetry distinguishes between Log Records and Event Records.

Log Record is for general-purpose logging, human-readable, "something happened."

Event Record is a well-defined occurrence with stronger semantics, "this specific thing happened."

## An Event Record is a Log Record with an event name and a well-known structure.

## event record vs log record example

Why the distinction?

Because not every log your application emits deserves to be treated like something meaningful.

Events are intentional, instrumented with purpose, not sprinkled around like debugging salt.

This matters because platforms can now treat Events as first-class citizens.

You can index them differently, correlate them intelligently, and maybe even generate useful alerts instead of just noise.

## spans & metrics from events

Jeremy Morrell wrote a nice blog post called "A Practitioner's Guide to Wide Events" that basically says: "What if we just logged everything we need in really rich, wide events?"

The idea is simple but powerful: instead of having traces, metrics, and logs as separate things, you emit super-detailed events that contain ALL the context you need.
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

Of course, there are huge trade-offs of such approach.
Additional storage costs, processing overhead on the backend, bigger network traffic, not mentioning the existential crisis of "am I doing observability wrong?".

But when it works, it's like having X-ray vision into your systems.
Everything is connected, everything has context, and you can slice and dice your data however you want without wondering if you forgot to add that one crucial attribute to your metric.

However, always remember, there is no silver bullet.

You can read about it more online.

## api

Until recently, OpenTelemetry Logs were mostly about getting logs from file streams or bridging logs from existing logging frameworks like log4j, Serilog, or Python's logging module.

But what if you want to log directly using OpenTelemetry APIs?

What if you want to emit those beautiful structured events we've been talking about?

## user-facing api

Now here's the big one – the user-facing OpenTelemetry logging API.
This is the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

There is no need to go through traditional logging libraries and have the bridge overhead.

You can use complex attributes and 

You can also reuse the attributes across signals.

It is critical for libraries and frameworks adding OpenTelemetry instrumentation to make sure to not couple it to any other library like log4j.

But it is all the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

The best part? It plays nicely with existing logging.
You can gradually migrate, use both approaches, or pick the right tool for each job.

It's not about replacing everything. It's about giving you options.

## Enabled API

Now, let's talk about performance.

Because what good is amazing structured logging if it kills your application's performance?

OpenTelemetry recently added an Enabled API.
It is a way to check if logging is actually needed before you do all the expensive work of building log records.
Think of it as "measure twice, cut once" but for telemetry.

The idea is simple: before you spend CPU cycles formatting messages, serializing complex objects, or building those beautiful structured attributes, you ask: "Hey, is anyone actually going to process this log?"
If the answer is no, you skip all the work.

## Enabled functionality

But here's where it gets really exciting.

This Enabled functionality doesn't just make the OpenTelemetry Logs faster.

It opens the door to hooking into extremely performant, OS-native tracing systems like Linux user_events and 
ETW (Event Tracing for Windows).
These systems are so fast they make regular logging look like writing with a pen.

If you want the deep dive, check out the presentation by Cijo Thomas & Chris Gray: "Beyond OTLP: Unlocking the Potential of OS-native Tracing".

## tl;dr;

What did we learn today?
Let me summarize this journey through the chaotic world of logs.

We've all been writing trash logs. "LOGGING HERE" with dashes?
We've ALL been there. But trash logs cost money, slow down apps, and make observability engineers cry into their coffee.

In OpenTelemetry, logs are structured and can contain complex data.
Moreover, complex attributes are going to also arrive to other signals.
OpenTelemetry has semantic conventions that not only define the attributes, but also distinguish between log and event records.
Logs are getting faster too.

That's the kind of evolution that should make everyone happy: developers get better observability, operations get cleaner data.

## q&a

Alright, that was a lot of information about logs, events, and complex data.

I apparently spend too much time talking.
Time for questions!

## thanks

Thank you all for listening.
I hope you learned something useful today.

Special credits to:

- People who continuously help with all the logs work
- Those who helped me a lot during implementation of logs in OTel Go 
- OTel contributors for reviewing my pull requests and providing a lot of comments
- Everyone who raised their hands during the "trash logs" section; your honesty is appreciated. Also thank you for all those that asked questions.
- GenAI and Copilot that helped me a lot in preparing the presentation.

Now go forth and log responsibly!

Find me on GitHub as pellared or CNCF Slack as Robert Pajak.
