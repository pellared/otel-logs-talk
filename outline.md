# OpenTelemetry Logs Driving a Major Shift: Events, Richer Data, and Smarter Semantics

## whoami

I've traveled from the beautiful city of Kraków (Cracow), where the pierogi are legendary and the WiFi is surprisingly good.
You can find me lurking around GitHub as **pellared**, probably opening yet another pull request at 2 AM.

I'm a Software Engineer at Splunk (a company aquired by Cisco), which means I get paid for contributing to open source.
I've been an OpenTelemetry Go maintainer a month or two before I foolishly volunteered to design OpenTelemetry Go Logs in November 2023.
Apparently, someone thought I knew what I was doing.

I'm also an OpenTelemetry Specification sponsor, contributing to logs since January 2024, because clearly I don't have enough meetings in my life.

Fun fact: I'm apparently the 3rd top OpenTelemetry Contributor according to Linux Foundation Insights.
I'm not sure if that's impressive or just means I need better hobbies.

I've been working with logs as one of the most important monitoring signals for almost 10 years now.
Yes, that's a decade of my life spent deciphering cryptic error messages and wondering why developers think "something went wrong" is an acceptable log entry.

Disclaimer: I am not an English native speaker.
But if something is not clear, don't blame my English.
It simply means I have no idea whay I am talking about.

## init()

People usually talk about the past, present, and future.
But honestly, who cares about the past?
We're here to talk about what's happening NOW and what's coming NEXT.

This presentation is about how OpenTelemetry Logs are driving a major shift across the entire OpenTelemetry ecosystem.
We're not just talking about logs in isolation.
We're talking about how logs are becoming the catalyst for foundational changes that reshape how we think about telemetry as a whole.

You'll get an inside look at the following game-changing developments:

- **OpenTelemetry Events** – the recent addition that's redefining structured logging
- **Expanded semantic conventions** – finally aligning with real-world logging use cases  
- **Complex attribute values** – support for nested objects and arrays that actually make sense
- **User-facing OpenTelemetry Logging API** – giving developers a proper way to log with OTel

These aren't isolated improvements.
They represent a coordinated effort to unify and modernize telemetry data, improve correlation across ALL signals, and enable richer observability experiences.

We'll dive into the technical challenges, design decisions, and emerging patterns that are turning logs from "legacy baggage" into the foundation for the next generation of OpenTelemetry-powered insights.

By the end of this talk, you'll understand why logs are no longer an afterthought, but the secret sauce for better trace-log correlation, more consistent metadata across signals, and more expressive data formats that reflect how we actually log in the real world.

## trash logs

```
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

But seriously, this is what we're working with in production systems everywhere.
Note that wasted logs cost money.
There's nothing like a free meal, and every useless "It works" message is burning through your observability budget.
Plus, it slows down your application.
Morevoert, it adds noise to the captured telemetry.
THIS is why we need better logging standards.

## syntax

So what does a proper log look like in OpenTelemetry? Let's talk about the Logs Data Model.

In OpenTelemetry, a log record isn't just a string with a timestamp. It's a structured piece of telemetry with:

- **Timestamp** – when it happened, obviously
- **Observed Timestamp** – when it was actually captured, because clocks drift
- **Trace Context** – the trace and span IDs for correlation magic
- **Severity** – not just "INFO/WARN/ERROR" but proper severity levels
- **Body** – the actual log message (can be structured!)
- **Attributes** – key-value pairs for structured metadata
- **EventName** – identifies the type of event being logged
- **Resource** – who/what generated this log
- **Instrumentation Scope** – which library/component created it

This isn't just regular logging made more complicated.
It's logs that actually integrate with your traces and metrics.
It's logs that carry context. It's logs that make sense.

Here is an example:

```json
{
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
    "exception.message": "Connection is not available, request timed out after 3000ms",
    "exception.stacktrace": "java.sql.SQLTransientConnectionException: Connection is not available, request timed out after 3000ms.\\n\\tat com.zaxxer.hikari.pool.HikariPool.createTimeoutException(HikariPool.java:696)\\n\\tat com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:197)\\n\\tat com.zaxxer.hikari.pool.HikariPool.getConnection(HikariPool.java:162)\\n\\tat com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:128)\\n\\tat org.example.user.repository.UserRepository.findUserById(UserRepository.java:42)\\n\\tat org.example.user.service.UserService.getUser(UserService.java:25)\\n\\tat org.example.user.controller.UserController.getUser(UserController.java:15)\\n\\t... 42 more"
  },
  "Resource": {
    "Attributes": {
      "service.name": "user-service",
      "service.version": "1.5.2",
      "deployment.environment": "staging"
    }
  },
  "InstrumentationScope": {
    "Name": "org.example.user.controller.UserController"
  }
}
```

## complex data

The **Body** and **Attributes** fields in OpenTelemetry logs can contain complex data structures.
Not just simple strings and numbers that make you cry at 3 AM.

You can have:
- **Nested objects** with multiple levels of structure (like a Russian doll, but useful)
- **Arrays** of values or objects (finally, proper lists!)  
- **Mixed data types** within the same attribute (chaos, but organized chaos)

This means you can log entire payloads, complex error objects, or structured events without flattening them into strings.
Remember the dark days of parsing "user=alice,role=admin,lastLogin=2023-01-01"?
Yeah, we don't talk about those anymore.
No more trying to reconstruct object hierarchies from flat attributes like some kind of archaeological dig through your own code.

But wait, there's more! These **complex values are coming to ALL OpenTelemetry signals**: traces, metrics, and profiles.
This isn't just a logs feature.
It's like getting a free upgrade to business class for your entire observability stack.
Imagine spans with complex attributes that can hold entire request/response objects.
We're talking about a unified approach to complex data across all telemetry signals.
Getting agreement that this is even acceptable took about a year of proposals, bikeshedding, and "is this not breakingp?" discussions.
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

Log Record is for general-purpose logging, human-readable, "something happened".
Event Record is a well-defined occurrence with stronger semantics, "this specific thing happened".

An Event Record is a Log Record with an event name and a well-known structure.

Regular log record:

```json
{
  "Body": "User login failed for alice",
  "Attributes": {
    "user.id": "alice",
    "error": "bad password"
  }
}
```

Event record:

```json
{
  "EventName": "user.login.failed", 
  "Attributes": {
    "user.id": "alice",
    "auth.method": "password",
    "error.type": "invalid_credentials"
  }
}
```

Why the distinction?
Because not every line your app spits out deserves to be treated like a meaningful business event.
Events are intentional, instrumented with purpose, not sprinkled around like debugging salt.

This matters because platforms can now treat Events as first-class citizens.
You can index them differently, correlate them intelligently, and maybe even generate useful alerts instead of just noise.
