---
title: Inject response headers
weight: 20
description: Extract values from a request header and inject it as a header to your response. 
---

The following example walks you through how to use an Inja template to extract a value from a request header and to add this value as a header to your responses. 

## Before you begin

{{< reuse "docs/snippets/prereq.md" >}}

## Inject response headers
   
1. Create a VirtualHostOption resource with your transformation rules. In the following example, you use the value from the `x-solo-request` request header and populate the value of that header into an `x-solo-response` response header.  
   ```yaml
   kubectl apply -n gloo-system -f- <<EOF
   apiVersion: gateway.solo.io/v1
   kind: VirtualHostOption
   metadata:
     name: transformation
     namespace: gloo-system
   spec:
     options:
       transformations:
         responseTransformation:
           transformationTemplate:
             headers:
               x-solo-response:
                 text: '{{ request_header("x-solo-request") }}'
     targetRefs:
     - group: gateway.networking.k8s.io
       kind: Gateway
       name: http
       namespace: gloo-system
   EOF
   ```

2. Send a request to the httpbin app and include the `x-solo-request` request header. Verify that you get back a 200 HTTP response code and that the value of the `x-solo-request` header was added to the `x-solo-response` response header. 
    {{< tabs tabTotal="2" >}}
   {{% tab tabName="LoadBalancer IP address or hostname" %}}
   ```sh
   curl -vik http://$INGRESS_GW_ADDRESS:8080/response-headers \
    -H "host: www.example.com:8080" \
    -H "x-solo-request: my custom request header" 
   ```
   {{% /tab %}}
   {{% tab tabName="Port-forward for local testing" %}}
   ```sh
   curl -vik localhost:8080/response-headers \
   -H "host: www.example.com" \
   -H "x-solo-request: my custom request header"
   ```
   {{% /tab %}}
   {{< /tabs >}}
   
   Example output: 
   ```yaml {linenos=table,hl_lines=[20,21],linenostart=1}
   * Mark bundle as not supporting multiuse
   < HTTP/1.1 200 OK
   HTTP/1.1 200 OK
   < access-control-allow-credentials: true
   access-control-allow-credentials: true
   < access-control-allow-origin: *
   access-control-allow-origin: *
   < content-type: application/json; encoding=utf-8
   content-type: application/json; encoding=utf-8
   < date: Wed, 26 Jun 2024 02:54:48 GMT
   date: Wed, 26 Jun 2024 02:54:48 GMT
   < content-length: 3
   content-length: 3
   < x-envoy-upstream-service-time: 2
   x-envoy-upstream-service-time: 2
   < server: envoy
   server: envoy
   < x-envoy-decorator-operation: httpbin.httpbin.svc.cluster.local:8000/*
   x-envoy-decorator-operation: httpbin.httpbin.svc.cluster.local:8000/*
   < x-solo-response: my custom request header
   x-solo-response: my custom request header
   ```
   
## Cleanup

{{< reuse "docs/snippets/cleanup.md" >}}

```sh
kubectl delete virtualhostoption transformation -n gloo-system
```
   