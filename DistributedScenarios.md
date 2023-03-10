## **Cosmos DB SDK Distributed Tracing**
Main work items:


- Create Activity object:

    SDK will generate Activity object with schema attributes everytime a network call is made. This activity object generates TraceId and SpanId.

- Context propagation

    Public SDK will send request with headers Traceparent consisting of TraceId and SpanID. This way TraceId can act as parent node in the distribution tree.
    This activity needs to be generated at network call level. (currently sdk creates an activity at operation level)
    Activity creation as well as adding traceparent header are done for both gateway and direct modes.

### **Different scenarios:**


- **EnableDistributedTracing Flag : True and Direct Mode**
    - Scenario 1:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **parentTraceId-spanId**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parentTraceId-spanId**
    - Scenario 2:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **Create a dummy activity with ParentTraceId**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **Parent TraceId-SpanId created for each request**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parent traceID-SpanId created for each request**
    - Scenario 3:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> No new activity created at network level
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **Checks if there is an existing Activity.Current.id then sets that and this way each network request has same ParentTraceId-SpanId combination**
    - Scenario 4:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **Create a dummy activity with ParentTraceId**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> **No Activity**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **Checks if there is an existing Activity.Current.id then sets that and this way each network request has same ParentTraceId-SpanId combination**
    - Scenario 5:
        - Batch and bulk operations

- **EnableDistributedTracing Flag : True and Gateway Mode**

    - Scenario 1:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **parentTraceId-spanId**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parentTraceId-spanId**
    - Scenario 2:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **Create a dummy activity with ParentTraceId**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **Parent TraceId-SpanId created for each request**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parent traceID-SpanId created for each request**
    - Scenario 3: (Still needs to be tested)
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> No new activity created at network level
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **Checks if there is an existing Activity.Current.id then sets that and this way each network request has same ParentTraceId-SpanId combination**
    - Scenario 4:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **Create a dummy activity with ParentTraceId**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> **No Activity**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **Checks if there is an existing Activity.Current.id then sets that and this way each network request has same ParentTraceId-SpanId combination**
     - Scenario 5:
        - Batch and bulk operations

- **EnableDistributedTracing Flag : False and Gateway/Direct Mode**
    - Currently we just have EnableDistributedTracing feature flag for controlling operation level activity creation.
    - For HTTP even if the EnableDistributedTracing flag is off HTTP request would have traceparent headers because that is something internally controlled by httpclient.
    - For TCP, if flag is off at operation level then again a flag check is needed in transportclient to stop request level activities to generate. Should we pass EnableDistributedTracing flag value all the way to TransportClient or just keep this behavior consistent to http?

