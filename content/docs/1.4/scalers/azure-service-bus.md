+++
title = "Azure Service Bus"
maintainer = "Microsoft"
description = "Scale applications based on Azure Service Bus Queues or Topics."
availability = "v1.0+"
go_file = "azure_servicebus_scaler"
+++

### Trigger Specification

This specification describes the `azure-servicebus` trigger for Azure Service Bus Queue or Topic.

```yaml
triggers:
- type: azure-servicebus
  metadata:
    # Required: queueName OR topicName and subscriptionName
    queueName: functions-sbqueue
    # or
    topicName: functions-sbtopic
    subscriptionName: sbtopic-sub1
    # Optional, required when pod identity is used
    namespace: service-bus-namespace
    # Optional, can use TriggerAuthentication as well
    connection: SERVICEBUS_CONNECTIONSTRING_ENV_NAME # This must be a connection string for a queue itself, and not a namespace level (e.g. RootAccessPolicy) connection string [#215](https://github.com/kedacore/keda/issues/215)
    # Optional
    queueLength: "5" # Optional. Subscription length target for HPA. Default: 5 messages
```

**Parameter list:**

- `queueName` - Name of the Azure Service Bus queue to scale on. (Optional)
- `topicName` - Name of the Azure Service Bus topic to scale on. (Optional)
- `subscriptionName` - Name of the Azure Service Bus queue to scale on. (Optional*, Required when `topicName` is specified)
- `namespace` - Name of the Azure Service Bus namespace that contains your queue or topic. (Optional*, Required when pod identity is used)
- `connection` - Name of the environment variable your deployment uses to get the connection string of the Azure Service Bus namespace. (Optional)
- `queueLength` - Amount of active messages in your Azure Service Bus queue or topic to scale on.

> 💡 **NOTE:** Service Bus Shared Access Policy needs to be of type `Manage`. Manage access is required for KEDA to be able to get metrics from Service Bus.

### Authentication Parameters

You can authenticate by using pod identity or connection string authentication.

**Connection String Authentication:**

- `connection` - Connection string for Azure Service Bus Namespace.

### Example

Here is an example of how to use managed identity:

```yaml
apiVersion: keda.k8s.io/v1alpha1
kind: TriggerAuthentication
metadata:
  name: azure-servicebus-auth
spec:
  podIdentity:
    provider: azure
---
apiVersion: keda.k8s.io/v1alpha1
kind: ScaledObject
metadata:
  name: azure-servicebus-queue-scaledobject
  namespace: default
spec:
  scaleTargetRef:
    deploymentName: azure-servicebus-queue-function
  triggers:
  - type: azure-servicebus
    metadata:
      # Required: queueName OR topicName and subscriptionName
      queueName: functions-sbqueue
      # or
      topicName: functions-sbtopic
      subscriptionName: sbtopic-sub1
      # Required: Define what Azure Service Bus to authenticate to with Managed Identity
      namespace: service-bus-namespace
      # Optional
      queueLength: "5" # default 5
    authenticationRef:
        name: azure-servicebus-auth # authenticationRef would need either podIdentity or define a connection parameter
```
