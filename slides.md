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

---
layout: center
class: text-center
---

<div class="text-5xl text-left mx-auto space-y-18">

<span v-click.hide>🕰️ Past (Deprecated)</span>

📍 Present (Stable)

🚀 Future (Development)

</div>

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

⏰ Timestamp

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
  <span class="text-green">✨</span> Rich, structured, meaningful data!
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

<h1>
An Event Record<br>
is a Log Record<br>
with an <span class="text-orange">event name</span><br>
and a <span class="text-blue">well-known structure</span>.
</h1>

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
  💸 Wasted CPU cycles<br/>
  📈 Memory allocations<br/>
  🐌 Performance impact
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
  🚀 OS-native performance<br/>
  🎯 Work only when necessary
</div>

</div>

</div>

---
layout: center
---

# `Enabled` functionality

<div class="text-left mx-auto space-y-6">

<v-clicks>

⚡ Check before you emit records to skip expensive work

🚀 A door for better integrations

<div class="text-xl mt-8">
  <span class="text-orange">Performance</span> + 
  <span class="text-blue">Rich telemetry</span> +
  <span class="text-green">Integrations</span> = 💖
</div>

<Youtube id="Ej-z2WwWWak" />

</v-clicks>

</div>

---
layout: center
---

# `tl;dr;`

<v-clicks>

### syntax

📝 Logs are more than strings

⚡ Structure unlocks superpowers  

### semantics

🎯 Semantic conventions = consistency

✨ Events are logs with well-known structure

### api

🔧 Direct usage

🚀 Performance and integration improvements

</v-clicks>

---
layout: center
class: text-center
---

# Questions?

<div class="text-6xl mb-8">🙋‍♀️🙋‍♂️</div>

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
