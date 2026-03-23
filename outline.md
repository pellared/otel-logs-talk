# OpenTelemetry Logs – Speaker Notes Outline

## Title slide

Hi everyone!
I want to start with a disclaimer.
This an updated version of my talk that I gave a few months ago.

## trash logs

Let me begin with a quick question.
Please raise your hand if you have seen logs like this.

*Say what part of the audience have their hands up.*

We've ALL been there.
"LOGGING HERE" with dashes, passwords in debug logs.

## THIS is why we need better logging standards

Wasted logs cost money, slow down applications, and add noise.

THIS is why we need better logging standards.

## on the other side...

But on the other side of the spectrum, far away from these noisy logs, there are technologies that have been doing structured event telemetry right for decades.

OS kernels and language runtimes emit events with known structure, typed fields, and near-zero overhead.

The lesson from these systems? Structure and performance are not opposites. You can have both.
That's exactly the direction we are aiming.

## uname OpenTelemetry

When I talk about "we", I mean OpenTelemetry.

For those who might be new to OpenTelemetry, let me give you a quick overview.

OpenTelemetry is a framework for generating, processing, and exporting telemetry data.
It provides a unified approach to observability across different signals such as traces, metrics, logs, events, profiles.

The beauty of OpenTelemetry is that it gives you one standard way to instrument your applications, regardless of the language or backend you're using.
Whether you're sending data to Jaeger, Prometheus, Splunk, or any other observability platform, OpenTelemetry provides the common foundation.

## whoami

Ah, my bad manners.
Maybe I should introduce myself?

[click] I am Robert Pająk. You can find me on GitHub as pellared.

[click] I've been actively contributing to OpenTelemetry Logs since November 2023. Yet, I have been working with different logging systems for a lot longer than that.

[click] I am responsible for designing and developing OpenTelemetry Go Logs.

[click] I am also part of the OpenTelemetry Logs Special Interest Group that works on the development of OpenTelemetry Logs and Events across the whole OpenTelemetry ecosystem.

[click] I'm a Software Engineer at Splunk, a company that embraces logging, where I get paid for working on open source.

[click] I am not an English native speaker.
But if something is not clear, don't blame my English.
It simply means I have no idea what I am talking about.

## init()

During the talk we are going to explore the following areas in the context of OpenTelemetry:

[click] the data model of logs and Logs API,

[click] the semantics of logs and events,

I hope that by the end of this talk, you'll understand why logs in OpenTelemetry are no longer just about bridging your logs from logging libraries and frameworks into OpenTelemetry SDK, but a first-class citizen among other OpenTelemetry signals. We are also going to see how it is affecting other areas of OpenTelemetry.

## syntax

What does a proper log record look like in OpenTelemetry? Let's talk about the Logs Data Model.

## OpenTelemetry Logs Data Model

In OpenTelemetry, a log record isn't just a string with a timestamp. It's a structured piece of telemetry with:

[click] timestamps when it happened and when it was captured

[click] trace context for correlation magic

[click] event name that identifies the type of the log record

[click] proper severity level

[click] human-readable log message

[click] any additional metadata or log fields

[click] Resource allows identifying the software that emitted the log record

[click] Instrumentation scope containing information about the source code that emitted the log record

[click]
This isn't just making your regular logging more complicated.

[click]
It is structured logging that actually integrates with your traces.

## complex attributes

The attribute values in OpenTelemetry logs can contain complex data structures. Not just simple strings and numbers.

Here are some examples of complex attributes based on the GenAI semantic conventions.

As you can see, you can log entire payloads, complex error objects, or structured events without flattening them into strings.

There would be no need to reconstruct object hierarchies from JSON-encoded strings.

## Complex values are supported across ALL signals

But wait, there's more! These complex values are supported across ALL OpenTelemetry signals: traces, metrics, and profiles.

This is no longer just a logging feature.

## log vs event

OpenTelemetry distinguishes between Log Records and Event Records.

Log Record is for general-purpose logging, human-readable, "something happened."

Event Record is a well-defined occurrence with stronger semantics, "this specific thing happened."

Why the distinction?

Because not every log your application emits deserves to be treated like something meaningful.

Events are intentional, instrumented with purpose, not sprinkled around like debugging salt.

This matters because platforms can now treat Events as first-class citizens.

You can index them differently, correlate them intelligently, and maybe even generate useful alerts instead of just noise.

## An Event Record is a Log Record with an event name and a well-known structure

You can check out the referenced blog post for more context.

## User-facing logging API

Now let's talk about the user-facing OpenTelemetry logging API.
Until recently, OpenTelemetry Logs were mostly about getting logs from file streams or bridging logs from existing logging frameworks like log4j, Serilog, or Python's logging module.
This is the piece that finally gives developers a proper, first-class way to emit logs directly through OpenTelemetry.

