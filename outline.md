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

But here's the kicker: these aren't isolated improvements.
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

