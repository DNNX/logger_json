# LoggerJSON

[![Deps Status](https://beta.hexfaktor.org/badge/all/github/Nebo15/logger_json.svg)](https://beta.hexfaktor.org/github/Nebo15/logger_json) [![Hex.pm Downloads](https://img.shields.io/hexpm/dw/logger_json.svg?maxAge=3600)](https://hex.pm/packages/logger_json) [![Latest Version](https://img.shields.io/hexpm/v/logger_json.svg?maxAge=3600)](https://hex.pm/packages/logger_json) [![License](https://img.shields.io/hexpm/l/logger_json.svg?maxAge=3600)](https://hex.pm/packages/logger_json) [![Build Status](https://travis-ci.org/Nebo15/logger_json.svg?branch=master)](https://travis-ci.org/Nebo15/logger_json) [![Coverage Status](https://coveralls.io/repos/github/Nebo15/logger_json/badge.svg?branch=master)](https://coveralls.io/github/Nebo15/logger_json?branch=master) [![Ebert](https://ebertapp.io/github/Nebo15/logger_json.svg)](https://ebertapp.io/github/Nebo15/logger_json)

JSON console back-end for Elixir Logger.

It can be used as drop-in replacement for default `:console` Logger back-end in cases where you use
use Google Cloud Logger or other JSON-based log collectors.

## Motivation

[We](https://github.com/Nebo15) deploy our applications as dockerized containers in Google Container Engine (Kubernetes cluster), in this case all your logs will go to `stdout` and log solution on top of Kubernetes should collect and persist it elsewhere.

In GKE it is persisted in Google Cloud Logger, but traditional single Logger output may contain newlines for a single log line, and GCL counts each new line as separate log entry, this making it hard to search over it.

This backed makes sure that there is only one line per log record and adds additional integration niceness, like [LogLine](https://cloud.google.com/logging/docs/reference/v1beta3/rest/v1beta3/LogLine) format support.

After adding this back-end you may also be interested in [redirecting otp and sasl reports to Logger](https://hexdocs.pm/logger/Logger.html#error-logger-configuration) (see "Error Logger configuration" section).

## Log Format

By default, output JSON is compatible with
[Google Cloud Logger format](https://cloud.google.com/logging/docs/reference/v1beta3/rest/v1beta3/LogLine) with
additional properties in `serviceLocation` and `metadata` objects:

  ```json
  {
     "timestamp":"2017-11-19T18:13:01.111Z",
     "sourceLocation":{
        "line":28,
        "function":"Elixir.LoggerJSON.Ecto.log/1",
        "file":"/Users/andrew/Projects/logger_json/lib/logger_json/ecto.ex"
     },
     "severity":"DEBUG",
     "resource":{
        "type":"elixir-application",
        "labels":{
           "version":"1.0.0",
           "service":"logger_json"
        }
     },
     "jsonPayload":{
        "metadata":{
           "queue_time":0.1,
           "query_time":2.1,
           "duration":2.7,
           "decode_time":0.5,
           "connection_pid":null,
           "application":"logger_json"
        },
        "message":"done"
     }
  }
  ```

You can change this structure by implementing `LoggerJSON.Formatter` behaviour and passing module
name to `:formatter` config option. Example module can be found in `LoggerJSON.Formatters.GoogleCloudLogger`.

  ```elixir
  config :logger_json, :backend,
    formatter: MyFormatterImplementation
  ```

## Installation

It's [available on Hex](https://hex.pm/packages/logger_json), the package can be installed as:

  1. Add `:logger_json` and `:jason` to your list of dependencies in `mix.exs`:

    def deps do
      [{:logger_json, "~> 1.0.1"}]
    end

  2. Ensure `logger_json` and `:jason` is started before your application:

    def application do
      [extra_applications: [:jason, :logger_json]]
    end

  3. Set configuration in your `config/config.exs`:

    config :logger_json, :backend,
      metadata: :all

  Some integrations (for eg. Plug) uses `metadata` to log request
  and response parameters. You can reduce log size by replacing `:all`
  (which means log all) with a list of the ones that you actually need.

  4. Replace default Logger `:console` back-end with `LoggerJSON`:

    config :logger,
      backends: [LoggerJSON]

  5. Optionally. Log requests and responses by replacing a `Plug.Logger` in your endpoint with a:

    plug LoggerJSON.Plug

  6. Optionally. Log Ecto queries via Plug:

    config :my_app, MyApp.Repo,
      adapter: Ecto.Adapters.Postgres,
      ...
      loggers: [{LoggerJSON.Ecto, :log, [:info]}]

## Dynamic configuration

For dynamically configuring the endpoint, such as loading data
from environment variables or configuration files, LoggerJSON provides
an `:on_init` option that allows developers to set a module, function
and list of arguments that is invoked when the endpoint starts.

    config :logger_json, :backend,
      on_init: {YourApp.Logger, :load_from_system_env, []}

## Encoders support

You can replace default Jason encoder with other module that supports `encode_to_iodata!/1` function.

On top of that `GoogleCloudLogger` formatter builds fragments at compile-time for a performance optimization, so you would either need to use your own formatter or make sure it's fragment API is compatible.

## Documentation

The docs can be found at [https://hexdocs.pm/logger_json](https://hexdocs.pm/logger_json)

## Thanks

Many source code has been taken from original Elixir Logger `:console` back-end source code, so I want to thank all it's authors and contributors.

Part of `LoggerJSON.Plug` module have origins from `plug_logger_json` by @bleacherreport,
originally licensed under Apache License 2.0. Part of `LoggerJSON.PlugTest` are from Elixir's Plug licensed under Apache 2.
