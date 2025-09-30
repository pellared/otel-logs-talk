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

# whoami

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

<!--
Introduce yourself with humor and credibility. The disclaimer is endearing and sets expectations.
-->

---
layout: center
class: text-center
---

# init()

<div class="text-2xl text-left mx-auto">

<span v-click.hide>🕰️ Past</span>

📍 Present

🚀 Future

</div>

---
layout: center
---

# Game-Changing Developments

<div class="text-lg space-y-4">

  <v-clicks>

  ✨ OpenTelemetry Logs Data Model

  🎯 Complex attribute values

  📚 Semantic conventions

  ⚡  Events vs Log Records

  🌊 Wide Events

  🚀 Enabled functionality

  🔧 User-facing Logging API

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

<div class="text-xl">
Who has seen logs like this? 🙋‍♂️
</div>


---
layout: center
class: text-center  
---

# There's Nothing Like a Free Meal 🍽️

<div class="text-3xl font-bold text-red-400 mb-8">
  Every useless "It works" message<br/>
  burns through your observability budget
</div>

<div class="text-xl">
  <strong>THIS</strong> is why we need better logging standards!
</div>

---
layout: center
class: text-center
---

# OpenTelemetry Logs Data Model

<div class="text-2xl mb-6 text-blue-400">
  Not just strings with timestamps
</div>

<div class="text-lg space-y-2 max-w-2xl mx-auto text-left">
  
⏰ **Timestamp** + **Observed Timestamp** (clocks drift!)

🔗 **Trace Context** (correlation magic)

📊 **Severity** (proper levels, not just INFO/WARN/ERROR)  

📝 **Body** (can be structured!)

🏷️ **Attributes** (key-value metadata)

🎯 **EventName** (event type identifier)

📍 **Resource** + **Instrumentation Scope**

</div>

<div class="mt-8 text-xl font-bold text-green-400">
  Logs that carry context. Logs that make sense! ✨
</div>

---
layout: default
---

# A Proper Log Record

```json {1-5|6-10|11-15|16-25}
{
  "Timestamp": "2023-11-15T09:15:20.250Z",
  "ObservedTimestamp": "2023-11-15T09:15:20.251Z", 
  "TraceId": "b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3",
  "SpanId": "c1d2e3f4a5b6c7d8",
  "TraceFlags": 1,
  "SeverityText": "ERROR",
  "SeverityNumber": 17,
  "Body": "Exception occurred while fetching user data",
  "EventName": "database.connection.failed",
  "Attributes": {
    "exception.type": "java.sql.SQLTransientConnectionException",
    "exception.message": "Connection timeout after 3000ms",
    "user.id": "alice",
    "db.operation": "select"
  },
  "Resource": {
    "Attributes": {
      "service.name": "user-service",
      "service.version": "1.5.2", 
      "deployment.environment": "staging"
    }
  }
}
```

---
layout: center
---

# Complex Data Types 📊

## Beyond Plain Strings

<v-clicks>

- **ArrayValue**: `["error", "timeout", "retry"]`
- **KeyValueList**: Nested maps within maps  
- **BytesValue**: Binary data in logs
- **AnyValue**: Dynamic type system

</v-clicks>

<div v-click class="mt-8 text-lg">
  <span class="text-orange-500">💡</span> Make your attributes as rich as your data!
</div>

