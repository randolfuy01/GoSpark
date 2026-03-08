# GoSpark

A native Go framework for Apache Spark — submit jobs, manipulate DataFrames, and interact with Spark clusters entirely from Go, using gRPC as the communication layer.

---

## Overview

GoSpark is born out of a simple frustration: there is no first-class Go experience for Apache Spark. Existing workarounds rely on shell invocations, REST wrappers, or JVM interop that feels foreign in a Go codebase.

GoSpark aims to fix that by providing:

- A **native Go DataFrame API** that mirrors the expressive power of the Spark DataFrame DSL
- A **gRPC-based connector** that communicates directly with a Spark-compatible endpoint
- A **SparkSession abstraction** as the single entry point for all cluster interactions
- A **built-in logger** with structured output, tunable verbosity, and integration hooks
- A clean **public API surface** designed for idiomatic Go usage

The result is a framework where submitting a Spark job feels as natural as making an HTTP request.

---

## Architecture
```
GoSpark Client (Go)
       │
       │  SparkSession.Submit(job)
       ▼
  Connector Layer
  (gRPC client, job serialization, auth)
       │
       │  gRPC / Spark Connect Protocol
       ▼
  Apache Spark Cluster
  (Spark Connect server or compatible endpoint)
       │
       │  Job execution
       ▼
  Response: JobResult
  (status, output schema, metrics, errors)
```

A `SparkSession` is the root object. It holds cluster configuration, manages the gRPC connection lifecycle, and exposes the DataFrame and job submission APIs. All jobs flow through the connector, which handles serialization, retries, and error mapping before results are returned to the caller.

---

## Core Components

### SparkSession

The entry point to GoSpark. Responsible for:

- Establishing and maintaining the gRPC connection to the Spark endpoint
- Providing access to the DataFrame API
- Managing session-level configuration (app name, cluster URL, auth tokens, etc.)
```go
session, err := gospark.NewSession(gospark.Config{
    Host:    "spark-connect.mycluster:15002",
    AppName: "my-go-app",
})
```

### Connector

The transport layer between GoSpark and Spark. Responsibilities include:

- Serializing job definitions into the wire format expected by the Spark endpoint
- Sending requests over gRPC and receiving responses
- Mapping Spark-side errors back to typed Go errors
- Supporting configurable timeouts, retries, and connection pooling

### DataFrame API

A Go-native representation of a Spark DataFrame. Operations are expressed as Go method calls, serialized into a logical plan, and executed remotely on the cluster. Targets an API feel familiar to users of `pyspark` or `spark-go`.
```go
df, err := session.Read().CSV("s3://my-bucket/data.csv")
result, err := df.Filter("age > 30").Select("name", "age").Collect()
```

### Logger

A structured, leveled logger built into the framework. Covers:

- Session lifecycle events (connect, disconnect, reconnect)
- Job submission and completion
- gRPC-level diagnostics (latency, retries, errors)
- Configurable output (stdout, file, custom writer) and log levels

### Job API

A lower-level API for submitting arbitrary Spark jobs when the DataFrame abstraction is not the right fit. Allows callers to construct job payloads directly and inspect raw `JobResult` responses containing execution metadata, output schemas, and cluster-reported metrics.

---

## Project Status

GoSpark is in early development. The following milestones are planned:

- [ ] gRPC connector with Spark Connect protocol support
- [ ] SparkSession with connection lifecycle management  
- [ ] Core DataFrame API (Read, Filter, Select, GroupBy, Collect)
- [ ] Structured logger
- [ ] Job submission API with JobResult types
- [ ] Error handling and typed error taxonomy
- [ ] Integration tests against a local Spark cluster
- [ ] Documentation and usage examples

---

## Design Goals

- **Idiomatic Go** — no JVM, no Python subprocess, no CGO
- **Type-safe** — job configurations and DataFrame schemas are strongly typed
- **Observable** — every operation is logged and traceable
- **Composable** — the connector, session, and DataFrame layers are independently usable

---

## Contributing

GoSpark is in active early design. If you have experience with Spark Connect, gRPC in Go, or distributed data systems, contributions and design feedback are welcome.

---

## License

MIT
