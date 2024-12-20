## Introduction

If you have a multi-goroutine program where each goroutine uses Lumberjack for log rotation, the official repo will experience a memory leak for sure. A fix was proposed years ago, but the pull request was never accepted. 

As a result, I'm creating my own repo to resolve this issue more quickly.

## Usage

You can retrieve a rotating log handler like this, using the power of the Logrus library:

```go
import (
    lumberjack "github.com/CaledoniaProject/gopkg-lumberjack"
)

func GetTTYFormatter() *logrus.TextFormatter {
	return &logrus.TextFormatter{
		TimestampFormat: "2006-01-02 15:04:05",
		DisableColors:   false,
		ForceColors:     true,
		FullTimestamp:   true,
		CallerPrettyfier: func(f *runtime.Frame) (string, string) {
			return "", fmt.Sprintf("[%s:%d]", path.Base(f.File), f.Line)
		},
	}
}

func GetRotatingFileLogger(filename string) (*logrus.Logger, *lumberjack.Logger) {
	lumberjack_logger := &lumberjack.Logger{
		Filename:   filename,
		MaxSize:    200,
		MaxBackups: 20,
		MaxAge:     7,
		Compress:   true,
	}
	logger := &logrus.Logger{
		Out:          lumberjack_logger,
		Formatter:    GetTTYFormatter(),
		ReportCaller: true,
		Level:        logrus.InfoLevel,
	}

	return logger, lumberjack_logger
}
```
