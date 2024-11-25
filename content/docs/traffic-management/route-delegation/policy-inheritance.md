---
title: Policy inheritance
weight: 80
description: Learn how policy inheritance works in a route delegation setup. 
---

Learn how policy inheritance works in a route delegation setup. 

## About policy inheritance

{{< reuse "docs/snippets/policy-inheritance.md" >}}

{{< callout type="info" >}}
The example in this guide attaches the RouteOption policies by using the [`targetRefs` attachment setting](/docs/about/policies/routeoption/#option-1-attach-the-policy-to-all-httproute-routes-targetrefs). You might attach RouteOption resources by using the [`extensionRef` filter in the HTTPRoute resource](/docs/about/policies/routeoption/#option-2-attach-the-policy-to-an-individual-route-extensionref) instead. Note that attaching RouteOptions in different ways can result in conflicting policies or policy merging. To learn more about how conflicting RouteOption resources are handled, see [Conflicting policies](/docs/about/policies/routeoption/#conflicting-policies-and-merging-rules) in the RouteOption concepts. 
{{< /callout >}}

## Configuration overview

In this guide you walk through a route delegation example where a child HTTPRoute inherits policies that are set on a parent HTTPRoute. In addition, the child HTTPRoute defines other policies that are only applied to the child HTTPRoute. 

The following image illustrates the route delegation hierarchy and policy inheritance:

{{< reuse-image src="img/route-delegation-policy-inheritance.svg" >}}
<!-- https://app.excalidraw.com/s/AKnnsusvczX/9uktq3x1i63-->

**`parent` HTTPRoute**: 
* The `parent` HTTPRoute resource delegates traffic as follows: 
  * Requests to`/anything/team1` are delegated to the child HTTPRoute resource `child-team1` in namespace `team1`. 
  * Requests to `/anything/team2` are delegated to the child HTTPRoute resource `child-team2` in namespace `team2`. 
* The response header `test: bar` is added to all requests to the routes that the parent HTTPRoute serves. This policy is defined in a RouteOption resource that is applied to the parent HTTPRoute by using the `targetRefs` field. 

**`child-team1` HTTPRoute**: 
* The child HTTPRoute resource `child-team1` matches incoming traffic for the `/anything/team1/foo` prefix path and routes that traffic to the httpbin app in the `team1` namespace. 
* The `child-team1` resource inherits the response header policy from the parent HTTPRoute. In addition, a RouteOption resource that specifies a fault injection policy is attached to all `child-team1` routes and aborts all incoming requests with a 418 HTTP response code. 

**`child-team2` HTTPRoute**: 
* The child HTTPRoute resource `child-team2` delegates traffic on the `/anything/team2/bar` path to a grandchild HTTPRoute resource in the `team2` namespace. 
* The `child-team2` resource inherits the response header policy from the parent HTTPRoute. In addition, a RouteOption resource that specifies a prefix rewrite policy is attached to all `child-team2` routes and rewrites prefix paths to `/anything/rewrite`. 

**`grandchild` HTTPRoute**: 
* The grandchild HTTPRoute resource `grandchild` matches incoming traffic for the `/anything/team2/bar/.*` regex path and routes that traffic to the httpbin app in the `team2` namespace. 
* The `grandchild` HTTPRoute inherits all policies from the `parent` and `child-team2` HTTPRoute resources. Because of that, requests to the `/anything/team2/bar/.*` regex path are rewritten to `/anything/rewrite`. In addition, the `test: bar` header is added to all responses. 


## Before you begin

{{< reuse "docs/snippets/prereq-delegation.md" >}}

## Setup

Set up a parent, child, grandchild route delegation example and explore how policies are inherited in the delegation chain.  

1. Create the parent HTTPRoute resource that matches incoming traffic on the `delegation.example` domain. The HTTPRoute resource specifies two routes: 
   * `/anything/team1`: The routing decision is delegated to a `child-team1` HTTPRoute resource in the `team1` namespace. 
   * `/anything/team2`: The routing decision is delegated to a `child-team2` HTTPRoute resource in the `team2` namespace. 

   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
    name: parent
    namespace: gloo-system
   spec:
    parentRefs:
    - name: http
    hostnames:
     - "delegation.example"
    rules:
    - matches:
      - path:
          type: PathPrefix
          value: /anything/team1
      backendRefs:
      - group: gateway.networking.k8s.io
        kind: HTTPRoute
        name: "*"
        namespace: team1
    - matches:
      - path:
          type: PathPrefix
          value: /anything/team2
      backendRefs:
      - group: gateway.networking.k8s.io
        kind: HTTPRoute
        name: "*"
        namespace: team2
   EOF
   ```

2. Create a RouteOption resource that adds the `test: bar` header to the response and apply this policy to the `parent` HTTPRoute resource by using the `spec.targetRefs` section. 
   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.solo.io/v1
   kind: RouteOption
   metadata:
    name: parent-remove-header
    namespace: gloo-system
   spec:
    targetRefs:
    - group: gateway.networking.k8s.io
      kind: HTTPRoute
      name: parent
    options:
      headerManipulation:
        responseHeadersToAdd:
        - header:
            key: test
            value: bar
   EOF
   ```

3. Create the `child-team1` HTTPRoute resource in the `team1` namespace that matches traffic on the `/anything/team1/foo` prefix and routes traffic to the httpbin app in the `team1` namespace. The child HTTPRoute resource does not select a specific parent HTTPRoute resource. Because of that, the child HTTPRoute resource is automatically selected by all parent HTTPRoute resources that delegate traffic to this child. 
   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
    name: child-team1
    namespace: team1
   spec:
    rules:
    - matches:
      - path:
          type: PathPrefix
          value: /anything/team1/foo
      backendRefs:
      - name: httpbin
        port: 8000
   EOF
   ```

4. Create a RouteOption resource that aborts all requests with a 418 HTTP response code and apply this policy to the `child-team1` HTTPRoute resource by using the `spec.targetRefs` section.  
   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.solo.io/v1
   kind: RouteOption
   metadata:
     name: child-team1-fault
     namespace: team1
   spec:
     targetRefs:
     - group: gateway.networking.k8s.io
       kind: HTTPRoute
       name: child-team1
     options:
       faults:
         abort:
           percentage: 100
           httpStatus: 418
   EOF
   ```

5. Create the `child-team2` HTTPRoute resource in the `team2` namespace that matches traffic on the `/anything/team2/bar/` prefix and delegates traffic to the `grandchild` HTTPRoute resource in the `team2` namespace. Note that because the child delegates traffic to a grandchild, a `PathPrefix` matcher must be used. 
   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: child-team2
     namespace: team2
   spec:
     parentRefs:
     - name: parent
       namespace: gloo-system
       group: gateway.networking.k8s.io
       kind: HTTPRoute
     rules:
     - matches:
       - path:
           type: PathPrefix
           value: /anything/team2/bar/
       backendRefs:
       - group: gateway.networking.k8s.io
         kind: HTTPRoute
         name: "*"
         namespace: team2
   EOF
   ```

6. Create a RouteOption resource that rewrites prefix paths to `/anything/rewrite` apply this policy to the `child-team2` HTTPRoute resource by using the `spec.targetRefs` section.  
   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.solo.io/v1
   kind: RouteOption
   metadata:
     name: child-team2-rewrite
     namespace: team2
   spec:
     targetRefs:
     - group: gateway.networking.k8s.io
       kind: HTTPRoute
       name: child-team2
     options:
       prefixRewrite: /anything/rewrite
   EOF
   ```

7. Create a grandchild HTTPRoute resource that matches traffic on the `/anything/team2/.*` regex path and routes traffic to the httpbin app in the `team2` namespace.
   ```yaml
   kubectl apply -f- <<EOF 
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: grandchild
     namespace: team2
   spec:
     rules:
     - matches:
       - path:
           type: RegularExpression
           value: /anything/team2/bar/.*
       backendRefs:
       - name: httpbin
         port: 8000
   EOF
   ```
   
Great job. You successfully deployed the multi-level route delegation example that is defined in the [Configuration overview](#configuration-overview). Next, you verify that the policies are correctly inherited by sending requests along the different requests paths in the delegation chain. 
   
## Verify 

Verify that the policies are correctly inherited along the delegation chain. 
   
1. Send a request to the `delegation.example` domain along the `/anything/team1/foo` path. Because `child1-team1` inherits the response header policy from the `parent` and applies a fault injection policy that aborts all incoming requests, verify that you get back a 418 HTTP response code and that you see the `test: bar` header in your response. 
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   curl -vik http://$INGRESS_GW_ADDRESS:8080/anything/team1/foo \
   -H "host: delegation.example:8080"
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   curl -vik localhost:8080/anything/team1/foo \
   -H "host: delegation.example"
   ```
   {{% /tab %}}
   {{< /tabs >}}

   Example output: 
   ```yaml {linenos=table,hl_lines=[2,3,4,5],linenostart=1}
   * Mark bundle as not supporting multiuse
   < HTTP/1.1 418 Unknown
   HTTP/1.1 418 Unknown
   < test: bar
   test: bar
   < content-length: 18
   content-length: 18
   < content-type: text/plain
   content-type: text/plain
   ...
   fault filter abort
   ```

2. Send another request to the `delegation.example` domain along the `/anything/team2/bar/test` path. The `grandchild` inherits the response header policy from the parent and the rewrite policy from `child-team2`. Verify that you get back a 200 HTTP response code, that the request path is rewritten to `/anything/rewrite`, and the `test: bar` response header is added. 
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   curl -vik http://$INGRESS_GW_ADDRESS:8080/anything/team2/bar/test \
   -H "host: delegation.example:8080"
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   curl -vik localhost:8080/anything/team2/bar/test \
   -H "host: delegation.example"
   ```
   {{% /tab %}}
   {{< /tabs >}}

   Example output:
   ```yaml {linenos=table,hl_lines=[2,3,16,17,37,38,48],linenostart=1}
   * Mark bundle as not supporting multiuse
   < HTTP/1.1 200 OK
   HTTP/1.1 200 OK
   < access-control-allow-credentials: true
   access-control-allow-credentials: true
   < access-control-allow-origin: *
   access-control-allow-origin: *
   < content-type: application/json; encoding=utf-8
   content-type: application/json; encoding=utf-8
   < date: Tue, 11 Jun 2024 17:42:45 GMT
   date: Tue, 11 Jun 2024 17:42:45 GMT
   < content-length: 579
   content-length: 579
   < x-envoy-upstream-service-time: 5
   x-envoy-upstream-service-time: 5
   < test: bar
   test: bar
   < server: envoy
   server: envoy

   < 
   {
     "args": {},
     "headers": {
       "Accept": [
         "*/*"
       ],
       "Host": [
         "delegation.example:8080"
       ],
       "User-Agent": [
         "curl/7.77.0"
       ],
       "X-Envoy-Expected-Rq-Timeout-Ms": [
         "15000"
       ],
       "X-Envoy-Original-Path": [
         "/anything/team2/bar/test"
       ],
       "X-Forwarded-Proto": [
         "http"
       ],
       "X-Request-Id": [
         "d936987f-879f-474c-be2c-ed66e85944f4"
       ]
     },
     "origin": "10.XX.X.XX:49018",
     "url": "http://delegation.example:8080/anything/rewrite",
     "data": "",
     "files": null,
     "form": null,
     "json": null
   }
   ```
   
## Cleanup

{{< reuse "docs/snippets/cleanup.md" >}}

```sh
kubectl delete httproute parent -n gloo-system
kubectl delete httproute child-team1 -n team1
kubectl delete httproute child-team2 -n team2
kubectl delete httproute grandchild -n team2
kubectl delete routeoption parent-remove-header -n gloo-system
kubectl delete routeoption child-team1-fault -n team1
kubectl delete routeoption child-team2-rewrite -n team2
kubectl delete -n team1 -f https://raw.githubusercontent.com/solo-io/gloo-mesh-use-cases/main/policy-demo/httpbin.yaml
kubectl delete -n team2 -f https://raw.githubusercontent.com/solo-io/gloo-mesh-use-cases/main/policy-demo/httpbin.yaml
kubectl delete namespaces team1 team2