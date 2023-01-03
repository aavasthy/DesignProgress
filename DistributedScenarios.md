## **Cosmos DB SDK Distributed Tracing**

### **Different scenarios:**


- **EnableDistributedTracing Flag : True and Direct Mode**
    - Scenario 1:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **parentTraceId-spanId**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parentTraceId-spanId**
    - Scenario 2:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **No operation level trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **Unique TraceId-SpanId created for each request**
        - DiagnosticsString: **Includes list of traceIDs**
        - TraceParentHeader: **Unique TraceId-SpanId created for each request**
    - Scenario 3:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> 
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: No traceparent header
    - Scenario 4:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **No Activity**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> **No Activity**
        - DiagnosticsString: **Capture all TraceIds that come from response header**
        - TraceParentHeader: 

- **EnableDistributedTracing Flag : True and Gateway Mode**
    - Scenario 1:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **parentTraceId-spanId**
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: **parentTraceId-spanId**
    - Scenario 2:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **No operation level trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): True --> **Unique TraceId-SpanId created for each request**
        - DiagnosticsString: **Includes list of traceIDs**
        - TraceParentHeader: **Unique TraceId-SpanId created for each request**
    - Scenario 3:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): True --> **Activity Parent Trace id**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> 
        - DiagnosticsString: **Includes just parent traceID**
        - TraceParentHeader: Activity Parent Trace id
    - Scenario 4:
        - Client Listener for Operation Level Activity(Azure.Cosmos.Operation): False --> **No Activity**
        - Client listener for Network Level Activity(Azure.Cosmos.Request): False --> **No Activity**
        - DiagnosticsString: **Capture all TraceIds that come from response header**
        - TraceParentHeader: HTTP creates traceid-spanid 
- **EnableDistributedTracing Flag : False and Gateway/Direct Mode**
    - For HTTP even if the flag is off HTTP request would have traceparent headers because usually we have httplistener enabled.
    - For TCP, if flag is off then I need to add the check again for flag in transportclient to stop request level activities to generate?

