## **Cosmos DB SDK Distributed Tracing**
 ### **Purpose**

The goal is to be able to implement distributed tracing transactions within the Cosmos DB .Net SDK for every request that is to be delivered downstream into the compute gateway/ routing gateway/backend services where these capabilities are to be handled by the Monitoring and Observability team. 

Distributed tracing is a diagnostic technique that helps localize issues within applications that are distributed across multiple machines or processes including retries on request timeouts.

With the proposed solution, Cosmos DB .Net SDK would have Traces enabled and it would be instrumented via OpenTelemetry. 

<img width="551" alt="image" src="https://user-images.githubusercontent.com/113193425/199226483-5e12d6d1-5822-4fa0-b635-b1a896d02622.png">

### **Main Deliverables:**


- #### **SDK team**
    #### **Feature**: Propagate Distributed trace context to the server side for both HTTP and TCP calls.
    The client SDKs need to propagate distributed  trace to service side/ dependencies. This propagation of trace would help in stitching together the client side/service side calls and will show the user the result in spans on Transaction diagnostics . This will enable Azure user to to troubleshoot quickly and swiftly any failures that occurs related to cosmos db.
    - Create design document for Distributed Tracing in .Net SDK- **DONE**
        - Have a design document created for the feature. Capture the workflows and implementation details for Direct mode and Gateway mode.
Present it to the Monitoring team and SDK team.
    - POC/Investigations and Implementation for Distributed Tracing in .Net SDK- **DONE**
        - Direct Mode: Added Diagnostic scope code from Azure.Core to Direct Package. Verified that this approach is feasible for adding diagnostics to Direct package.

        - Gateway Mode: Implemented distributed tracing using HTTPClient. As it already creates and propagates activity with traceId and spanId information. 
        Tested scenarios for making HTTP calls and verified that expected activity results are obtained.

    - Implementation of Distributed Tracing for Direct Mode- **IN PROGRESS**
        - Add Azure Core Diagnostics code in direct package.
        - Start Activity around Transport Client to cover regular as well as retry requests.
        - Run and test scenarios doing TCP call. Add test cases for the same.
    - Implementation of Distributed Tracing for Gateway Mode- **NEW**
        - Implement enabled/disabled logic for distributed tracing.
        - Include TraceId in Diagnostics string
        - Test and verify the existing logic for trace distribution in HTTPClient calls.
    - Cleanup: Using Diagnostic Factory from Direct package in V3- **NEW**
        - This is independent from feature objective but we need to maintain copied Azure Core diagnostic code only in one place. Therefore, remove Azure Core Diagnostic code from V3 and using Direct Package reference.

## **Different scenarios:**


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



- #### **Monitoring team**
    #### Feature: Enabling Distributed Tracing across components within Cosmos DB Service
    - Emitting Distributed Trace into the Azure Monitor Storage Account (Compute Gateway)- **DONE**
    - POC/Investigations and Implementation for Compute Gateway Incoming Span- **DONE**
    - POC implementation for emitting a well defined V1 version of the trace- **DONE**
    - POC/Investigations and Implementation for Backend Incoming Span- **IN PROGRESS**
    - POC/Investigations and Implementation for Routing Gateway Incoming Span- **IN PROGRESS**
    - Implementation of DT in all the entry point services of the COSMOS DB- **NEW**
    - Design and implement Sampling- **NEW**

### **Blockers**
- Initially it was team inter dependency but planning to setup our own test plans
- No blocker as such but setting up environments for E2E integration testing takes time.

### **Timelines**

- SDK team is aiming for code complete and just sdk scenario level testing by January
- Monitoring was initially planning to release changes in March but they revising their timelines.

