<h2 align="center">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/parseablehq/.github/main/images/logo-dark.png">
      <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/parseablehq/.github/main/images/logo.svg">
      <img alt="Parseable Logo" src="https://raw.githubusercontent.com/parseablehq/.github/main/images/logo.svg">
    </picture>
    <br>
    Parseable Golang SDK
</h2>

<div align="center">

[![Docker Pulls](https://img.shields.io/docker/pulls/parseable/parseable?logo=docker&label=Docker%20Pulls)](https://hub.docker.com/r/parseable/parseable)
[![Slack](https://img.shields.io/badge/slack-brightgreen.svg?logo=slack&label=Community&style=flat&color=%2373DC8C&)](https://launchpass.com/parseable)
[![Docs](https://img.shields.io/badge/stable%20docs-parseable.io%2Fdocs-brightgreen?style=flat&color=%2373DC8C&label=Docs)](https://www.parseable.io/docs)
[![Build](https://img.shields.io/github/checks-status/parseablehq/parseable/main?style=flat&color=%2373DC8C&label=Checks)](https://github.com/parseablehq/parseable/actions)

</div>

Parseable is a lightweight, cloud native log observability engine. Written in Rust, Parseable is built for high ingestion rates and low resource consumption. It is compatible with all major log agents and can be configured to collect logs from any source. Read more in [Parseable docs](https://www.parseable.io/docs).

## Parseable Golang SDK

This golang sdk can be embeeded in applications to send logs directly to parseable.

## Installation

The Parseable  golang sdk can be installed using go:

```bash
go get github.com/parseablehq/parseable-sdk-go/parseable
```
## Example Usage

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
	"time"

	"github.com/parseablehq/parseable-sdk-go/parseable"
)

// Sample log data in JSON format
var log = `[
	{
		"message": "tenants creation failed",
		"type": "error",
		"reason": "TenantsCreationFailed",
		"object": "baaz/tenants"
	}
]`

// Default duration for the query
var duration = "10h"

// QueryBuilder represents a query builder for fetching logs
type QueryBuilder struct {
	Query     string    `json:"query"`     // Query string
	StartTime time.Time `json:"startTime"` // Start time for the query
	EndTime   time.Time `json:"endTime"`   // End time for the query
}

func main() {
	// Set environment variables for Parseable SDK
	os.Setenv("PARSEABLE_URL", "http://localhost:9000")
	os.Setenv("PARSEABLE_USERNAME", "admin")
	os.Setenv("PARSEABLE_PASSWORD", "admin")

	// Define stream name
	stream := "mystream"

	// Create a new stream
	streamBuilder := parseable.NewStreamBuilder(
		stream,
		nil, // No initial data
		nil, // No metadata
		nil, // No tags
	)

	// Insert logs into the stream
	resp, err := streamBuilder.CreateStream()
	if err != nil && resp == 400 {
		fmt.Println(err)
		os.Exit(1)
	}

	// Labels associated with the log data
	labels := map[string]string{
		"type": "json",
	}

	// Tags associated with the log data
	tags := map[string]string{
		"app": "myapp",
	}

	// Create a new stream with log data, labels, and tags
	logStream := parseable.NewStreamBuilder(
		stream,
		[]byte(log), // Log data
		labels,      // Labels
		tags,        // Tags
	)

	// Insert logs into the stream
	_, err = logStream.InsertLogs()
	if err != nil {
		fmt.Println(err)
	}

	// Parse duration string to time.Duration
	duration, err := time.ParseDuration("-" + duration)
	if err != nil {
		fmt.Println(err)
	}

	// Get current time
	currentTime := time.Now()

	// Calculate start time by subtracting duration from current time
	startTime := currentTime.Add(duration)

	// Construct a new query with stream name and start/end times
	newQuery := QueryBuilder{
		Query:     fmt.Sprintf("select * from %s", stream),
		StartTime: startTime,
		EndTime:   currentTime,
	}

	// Marshal query to JSON format
	body, err := json.Marshal(newQuery)
	if err != nil {
		fmt.Println(err)
	}

	// Create a new query builder with the query body
	queryBuilder := parseable.NewQueryBuilder(string(body))

	// Query logs using the specified criteria
	resp, err := queryBuilder.QueryStream()
	if err != nil {
		fmt.Println(err)
	}

	// Print query response
	fmt.Println(resp)
}


```