[click] There is no need to go through traditional logging libraries and have the bridge overhead.

[click] You can use complex attributes.

[click] You can also reuse the attributes across signals.

[click] It is critical for libraries and frameworks adding OpenTelemetry instrumentation to make sure to not couple it to any other library like log4j.

[click] But it is also the piece that finally gives developers a proper, first-class way to emit logs and events directly through OpenTelemetry.

The best part? It plays nicely with existing logging.
You can gradually migrate, use both approaches, or pick the right tool for each job.

It's not about replacing everything. It's about giving you options.

## Enabled API

Now, let's talk about performance.

[click]
Because what good is amazing structured logging if it kills your application's performance?

[click]
OpenTelemetry recently added an Enabled API.
It is a way to check if logging is actually needed before you do all the expensive work of building log records.
Think of it as "measure twice, cut once" but for telemetry.

The idea is simple: before you spend CPU cycles formatting messages, serializing complex objects, or building those beautiful structured attributes, you ask: "Hey, is anyone actually going to process this log?"
If the answer is no, you skip all the work.

## Enabled API is being added to ALL signals

But here's where it gets really exciting.

Enabled has been added to other signals to offer the same functionality.

## span events → log events

All of this progress around logs and events leads us to one of the biggest upcoming changes in OpenTelemetry: the deprecation of the Span Events API in favor of emitting logs through Logs API.

## Two ways to emit events is one too many

Until now, OpenTelemetry offered two competing ways to emit events correlated with traces:

[click] Span Events, created by calling Span.AddEvent or Span.RecordException on a span through the Tracing API.

[click] Log-based Events, emitted through the Logs API and correlated with the active trace context.

[click] Having both creates split guidance for instrumentation authors, duplicate mental models for users, and means every improvement to the event model has to be specified and implemented twice.

## What is changing

Here is what is actually changing:

[click] The Tracing API methods Span.AddEvent and Span.RecordException are going to be deprecated.

[click] The Logs API is going to be the single recommended way to emit events, including exceptions.

[click] SDKs will provide a compatibility layer that can project log-based events back onto spans, so backends that show events in trace views keep working.

[click] Semantic conventions will migrate events and exceptions to log-based definitions in their next major versions, while keeping existing behavior stable.

[click] The plan is to deprecate the API for recording span events, not the span event data model itself. Existing span event data stays valid, and backends can still surface events in span timeline views. What changes is the recommended way to write new events. Our goal is to make events in OpenTelemetry simpler, more consistent, and more powerful, while providing a graceful transition to the new recommendations.

## semantics (section)

We've got structure.
But how do we make sure everybody calls things the same way so we can actually search, correlate, merge data from five services written by five teams?

## OpenTelemetry Semantic Conventions

That's where semantic conventions come in.

Think of them as telemetry guidelines containing a giant shared dictionary of attribute names that all SDKs, libraries, instrumentation, and software agree to use.

## Semantic conventions registry

The semantic conventions registry is organized into groups.

[click] Cross-cutting attributes apply across many signals and use cases: code.line_number tells you where it broke, error.type gives you the actual error, and service.name tells you what application broke.

[click] Signal-specific attributes only make sense for a particular signal, like log.record.uid or log.iostream for log records.

[click] Domain-specific attributes are focused on protocols, systems, and shiny new tech like HTTP, databases, or GenAI.

## Semantic conventions also guide *how* to emit

Semantic conventions don't just define attributes.
They also specify guidelines and best practices for how to emit telemetry correctly.

[click] Event conventions define the expected structure of an event: required fields, recommended fields, and how to pick the right severity level, and what the body should contain.

[click] For example, the exception event convention tells you exactly how to record an error: which attributes are required, how to capture the stack trace.

[click] Domain specific, e.g. GenAI semantic conventions, defining how to capture AI prompts, responses, and tool calls as structured events so any platform can understand and query them.

[click] In short, semantic conventions guide not just naming, but behavior — ensuring every library and SDK emits telemetry in a consistent, interoperable way.

## tl;dr;

What did we learn today?

[click] Logs are more than strings, they carry structure, context, and typed attributes.

[click] Complex attributes are now supported across ALL signals, not just logs.

[click] Events are logs with a well-known structure and an event name.

[click] The Logs API can be used directly. There is no need to go through a logging library.

[click] The Enabled API is being added to ALL signals to avoid doing expensive work when telemetry is going to be discarded.

[click] Span Events are being deprecated in favor of log-based events via the Logs API.

[click] Follow semantic conventions guidelines and attributes registry.

[click] We want logging in OpenTelemetry to be performant, structured, and possible to be integrated with robust event tracing systems.

## Thank You!

Thank you all for listening.
I hope you learned something useful today.

[click] Now go forth and log responsibly!

[click] Find me on GitHub as pellared or CNCF Slack as Robert Pajak.
