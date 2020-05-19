# Sensu PagerDuty Handler

[![Bonsai Asset Badge](https://img.shields.io/badge/Sensu%20PagerDuty%20Handler-Download%20Me-brightgreen.svg?colorB=89C967&logo=sensu)](https://bonsai.sensu.io/assets/sensu/sensu-pagerduty-handler)
![Go Test](https://github.com/sensu/sensu-pagerduty-handler/workflows/Go%20Test/badge.svg)
![goreleaser](https://github.com/sensu/sensu-pagerduty-handler/workflows/goreleaser/badge.svg)

# Sensu PagerDuty Handler

## Table of Contents
- [Overview](#overview)
- [Usage examples](#usage-examples)
  - [Help](#help)
  - [Deduplication Key](#deduplication-key)
  - [PagerDuty Severity Mapping](#pagerduty-severity-mapping)
- [Configuration](#configuration)
  - [Asset registration](#asset-registration)
  - [Handler definition](#handler-definition)
  - [Environment Variables](#environment-variables)
  - [Argument Annotations](#argument-annotations)
  - [Proxy support](#proxy-support)
- [Installation from source](#installation-from-source)
- [Contributing](#contributing)

## Overview

The Sensu PagerDuty Handler is a [Sensu Event Handler][3] which manages
[PagerDuty][2] incidents, for alerting operators. With this handler,
[Sensu][1] can trigger and resolve PagerDuty incidents.

## Usage Examples

### Help
```
The Sensu Go PagerDuty handler for incident management

Usage:
  sensu-pagerduty-handler [flags]
  sensu-pagerduty-handler [command]

Available Commands:
  help        Help about any command
  version     Print the version number of this plugin

Flags:
  -t, --token string                The PagerDuty V2 API authentication token, can be set with PAGERDUTY_TOKEN
  -k, --dedup-key-template string   The PagerDuty V2 API deduplication key template, can be set with PAGERDUTY_DEDUP_KEY_TEMPLATE (default "{{.Entity.Name}}-{{.Check.Name}}")
  -S, --summary-template string     The template for the alert summary, can be set with PAGERDUTY_SUMMARY_TEMPLATE
  -s, --status-map string           The status map used to translate a Sensu check status to a PagerDuty severity, can be set with PAGERDUTY_STATUS_MAP (default "{{.Entity.Name}}/{{.Check.Name}} : {{.Check.Output}}")
  -h, --help                        help for sensu-pagerduty-handler

Use "sensu-pagerduty-handler [command] --help" for more information about a command.

```

### Deduplication Key

The deduplication key is determined via the `--dedup-key-template` argument.  It
is a Golang template containing the event values and defaults to
`{{.Entity.Name}}-{{.Check.Name}}`.

### PagerDuty Severity Mapping

Optionally you can provide mapping information between the Sensu check status
and the PagerDuty incident severity. To provide the mapping you need to use the
`--status-map` command line option or the `PAGERDUTY_STATUS_MAP` environment
variable.  The option accepts a JSON document containing the mapping
information. Here's an example of the JSON document:

```json
{
    "info": [
        0,
        1
    ],
    "warning": [
        2
    ],
    "critical:": [
        3
    ],
    "error": [
        4,
        5,
        6,
        7,
        8,
        9,
        10
    ]
}
```

The valid [PagerDuty alert severity levels][5] are the following:
* `info`
* `warning`
* `critical`
* `error`

## Configuration
### Sensu Go
#### Asset registration

[Sensu Assets][6] are the best way to make use of this plugin. If you're not
using an asset, please consider doing so! If you're using sensuctl 5.13 with
Sensu Backend 5.13 or later, you can use the following command to add the asset:

```
sensuctl asset add sensu/sensu-pagerduty-handler
```

If you're using an earlier version of sensuctl, you can find the asset on the
[Bonsai Asset Index][7].

#### Handler definition

```yml
---
type: Handler
api_version: core/v2
metadata:
  name: pagerduty
  namespace: default
spec:
  type: pipe
  command: sensu-pagerduty-handler
  timeout: 10
  runtime_assets:
  - sensu/sensu-pagerduty-handler
  filters:
  - is_incident
  secrets:
  - name: PAGERDUTY_TOKEN
    secret: pagerduty_authtoken
```

#### Environment Variables

Most arguments for this handler are available to be set via environment
variables.  However, any arguments specified directly on the command line
override the corresponding environment variable.

|Argument            |Environment Variable        |
|--------------------|----------------------------|
|--token             |PAGERDUTY_TOKEN             |
|--summary-template  |PAGERDUTY_SUMMARY_TEMPLATE  |
|--dedup-key-template|PAGERDUTY_DEDUP_KEY_TEMPLATE|
|--status-map        |PAGERDUTY_STATUS_MAP        |

**Security Note:** Care should be taken to not expose the auth token for this
handler by specifying it on the command line or by directly setting the
environment variable in the handler definition.  It is suggested to make use of
[secrets management][8] to surface it as an environment variable.  The handler
definition above references it as a secret.  Below is an example secrets
definition that make use of the built-in [env secrets provider][9].

```yml
---
type: Secret
api_version: secrets/v1
metadata:
  name: pagerduty_token
spec:
  provider: env
  id: PAGERDUTY_TOKEN
```

#### Argument Annotations

All arguments for this handler are tunable on a per entity or check basis based
on annotations.  The annotations keyspace for this handler is
`sensu.io/plugins/sensu-opsgenie-handler/config`.

###### Examples

To change the token argument for a particular check, for that checks's metadata
add the following:

```yml
type: CheckConfig
api_version: core/v2
metadata:
  annotations:
    sensu.io/plugins/sensu-pagerduty-handler/config/token: abcde12345fabcd67890efabc12345de
[...]
```

#### Proxy Support

This handler supports the use of the environment variables HTTP_PROXY,
HTTPS_PROXY, and NO_PROXY (or the lowercase versions thereof). HTTPS_PROXY takes
precedence over HTTP_PROXY for https requests.  The environment values may be
either a complete URL or a "host[:port]", in which case the "http" scheme is
assumed.

## Installation from source

Download the latest version of the sensu-pagerduty-handler from [releases][4],
or create an executable script from this source.

From the local path of the sensu-pagerduty-handler repository:
```
go build -o /usr/local/bin/sensu-pagerduty-handler
```

## Contributing

See https://github.com/sensu/sensu-go/blob/master/CONTRIBUTING.md

[1]: https://github.com/sensu/sensu-go
[2]: https://www.pagerduty.com/
[3]: https://docs.sensu.io/sensu-go/5.0/reference/handlers/#how-do-sensu-handlers-work
[4]: https://github.com/sensu/sensu-pagerduty-handler/releases
[5]: https://support.pagerduty.com/docs/dynamic-notifications#section-eventalert-severity-levels
[6]: https://docs.sensu.io/sensu-go/latest/reference/assets/
[7]: https://bonsai.sensu.io/sensu/sensu-pagerduty-handler
[8]: https://docs.sensu.io/sensu-go/latest/guides/secrets-management/
[9]: https://docs.sensu.io/sensu-go/latest/guides/secrets-management/#use-env-for-secrets-management
