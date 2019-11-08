# ⚡ gcloudzap [![GoDoc](https://godoc.org/github.com/jonstaryuk/gcloudzap?status.svg)](https://godoc.org/github.com/danielweidman/gcloudzap)




**(experimental)**

This package provides a zap logger that forwards entries to the Google Stackdriver Logging service as structured payloads.

This version has two main changes from the version fully created by jonstaryuk:
- It lets one override the "stack" and "caller" fields to be sent to StackDriver with custom inputs. 
- It has versions of NewProduction() and NewDevelopment() [NewProductionWithOptions() and NewDevelopmentWithOptions()] that let you create a new client that will have GCP credentials other than those based on the APPLICATION_DEFAULT_PATH environment variable. Before you could do this with the New() function, but this way may be easier.
If you don't need these feature, then use the main project: github.com/jonstaryuk/gcloudzap

## Quickstart

Outside of Google Compute Engine, just add the environment variable `GOOGLE_APPLICATION_CREDENTIALS` with a path to a JSON credential file. For more info on this approach, see the [docs](https://developers.google.com/identity/protocols/application-default-credentials#howtheywork).

#### Option 1: Less configuration

```go
import "github.com/danielweidman/gcloudzap"

log, err := gcloudzap.NewDevelopment("your-project-id", "your-log-id")
if err != nil {
    panic(err)
}

log.Sugar().
    With("simple", true).
    With("foo", "bar").
    Info("This will get written to both stderr and Stackdriver Logging.")
```

#### Option 2: More flexibility

```go
import (
    "go.uber.org/zap"
    "cloud.google.com/go/logging"
    "github.com/danielweidman/gcloudzap"
)

// Configure the pieces
client, err := logging.NewClient(...)
cfg := zap.Config{...}

// Create a logger
log, err := gcloudzap.New(cfg, client, "your-log-id", zap.Option(), ...)
```

#### Option 3: Most flexibility

```go
import (
    "go.uber.org/zap/zapcore"
    "cloud.google.com/go/logging"
    "github.com/danielweidman/gcloudzap"
)

// Configure the pieces
client, err := logging.NewClient(...)
baseCore := zapcore.NewCore(...)

// Create a core
core := gcloudzap.Tee(baseCore, client, "your-log-id")
```