<style>
h1 {
  background: linear-gradient(45deg, #f39800, #ff6b35);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>

---
layout: default
---

# Complex Values In Action 🚀

```json {1-4|5-10|11-16|17-22}
{
  "event": "user.login",
  "result": "success",
  "tags": ["important", "security", "audit"],
  "retry_attempts": [
    {"timestamp": "2023-11-15T09:15:15.100Z", "result": "timeout"},
    {"timestamp": "2023-11-15T09:15:18.200Z", "result": "success"}
  ],
  "user_permissions": {
    "read": true,
    "write": false,
    "admin": false
  },
  "session_metadata": {
    "ip": "192.168.1.100",
    "user_agent": "Mozilla/5.0...", 
    "geo": {
      "country": "US",
      "city": "San Francisco"
    }
  }
}
```

<div class="text-center mt-6 text-lg">
  <span class="text-green-400">✨</span> Rich, structured, meaningful data!
</div>

---
layout: center
class: text-center
---

# Semantic Conventions 🧭

## Making Logs Searchable & Comparable

<div class="text-2xl font-bold text-blue-400 mb-6">
  Same semantics = Better queries
</div>

<div class="grid grid-cols-2 gap-8 text-left max-w-4xl mx-auto">
  
<div>

### HTTP Requests 🌐
- `http.request.method`
- `http.response.status_code`  
- `url.full`
- `user_agent.original`

</div>

<div>

### Database Operations 🗄️
- `db.system` (postgresql, mysql)
- `db.statement`
- `db.operation.name`
- `server.address`

</div>

</div>

<div class="mt-8 text-lg text-green-400">
  <strong>One convention, infinite possibilities!</strong> 🚀
</div>

---
layout: center
---

# Events vs Log Records 🎭

<div class="text-2xl mb-8 text-center">
  <span class="text-blue-400">Event</span> or <span class="text-green-400">Log Record</span>?<br/>
  <span class="text-gray-400 text-lg">Plot twist: They're the same!</span>
</div>

<div class="grid grid-cols-2 gap-8 text-lg">

<div class="text-center">

### Event 🎉
```json
{
  "EventName": "user.login",
  "Body": "User logged in successfully"
}
```

**When something happens**

</div>

<div class="text-center">

### Log Record 📝
```json
{
  "SeverityText": "INFO", 
  "Body": "User logged in successfully"
}
```

**When you want to remember it**

</div>

</div>

<div class="text-center mt-8 text-xl text-yellow-400">
  Same data model, different perspectives! 🤝
</div>

---
layout: center
---

# What Gets Enabled? 🔥

## The Magic Happens Here

<v-clicks>

<div class="text-xl space-y-4">

🔍 **Cross-service tracing** - Follow requests across microservices

📊 **Rich dashboards** - Visual metrics that actually make sense  

🚨 **Smart alerting** - Context-aware notifications

🐛 **Faster debugging** - Jump from log → trace → problem

⚡ **Performance insights** - See the full picture, not just fragments

🤖 **AI/ML ready** - Structured data feeds better into models

</div>

</v-clicks>

<div v-click class="mt-8 text-center text-xl text-green-400">
  **This is why we do the hard work!** ✨
</div>

---
layout: center
class: text-center
---

# Wide Events 🌊

## Jeremy Morrell's Big Idea

<div class="text-xl mb-8 text-blue-400">
  Capture everything in ONE event instead of scattered logs
</div>

<div class="grid grid-cols-2 gap-8 max-w-4xl mx-auto text-left">

<div>

### Before: 😵‍💫
```text
[INFO] Request started
[DEBUG] Validating input  
[DEBUG] Calling database
[INFO] Response sent
[ERROR] Cache miss
```

</div>

<div>

### After: 🎯
```json
{
  "event": "http.request.complete",
  "duration_ms": 245,
  "cache_hit": false,
  "validation_errors": 0,
  "db_calls": 3,
  "response_size": 1024
}
```

</div>

</div>

<div class="mt-8 text-lg text-green-400">
  **One event to rule them all!** 💍
</div>

---
layout: center
---

# What Gets Enabled? 🔥

## The Magic Happens Here

<v-clicks>

<div class="text-xl space-y-4">

🔍 **Cross-service tracing** - Follow requests across microservices

📊 **Rich dashboards** - Visual metrics that actually make sense  

🚨 **Smart alerting** - Context-aware notifications

🐛 **Faster debugging** - Jump from log → trace → problem

⚡ **Performance insights** - See the full picture, not just fragments

🤖 **AI/ML ready** - Structured data feeds better into models

</div>

</v-clicks>

<div v-click class="mt-8 text-center text-xl text-green-400">
  **This is why we do the hard work!** ✨
</div>

---
layout: default
---

# User-Facing Logging APIs 🛠️

## Making It Easy for Developers

```go {1-5|6-10|11-15|16-20}
// The olog library - Zero-alloc, structured logging
import "go.opentelemetry.io/contrib/bridges/olog"

// Simple structured logging
olog.InfoContext(ctx, "User operation completed",
    "user.id", userID,
    "operation", "update_profile",
    "duration_ms", duration)

// Event-style logging  
olog.WarnContext(ctx, "Cache miss occurred",
    "cache.key", cacheKey,
    "fallback", "database_query")

// Rich context flows naturally
olog.ErrorContext(ctx, "Payment processing failed",
    "transaction.id", txnID,
    "amount", amount,
    "error", err.Error())
```

<div class="text-center mt-6 text-lg text-blue-400">
  **Clean code, structured output!** 🎯
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
