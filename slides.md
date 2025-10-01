---
theme: default
title: OpenTelemetry Logs Driving a Major Shift
class: text-center
transition: slide-left
mdc: true
---

# OpenTelemetry Logs Driving a Major Shift

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

<!--
Welcome to my talk about how OpenTelemetry logs evolved from the least mature signal to potentially the foundation of everything else.
-->

---
layout: center
---

# `whoami`

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">

<v-clicks>

🏠 Kraków, Poland

‍💻 @pellared on GitHub

💼 Software Engineer at Splunk a Cisco company

🔧 OpenTelemetry Go maintainer

🌟 3rd top OTel contributor

🌴 Contributing to OTel Logs since Nov 2023

<div class="text-lg mt-8 opacity-80">
  <strong>Disclaimer:</strong> Non-native English speaker.<br/>
</div>

</v-clicks>

</div>

---
layout: center
class: text-center
---

<div class="text-5xl text-left mx-auto space-y-18">

<span v-click.hide>🕰️ Past</span>

📍 Present

🚀 Future

</div>

---
layout: center
---

# `init()`

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">

<v-clicks>

✨ OpenTelemetry Logs Data Model

🎯 Complex values

📚 Semantic conventions

⚡  Events vs Logs

🔧 User-facing logging API

🚀 Enabled functionality

</v-clicks>

</div>

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

---
layout: statement
---

# THIS is why<br>we need better logging standards

---
layout: section
---

# syntax

---
layout: center
class: text-center
---

# OpenTelemetry Logs Data Model

<div class="text-2xl text-left mx-auto space-y-6 max-w-3xl">
<v-clicks>

⏰ Timestamp

🔗 Trace Context

📊 Severity

📝 Body

🏷️ Attributes

🎯 Event Name

📍 Resource & Instrumentation Scope

<div class="text-xl mb-6 text-blue-400">
  Not just strings with timestamps.
</div>
<div class="mt-8 text-xl font-bold text-green-400">
  Logs that carry context. Logs that make sense! ✨
</div>

</v-clicks>
</div>

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

---
layout: center
---

# complex data

`Attribute` and `Body` values can be complex.

`AnyValue` can be one of:

<v-clicks>

- primitive value (`string`, `int`, `bool`, `double`)
- array of `AnyValue`
- map of `string` to `AnyValue`
- byte array (BLOB)
- empty value

</v-clicks>

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

<div class="mt-6 text-lg">
  <span class="text-green-400">✨</span> Rich, structured, meaningful data!
</div>

---
layout: statement
---

# Complex values<br>are coming<br>to ALL signals!

OTel is dropping the attributes values restriction

---
layout: section
---

# semantics

---
layout: statement
---

# OpenTelemetry<br>Semantic Conventions

Making logs searchable & comparable

<div class="text-2xl font-bold text-blue-400 mb-6">
  Same semantics = Better queries and dashbaords
</div>

---
layout: center
---

# Cross-cutting attributes

- `code.line_number`
- `exception.message`
- `service.name`, `deployment.environment`

---
layout: center
---

# Signal-specific attributes

- `log.record.uid`
- `log.iostream` 

---
layout: center
---

# Domain-specific attributes

- `http.method`, `http.request.body.size`
- `db.system`, `db.statement`
- `gen_ai.provider.name`, `gen_ai.response.id`

---
layout: section
---

# event vs log

---
layout: statement
---

# An Event Record<br>is a Log Record<br>with an event name<br>and a well-known structure

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

<div class="text-center mt-8 text-xl text-blue-400">
  Same data model, different semantics.
</div>

---
layout: statement
---

# spans & metrics from events

<v-clicks>

<div class="text-left">

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

<div v-click class="text-xl mb-8 text-red-400">
⚠️ There are trade-offs ⚠️
</div>

<div>
Read more about: <code>Canonical Log Lines</code> and <code>Wide Events</code>.
</div>

</v-clicks>

---
layout: section
---

# api

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

<div class="text-center mt-6 text-lg text-blue-400">
  For instrumentation authors as well as application developers
</div>

---
layout: center
---

# `Enabled` functionality

<div class="text-left mx-auto space-y-6">

<v-clicks>

⚡ Check before you emit records to skip expensive work

🚀 A door for better integrations

<div class="text-xl mt-8 text-blue-400">
  Performance + Rich telemetry + Integrations = 💖
</div>

</v-clicks>

</div>

---
layout: center
---

<div class="grid grid-cols-2 gap-8 text-lg">

<div v-click class="text-left">

### Before ❌

```go
// Always does the work
logger.Info(ctx, "User operation",
  formatMessage(),             // CPU cycles!
  buildComplexAttributes()...) // Memory allocs!
```

<div class="text-red-400 mt-4">
  💸 Wasted CPU cycles<br/>
  📈 Memory allocations<br/>
  🐌 Performance impact
</div>

</div>

<div v-click class="text-left">

### After ✅

```go
// Check first, work later
if logger.InfoEnabled(ctx) {
  logger.Info(ctx, "User operation",
    formatMessage(),          // Smart!
    buildComplexAttributes()) // Only when needed!
}
```

<div class="text-green-400 mt-4">
  ⚡ Zero overhead when disabled<br/>
  🚀 OS-native performance<br/>
  🎯 Work only when necessary
</div>

</div>

</div>


---
layout: center
class: text-center
---

# TL;DR 📝

<div class="text-2xl mb-8 text-blue-400">
  OpenTelemetry Logs: The Missing Piece
</div>

<div class="grid grid-cols-2 gap-8 text-lg max-w-4xl mx-auto text-left">

<div>

### What We Learned 🧠
- Logs are more than strings
- Structure unlocks superpowers  
- Semantic conventions = consistency
- Events = structured logs
- Wide events = fewer, richer logs

</div>

<div>

### What You Get 🎁
- Cross-service correlation
- Better dashboards & alerts
- Faster debugging
- ML/AI ready data
- Future-proof observability

</div>

</div>

<div class="mt-8 text-xl text-green-400">
  **Ready to level up your logging game?** 🚀
</div>

---
layout: center
class: text-center
---

# Questions? 🤔

<div class="text-6xl mb-8">🙋‍♀️🙋‍♂️</div>

<div class="text-2xl mb-6 text-blue-400">
  Let's chat about logs, traces, and everything observability!
</div>

<div class="text-lg space-y-2">
  
**Twitter/X**: @ropajak_robert  
**Email**: robert@example.com  
**LinkedIn**: /in/robertropajak

</div>

<div class="mt-8 text-xl text-green-400">
  Don't let your logs be trash! 🗑️➡️✨
</div>

---
layout: center
class: text-center
---

# Thank You! 🙏

<div class="text-6xl mb-8">🎉</div>

<div class="text-2xl mb-6 text-blue-400">
  May your logs be structured<br/>
  and your traces be complete!
</div>

<div class="mt-8 text-xl">
  <span class="text-red-500">❤️</span> 
  <span class="text-blue-400">OpenTelemetry</span>
  <span class="text-red-500">❤️</span>
</div>
