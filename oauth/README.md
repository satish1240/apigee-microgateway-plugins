# oauth plugin

## Summary

The `oauth` plugin allows you to secure API requests to Apigee Edge Microgateway with either API keys or OAuth 2.0 tokens. Note that, buy default, when you include this plugin in the plugins sequence, you can use both API Key validation and OAuth 2.0 validation on API requests. However, you can change the default behavior to allow either API key validation **or** OAuth 2.0  validation.


## When to use this plugin?

Use this plugin when you want to enable **both**:

* API Key validation **and**
* OAuth 2.0

This plugin also allows you to change the default behavior above by setting one of two properties.
* `allowOAuthOnly` - Set to `true` to only allow OAuth 2.0 access tokens. These tokens are formatted as Java Web Tokens (JWT). This setting disables API Key validation
* `allowAPIKeyOnly` - Set to `true` to only allow API key validation, which disables OAuth 2.0 access tokens

## Process summary

The following steps describe how the plugin operates in an API request flow:

1. The client obtains a client ID and secret.
2. The client includes either:
   * the API Key on each API request or
   * exchanges the client ID and secret for a JWT and includes the JWT in each request to Edge Microgateway.
3. The `oauth` plugin validates either the API Key or the JWT.
4. If the API Key or JWT is valid, then processing continues to the next plugin, otherwise an error message is returned to the client application.

## Prerequisites

To use the plugin, you must have installed and configured an instance of Edge Microgateway. You also must have
created a product, developer, and developer app on Apigee Edge Cloud. For details, see the following documentation topics:

1. [Install Edge Microgateway](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure Edge Microgateway](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)

3. [Create entities on Apigee Edge](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part2)


## Plugin configuration properties

You can set the following properties in the `oauth` stanza in the Edge Microgateway configuration file.

```yaml
oauth:
  # Header name used to send the JWT to Edge Microgateway
  # Default: Authorization: Bearer

  authorization-header: "x-custom-auth-header"

  # Header name used to send the API Key to Edge Microgateway

  api-key-header: "x-custom-header" # defaults to x-api-key

  # Set to true if you want to send the Authorization header to the target server;
  # Set to false when you want this plugin to remove the header after it is validated.
  # Default: false

  keep-authorization-header: true

  # Set to true if you want to enable Edge Microgateway to cache the JWT that is
  # received when the API Key is validated.
  # Default: false, which validates the API key with Apigee Edge on each request

  cacheKey: true

  # Number of seconds before the token is removed from the cache.
  # If you set this to 5 (seconds) then Edge Microgateway will check if the difference
  # between the expiry time and the current time [abs(expiry time - current time)] is
  # less than or equal (<=) to the grace period.  If true, then Edge Microgateway will
  # remove the token from the cache.  
  # Default: 0 seconds

  gracePeriod: 5

  ## Do not set allowOAuthOnly and allowAPIKeyOnly both to true. Only one of them should be set true.
  # Set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  # Default: false, which allows both API Key and OAuth 2.0

  allowOAuthOnly: true

  # Set to true if you want to allow OAuth 2.0 only.  This will disable API Key validation.
  # Default: false, which allows both API Key and OAuth 2.0

  allowAPIKeyOnly: true

  # Set to true to enable Edge Microgateway to check against the resource paths only.
  # In this case it ignores the proxy name check.  
  # Default: false, which enables Edge Microgateway to check if the proxy name is included in the product.


  productOnly: true

  ## Note that if you set the tokenCacheSize, then you should also enable it (tokenCache: true)
  # Set tokenCache to true if you want to cache the access token (JWT) that is received after the
  # API key is validated.
  # Default: false, which does not cache the access token.

  tokenCache: true

  # Set the number of tokens allowed to be cached locally.
  # Default: 100

  tokenCacheSize: 150
  
  # Skips the oauth if this is present and urls are mentioned. Example as:
  
  skipurl:
     - /healthcheck
     - /ping
```

## Enable the plugin
In the Edge Microgateway configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauth
      # other plugins can be listed here
```

## Configure the plugin
In the same configuration file you also need to configure the `oauth` plugin if you want to change the default behavior.  The example below changes the `oauth` plugin to allow OAuth 2.0 access tokens (JWTs) only, enables access token caching, and changes the `tokenCacheSize` to 150 tokens.    

```yaml
oauth:
  allowOAuthOnly: true
  tokenCache: true
  tokenCacheSize: 150
```

## API key validation

Edge Microgateway exchanges the API Key for a JWT when it validates the API key with Apigee Edge.

## Caching

### Caching API keys
If you set `cacheKey` to `true`, then Edge Microgateway will cache the JWT token that it receives in exchange for the API Key.  This configuration prevents Edge Microgateway from sending a request to Apigee Edge to validate the API Key on every request.

### Using cache headers
Edge Microgateway observes the [`cache-control`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching) header, but in a limited fashion:
  * If a client application wants to cache the JWT that is received after the API Key is validated, then it should set the `cache-control` header to any value except `no-cache`.
  * If you send this header in the request (`cache-control: max-age=120`), then the JWTs will be cached in Edge Microgateway even if the `cacheKey` is set to `false`.
  * If you set `cacheKey` to `true` and you send a request to Edge Microgateway with `cache-control: no-cache`, the JWT **will** be cached anyway.  

Client applications are not allowed to override the `cacheKey` setting with the `cache-control` header.  This plugin does not allow you to set the cache expiry time in the `cache-control` header; it uses the expiry time in the JWT to determine the TTL of the cache.  

### Caching access tokens
Access tokens (JWTs) can also be cached in Edge Microgateway to avoid validating the JWT on every request.  Access tokens are only cached if you set `tokenCache` to `true` and the JWT is valid.  

## Best Practices for configuring this plugin
* List the `oauth` plugin first in the plugin sequence.  
* Although the plugin lets you use an API key or an OAuth token for authentication, it's a good practice to restrict the plugin to use one or the other.
* Do not set `allowOAuthOnly` and `allowAPIKeyOnly` both to true. Only one of them should be set true.
* Consider using the `oauthv2` or `apikeys` plugins instead of this plugin.  
* If you need one set of APIs to be validated by JWTs and another set to be validated by API Keys (lower security), then consider the following:
  * Create two products, one for API Key validation and one set for access token (JWT) validation, then configure two sets for Edge Microgateway instances - one set for API key validation and one set for access token validation.
  * Use an HTTPS load balancer to handle routing between these Edge Microgateway instances.  

## To learn more
To learn more, review the following `oauth` plugin documentation.  
* [Edge Microgateway existing plugins](https://docs.apigee.com/api-platform/microgateway/2.5.x/use-plugins#existingpluginsbundledwithedgemicrogateway)
* [Creating entities on Apigee Edge plugin](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part2createentitiesonapigeeedge)
* [Secure Edge Microgateway with OAuth2.0](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway)
* [Secure Edge Microgateway with API Key validation](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway-securingtheapiwithanapikey)
