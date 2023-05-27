---
id: collect-logs
title: Collect Logs
sidebar_label: Collect Logs
description: Learn how to collect logs using the Sumo Logic OpenTelemetry Collector.
---

The Sumo Logic Distribution for OpenTelemetry Collector provides various receivers for log collection. This document describes the receivers most commonly used for logs:


* [Filelog Receiver](#filelog-receiver)
* [Windows Log Event Receiver](#windows-log-event-receiver)
* [Syslog Receiver](#syslog-receiver)

You can find the full list of receivers on our [Sumologic OpenTelemetry Collector page](https://github.com/SumoLogic/sumologic-otel-collector/tree/main#components).


## Filelog Receiver

Following configuration demonstrates : 
 1. Collect   : How to collect logs stored in /var/log/syslog and /var/log/audit/audit.log
 2. Transform : Set \_sourceCatetgory field to "application_logs_prod"
 3. Export    : Send data to authenticated Sumologic organization.

```yaml
receivers:
  filelog/custom_files:                                         
    include_file_path: true
    include:                                         
      - /var/log/syslog
      - /var/log/audit/audit.log
     storage: file_storage
     
processors:
  resource/processor1:
    attributes:                                       
      - key: _sourceCategory
        value: application_logs_prod
        action: insert
service:
  pipelines:
    logs/pipeline1:
      receivers:
        - filelog/1
      processors:
        - resource/1
        - batch
        - memory_limiter
      exporters:
        - sumologic
```


### Configuration details

 1. Create a file in folder /etc/otelcol-sumo/conf.d  with name for your choice, sample_app_configuration.yaml.
 2. Paste the above content.
 3. Restart collector with following command:
      - Linux : systemctl restart otelcol-sumo
      - Windows : Restart-Service -Name OtelcolSumo

* __receivers__: 
  * `filelog/custom_files:` Unique name defined for the filelog receiver. You can add multiple file receivers as filelog/2 , filelog/prod etc.
  * `include:` Lets the Filelog Receiver know where the log file is placed. Make sure that the collector has permissions to access the files; otherwise, it will not be collected.
  * `include_file_path_resolved: true` Adds a `log.file.path_resolved` attribute to the logs that contain the whole path of the file.
  * `storage: file_storage:` Prevents the receiver from reading the same data over and over again on each otelcol restart. Extension defined by default in sumologi.yaml
 
* __processors__: 
  * `resource/1` Name of the processor that uses `attributes` to add \_sourceCategory field with value `application_logs_prod`
 
* __exporters__: 
  *  `sumologic` Sends data to registered Sumologic Organization.  Auto configured in sumologic.yaml during installation using script.
 
* __service__: 
  *  `logs/pipeline1:`  Pipeline glues together what data to collect(receiver), how to process the data(processors) and where to send logs(exporter).



The remaining processors in pipeline are from `sumologic.yaml` file and should be applied for better performance of the collector.

:::note
The receiver (`filelog/custom_files`) ,  (`resource/processor1`) and  pipeline (`logs/pipeline1:`) names should be unique across all configuration files to avoid conflicts and unexpected behavior.
:::


For more details, see the [Filelog Receiver documentation][filelogreceiver_readme].

## Windows Log Event Receiver

Windows Log Event Receiver reads and parses logs from windows event log API to collect local events as you see on Windows Event Viewer.

Following configuration demonstrates:
 1. Collect : Collect Application, Security and System channels 
 2. Transform : Set \_sourceCatetgory field to "windows_event_log_prod" 
 3. Export : Send data to authenticated Sumologic organization
 
 
```yaml
receivers:
  windowseventlog/application/localhost:
    channel: Application
  windowseventlog/security/localhost:
    channel: Security
  windowseventlog/system/localhost:
    channel: System

processors:
  resource/windows_resource_attributes/localhost:
    attributes:
      - key: _sourceCategory
        value: windows_event_log_prod
        action: insert
service:
  pipelines:
    logs/windows/localhost:
      receivers:
        - windowseventlog/application/localhost
        - windowseventlog/system/localhost
        - windowseventlog/security/localhost
      processors:
        - resource/windows_resource_attributes/localhost
        - batch
        - memory_limiter
      exporters:
        - sumologic
```
### Configuration details

 1. Create a file in folder /etc/otelcol-sumo/conf.d  with name sample_windows.yaml.
 2. Paste the above content.
 3. Restart collector with following command:
      - Restart-Service -Name OtelcolSumo


* __receivers__: 
  * `windowseventlog/application/localhost:` Collect logs from application channel.
  * `windowseventlog/security/localhost:` Collect logs from security channel..
  * `windowseventlog/system/localhost:` Collect logs from system channel.
 
* __processors__: 
  * `  resource/windows_resource_attributes/localhost:` Name of the processor that uses `attributes` to add \_sourceCategory field with value `windows_event_log_prod`
 
* __exporters__: 
  *  `sumologic` Sends data to registered Sumologic Organization.  Auto configured in sumologic.yaml during installation using script.
 
* __service__: 
  *  `logs/windows/localhost:`  Pipeline glues together what data to collect(receiver), how to process the data(processors) and where to send logs(exporter). 


For more details, see the [Windows Event Log receiver][windowseventlogreceiver].


## Syslog Receiver

The OpenTelemetry Collector offers two approaches for Syslog processing:

* [Syslog Receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/syslogreceiver)
* [TCPlog](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/tcplogreceiver) / [UDPlog Receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/udplogreceiver)

Following configuration demonstrates:
 1. Collect : Collect syslog sent using udp protocol to 127.0.0.1 on port 514
 2. Transform : Set \_sourceCatetgory field to "syslog_event_log_prod" 
 3. Export : Send data to authenticated Sumologic organization

```yaml

receivers:
  syslog:
     udp:
    listen_address: "127.0.0.1:514"
   

processors:
  resource/syslog:
    attributes:
      - key: _sourceCategory
        value: syslog_event_log_prod
        action: insert

 
service:
  pipelines:
    logs/syslog:
      receivers: 
        - syslog
      processors: 
        - resource/syslog:
      exporters: 
              sumologic
  ```

### Configuration details

 1. Create a file in folder /etc/otelcol-sumo/conf.d  with name for your choice, sample_syslog_configuration.yaml.
 2. Paste the above content.
 3. Restart collector with following command:
      - Linux : systemctl restart otelcol-sumo


The following table shows the comparison of source specific configurations between the Installed Collector and OpenTelemetry Collector.

| Feature/Capability | OpenTelemetry Syslog Receiver | TCPlog/UDPlog Receiver and Sumo Logic Syslog Processor |
|:--------------------|:----------------|:---------------------------------|
| Accepts logs        | `RFC3164` and `RFC5424` formats | Any format |
| Field Parsing       | Collector side | Not on collector side |
| Protocol Verification | Strict parsing; logs sent to the wrong endpoint will not be parsed | No protocol verification; all formats are treated the same |
| Recommendation      | Sending logs using a certain RFC protocol | Compatibility with the current Installed Collector behavior is needed |


For more details, see the [Syslog receiver](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/syslogreceiver)


## Parsing JSON logs with File Receiver

Filelog Receiver with [json_parser][json_parser] operator can be used for parsing JSON logs. The [json_parser][json_parser] operator parses the string-type field selected by `parse_from` as JSON (by default, `parse_from` is set to `$body` which indicates the whole log record).

For example, when logs has following form in the file:

```json
{
  "message": "{\"key\": \"val\"}"
}
```

Then configuration to extract JSON which is represented as string (`{\"key\": \"val\"}`) has following form:

```yaml
receivers:
  filelog/custom_files:
    include:
    - /var/log/myservice/*.log
  
    include_file_path_resolved: true
    storage: file_storage
    operators:
      # this parses log line as JSON for all files ins
      - type: json_parser
        parse_to: body
   
      # this parses string under 'message' key as JSON for all files
      - type: json_parser
        parse_from: body.message
        parse_to: body.message
    
processors:
  groupbyattrs/file path resolved:
    keys:
      - log.file.path_resolved
service:
  pipelines:
    logs/custom_files:
      receivers:
      - filelog/custom_files
      processors:
      - memory_limiter
      - groupbyattrs/file path resolved
      - resourcedetection/system
      - batch
      exporters:
      - sumologic
```




:::tip Additional information
* See [OpenTelemetry documentation][windowseventlogreceiver] for more information about Windows Event Log receiver.
* See [Additional Configurations Reference](/docs/send-data/opentelemetry-collector/data-source-configurations/additional-configurations-reference/) for more details about OpenTelemetry configuration.
:::

[json_parser]: https://github.com/open-telemetry/opentelemetry-log-collection/blob/main/docs/operators/json_parser.md
[filelogreceiver_readme]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/filelogreceiver
[filestorageextension_docs]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/extension/storage/filestorage
[groupbyattrprocessor]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/processor/groupbyattrsprocessor#group-by-attributes-processor
[windowseventlogreceiver]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/receiver/windowseventlogreceiver/README.md
