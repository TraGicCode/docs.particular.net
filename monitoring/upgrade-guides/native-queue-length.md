---
title: Upgrade from Estimated to Native Queue Length Metric
summary: Instructions on how to upgrade an NServiceBus system from estimated to native queue length metric 
isUpgradeGuide: true
reviewed: 2018-05-22
---

## Overview

The experimental Queue Length metrics feature based on the estimation algorithm has been replaced by the Native Queue Length metric. This document describes the migration path for systems using the old metrics.

### Upgrading monitoring instance

Service Control should be upgraded to version 1.48.0+. Starting the upgrade process with an old monitoring instance (and not the endpoints) prevents old version of monitoring instance from receiving new, unsupported metric reports. In addition new monitoring instances produce `WARN` log entries of the form:

> Legacy queue length report received from {InstanceName} instance of {EndpointName}.

that enable identification of the endpoints running `NServiceBus.Metrics.ServiceControl` packages that do not support Native Queue Length metric.

### Upgrading ServicePulse

ServicePulse should be upgraded to the newest version. This step ensures compatibility of the updated version of the monitoring instance and ServicePulse.

### Upgrading endpoints

All endpoints sending monitoring data need to be upgraded to the newest version of `NServiceBus.Metrics.ServiceControl`. The exact version depends on the `NServiceBus` version the endpoint is running. The table below shows the necessary version ranges.

|NServiceBus|NServiceBus.Metrics.ServiceControl|
|--|--|
| 5.*       | [1.3.0, 2.0)|
| 6.*       | [2.1.0, 3.0)|
| 7.*       | [3.0,)|

WARN: When running the MSMQ transport it is necessary to install the latest [`NServiceBus.Metrics.ServiceControl.Msmq`](/monitoring/metrics/msmq-queue-length.md) package. 
