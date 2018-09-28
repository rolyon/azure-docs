---
title: include file
description: include file
services: virtual-machines
author: changov
ms.service: virtual-machines
ms.topic: include
ms.date: 09/10/2018
ms.author: vashan, rajraj, changov
ms.custom: include file
---


Azure Compute requests may be throttled at a subscription and on a per-region basis to help with the overall performance of the service. We ensure all the calls to the Azure Compute Resource Provider (CRP) that manages resources under Microsoft.Compute namespace don't exceed the maximum allowed API request rate. This document describes API throttling, details on how to troubleshoot throttling issues, and best practices to avoid being throttled.  

## Throttling by Azure Resource Manager vs Resource Providers  

As the front door to Azure, Azure Resource Manager does the authentication and first-order validation and throttling of all incoming API requests. Azure Resource Manager call rate limits and related diagnostic response HTTP headers are described [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-request-limits).
 
When an Azure API client gets a throttling error, the HTTP status is 429 Too Many Requests. To understand if the request throttling is done by Azure Resource Manager or an underlying resource provider like CRP, inspect the `x-ms-ratelimit-remaining-subscription-reads` for GET requests and `x-ms-ratelimit-remaining-subscription-writes` response headers for non-GET requests. If the remaining call count is approaching 0, the subscription’s general call limit defined by Azure Resource Manager has been reached. Activities by all subscription clients are counted together. Otherwise, the throttling is coming from the target resource provider (the one addressed by the `/providers/<RP>` segment of the request URL). 

## Call rate informational response headers 

| Header                            | Value format                           | Example                               | Description                                                                                                                                                                                               |
|-----------------------------------|----------------------------------------|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| x-ms-ratelimit-remaining-resource |```<source RP>/<policy or bucket>;<count>```| Microsoft.Compute/HighCostGet3Min;159 | Remaining API call count for the throttling policy covering the resource bucket or operation group including the target of this request                                                                   |
| x-ms-request-charge               | ```<count>   ```                             | 1                                     | The number of call counts “charged” for this HTTP request toward the applicable policy’s limit. This is most typically 1. Batch requests, such as for scaling a virtual machine scale set, can charge multiple counts. |


Note that an API request can be subjected to multiple throttling policies. There will be a separate `x-ms-ratelimit-remaining-resource` header for each policy. 

Here is a sample response to delete a VM in a virtual machine scale set request.

```
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet3Min;107 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet30Min;587 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VMScaleSetBatchedVMRequests5Min;3704 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VmssQueuedVMOperations;4720 
```

##Throttling error details

The 429 HTTP status is commonly used to reject a request because a call rate limit is reached. A typical throttling error response from Compute Resource Provider will look like the example below (only relevant headers are shown):

```
HTTP/1.1 429 Too Many Requests
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet3Min;46
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet30Min;0
Retry-After: 1200
Content-Type: application/json; charset=utf-8
{
  "code": "OperationNotAllowed",
  "message": "The server rejected the request because too many requests have been received for this subscription.",
  "details": [
    {
      "code": "TooManyRequests",
      "target": "HighCostGet30Min",
      "message": "{\"operationGroup\":\"HighCostGet30Min\",\"startTime\":\"2018-06-29T19:54:21.0914017+00:00\",\"endTime\":\"2018-06-29T20:14:21.0914017+00:00\",\"allowedRequestCount\":800,\"measuredRequestCount\":1238}"
    }
  ]
}

```

The policy with remaining call count of 0 is the one due to which the throttling error is returned. In this case that is `HighCostGet30Min`. The overall format of the response body is the general Azure Resource Manager API error format (conformant with OData). The main error code, `OperationNotAllowed`, is the one Compute Resource Provider uses to report throttling errors (among other types of client errors). 

As illustrated above, every throttling error includes the `Retry-After` header, which provides the minimum number of seconds the client should wait before retrying the request. 

## Best practices 

- Do not retry Azure service API errors unconditionally. A common occurrence is for client code to get into a rapid retry loop when encountering an error that’s not retry-able. Retries will eventually exhaust the allowed call limit for the target operation’s group and impact other clients of the subscription. 
- In high-volume API automation cases, consider implementing proactive client-side self-throttling when the available call count for a target operation group drops below some low threshold. 
- When tracking async operations, respect the Retry-After header hints. 
- If the client code needs information about a particular Virtual Machine, a query that VM directly instead of listing all VMs in the containing resource group or the entire subscription and then picking the needed VM on the client side. 
- If client code needs VMs, disks and snapshots from a specific Azure location, use location-based form of the query instead of querying all subscription VMs and then filtering by location on the client side: `GET /subscriptions/<subId>/providers/Microsoft.Compute/locations/<location>/virtualMachines?api-version=2017-03-30` and `/subscriptions/<subId>/providers/Microsoft.Compute/virtualMachines`  query to Compute Resource Provider regional endpoints. 
•	When creating or updating API resources in particular, VMs and virtual machine scale sets, it is far more efficient to track the returned async operation to completion than do polling on the resource URL itself (based on the `provisioningState`).

## Next steps

For more information about retry guidance for other services in Azure, see [Retry guidance for specific services](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific)