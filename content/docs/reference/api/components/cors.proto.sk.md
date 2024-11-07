---
title: "cors.proto"
weight: 10
---

<!-- Code generated by solo-kit. DO NOT EDIT. -->


**Package**: `cors.options.gloo.solo.io` 
**Types**:


- [CorsPolicy](#corspolicy)
- [CorsPolicyMergeSettings](#corspolicymergesettings)
- [MergeStrategy](#mergestrategy)
  


**Source File**: [github.com/solo-io/gloo/projects/gloo/api/v1/options/cors/cors.proto](https://github.com/solo-io/gloo/blob/main/projects/gloo/api/v1/options/cors/cors.proto)





---
### CorsPolicy

 
CorsPolicy defines Cross-Origin Resource Sharing for a virtual service.

```yaml
"allowOrigin": []string
"allowOriginRegex": []string
"allowMethods": []string
"allowHeaders": []string
"exposeHeaders": []string
"maxAge": string
"allowCredentials": bool
"disableForRoute": bool

```

| Field | Type | Description |
| ----- | ---- | ----------- | 
| `allowOrigin` | `[]string` | Specifies the origins that will be allowed to make CORS requests. An origin is allowed if either allow_origin or allow_origin_regex match. |
| `allowOriginRegex` | `[]string` | Specifies regex patterns that match origins that will be allowed to make CORS requests. An origin is allowed if either allow_origin or allow_origin_regex match. |
| `allowMethods` | `[]string` | Specifies the content for the *access-control-allow-methods* header. |
| `allowHeaders` | `[]string` | Specifies the content for the *access-control-allow-headers* header. |
| `exposeHeaders` | `[]string` | Specifies the content for the *access-control-expose-headers* header. |
| `maxAge` | `string` | Specifies the content for the *access-control-max-age* header. |
| `allowCredentials` | `bool` | Specifies whether the resource allows credentials. |
| `disableForRoute` | `bool` | Optional, only applies to route-specific CORS Policies, defaults to false. If set, the CORS Policy (specified on the virtual host) will be disabled for this route. |




---
### CorsPolicyMergeSettings

 
Settings to determine how to merge CORS settings when present on both the VirtualHost and Route.
This option can be useful when different user personas or teams share ownership of a VirtualService.
For example, you might not want CORS settings on each route to override the virtual host settings, but instead merge them with a `UNION` strategy.

```yaml
"exposeHeaders": .cors.options.gloo.solo.io.CorsPolicyMergeSettings.MergeStrategy

```

| Field | Type | Description |
| ----- | ---- | ----------- | 
| `exposeHeaders` | [.cors.options.gloo.solo.io.CorsPolicyMergeSettings.MergeStrategy](../cors.proto.sk/#mergestrategy) |  |




---
### MergeStrategy



| Name | Description |
| ----- | ----------- | 
| `DEFAULT` | Follow the default Envoy behavior, which is for Route settings to override VirtualHost settings if non-nil. |
| `UNION` | When a CORS policy is present on both VirtualHost and Route CORS policy, merge the settings. The merge is done by concatenating for list fields and by ORing for boolean fields. |





<!-- Start of HubSpot Embed Code -->
<script type="text/javascript" id="hs-script-loader" async defer src="//js.hs-scripts.com/5130874.js"></script>
<!-- End of HubSpot Embed Code -->