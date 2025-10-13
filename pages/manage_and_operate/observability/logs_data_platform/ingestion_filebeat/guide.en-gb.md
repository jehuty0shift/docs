---
title: Pushing logs with a forwarder - Fluent Bit (Linux)
updated: 2025-03-14
---

## Objective

[Fluent Bit](https://fluentbit.io/) is a lightweight and high-performance log processor able to collect data from many sources and send it to numerous outputs, including OpenSearch. This guide explains how to replace Filebeat with Fluent Bit in order to forward **system** and **Apache HTTP Server (apache2)** logs to Logs Data Platform by using a single OpenSearch output.

The tutorial focuses on Debian/Ubuntu-based systems but can be adapted to other Linux distributions.

## Requirements

To complete this tutorial, you need:

- An active [Logs Data Platform account](https://www.ovh.com/fr/order/express/#/new/express/resume?products=~%28~%28planCode~%27logs-account~productId~%27logs%29)).
- A stream created in Logs Data Platform with its [IAM bearer token ready](/pages/manage_and_operate/observability/logs_data_platform/security_tokens/guide.en-gb/#generating-tokens-with-iam). Fluent Bit will authenticate to OpenSearch with this token instead of username/password credentials.
- Root or sudo access on the machine that will run Fluent Bit.

## Instructions

### 1. Install the latest Fluent Bit release

At the time of writing, the current stable version of Fluent Bit is **3.0.4**. Use the official packages to ensure you receive the latest binaries and dependencies.

```bash
curl https://packages.fluentbit.io/fluentbit.key | sudo gpg --dearmor -o /usr/share/keyrings/fluentbit-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/fluentbit-keyring.gpg] https://packages.fluentbit.io/debian/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/fluent-bit.list

sudo apt update
sudo apt install fluent-bit
```

> [!primary]
> Adapt the repository URL if you are not using a Debian-based system. RPM packages and tar archives are available on the [Fluent Bit downloads page](https://fluentbit.io/download/).

### 2. Create the Fluent Bit configuration

We will collect two distinct sources:

- **System logs** from the local systemd journal using the `systemd` input.
- **Apache HTTP Server logs** (`access.log` and `error.log`) using the `tail` input.

Both streams will be enriched with common fields (service name, environment) and sent to OpenSearch through a single `opensearch` output configured to use your IAM bearer token.

Create `/etc/fluent-bit/fluent-bit.conf` with the following content:

```ini
[SERVICE]
    Flush        5
    Daemon       Off
    Log_Level    info
    Parsers_File parsers.conf

[INPUT]
    Name              systemd
    Tag               systemd.*
    Read_From_Tail    On

[INPUT]
    Name              tail
    Tag               apache.*
    Path              /var/log/apache2/access.log,/var/log/apache2/error.log
    Path_Key          log_file
    Refresh_Interval  10
    Skip_Long_Lines   On
    DB                /var/lib/fluent-bit/apache2.db
    Parser            apache2

[FILTER]
    Name          modify
    Match         systemd.*
    Add           service_type system
    Add           environment  production

[FILTER]
    Name          modify
    Match         apache.*
    Add           service_type apache2
    Add           environment  production

[OUTPUT]
    Name            opensearch
    Match           *
    Host            <your_cluster>.logs.ovh.com
    Port            9200
    Logstash_Format On
    Logstash_Prefix ${LD_STREAM_ID}-fluentbit
    Replace_Dots    On
    Header          Authorization Bearer ${IAM_BEARER_TOKEN}
    tls             On
    tls.verify      On
```

Replace the placeholders as follows:

- `<your_cluster>.logs.ovh.com`: the endpoint provided by Logs Data Platform for your OpenSearch cluster.
- `${LD_STREAM_ID}`: the identifier of your target stream. For example, `ld-1234567890`.
- `${IAM_BEARER_TOKEN}`: the IAM bearer token value generated for that stream (see the [Security Tokens guide](/pages/manage_and_operate/observability/logs_data_platform/security_tokens/guide.en-gb/#generating-tokens-with-iam)). You can export it as an environment variable prior to starting Fluent Bit, or replace the variable with the raw token string.

The output section relies on the OpenSearch plugin. The `Header` directive injects the IAM bearer token as a standard `Authorization: Bearer` header, which is the authentication method described in the Security Tokens guide. With `Logstash_Format On`, Fluent Bit appends the current date to the index name; `Logstash_Prefix` ensures each daily index starts with your stream identifier (for example, `ld-1234567890-fluentbit-2025.03.14`).

> [!warning]
> Make sure the token you generated has the `PUSH` permission on the stream, otherwise Fluent Bit will receive HTTP 403 errors.

### 3. Define the Apache log parser

Fluent Bit’s default configuration file includes common parsers, but we explicitly add one tailored for the Apache combined log format.

Create `/etc/fluent-bit/parsers.conf` with:

```ini
[PARSER]
    Name        apache2
    Format      regex
    Regex       ^(?<remote_host>[^ ]*) (?<remote_logname>[^ ]*) (?<remote_user>[^ ]*) \[(?<time_local>[^\]]*)\] "(?<request>[^"\\]*(?:\\.[^"\\]*)*)" (?<status>\d{3}) (?<body_bytes_sent>[^ ]*) "(?<http_referer>[^"\\]*(?:\\.[^"\\]*)*)" "(?<http_user_agent>[^"\\]*(?:\\.[^"\\]*)*)"
    Time_Key    time_local
    Time_Format %d/%b/%Y:%H:%M:%S %z
```

If your Apache logs use the default combined format, this parser will extract the usual fields. Adjust the regular expression if you customized the log format.

### 4. Provide the IAM bearer token to Fluent Bit

Export the token as an environment variable before starting the service:

```bash
export IAM_BEARER_TOKEN="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
export LD_STREAM_ID="ld-1234567890"
```

You can also set these variables permanently in `/etc/default/fluent-bit` (Debian/Ubuntu) so that they are available to the systemd unit.

### 5. Test the configuration interactively

Before enabling the service, run Fluent Bit in the foreground to confirm that records are sent correctly:

```bash
sudo -E fluent-bit -c /etc/fluent-bit/fluent-bit.conf
```

Keep the `sudo -E` flag so that the environment variables are preserved. Send a few Apache requests (for example with `curl http://localhost/`) and check that Fluent Bit prints successful delivery messages. Use `Ctrl+C` to stop the test run.

### 6. Enable Fluent Bit as a service

Once the configuration is validated, enable and start Fluent Bit with systemd:

```bash
sudo systemctl enable fluent-bit
sudo systemctl start fluent-bit
sudo systemctl status fluent-bit
```

The status command should report `active (running)`. Check `/var/log/syslog` or `journalctl -u fluent-bit` if you encounter issues.

## Troubleshooting tips

- **Authentication errors (403)**: Regenerate the IAM bearer token with the correct scope (`PUSH` permission) and make sure it matches the stream referenced by `Logstash_Prefix`.
- **Index naming**: Fluent Bit does not automatically create streams. Ensure the `Logstash_Prefix` matches the naming convention expected by Logs Data Platform. If you need multiple environments, create separate streams and adjust the prefix accordingly.
- **Apache permissions**: Fluent Bit must be able to read `/var/log/apache2/*.log`. If the service runs under the `fluent-bit` user, add it to the `adm` group on Debian/Ubuntu (`sudo usermod -a -G adm fluent-bit`).
- **Systemd input volume**: Add `Systemd_Filter` directives if you want to limit Fluent Bit to specific units (for example, `Systemd_Filter _SYSTEMD_UNIT=ssh.service`). Collecting the full journal can generate a high volume of events on busy hosts.

Fluent Bit now forwards both system and Apache logs to Logs Data Platform using the IAM bearer token-based authentication mechanism.
