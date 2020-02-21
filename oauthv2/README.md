# oauthv2 plugin

## Summary
The `oauthv2` plugin allows you to secure API requests to Apigee Edge Microgateway with OAuth 2.0 (JWT validation).  Therefore, it was created to separate API key validation from OAuth 2.0 token validation and simply its configuration.

## When to use this plugin?
Use this plugin when you only want to enable OAuth 2.0 JWT validation.

## Process Summary

1. The client obtains an client ID and secret.
2. The client exchanges the client ID and secret for a JWT and includes it the request to Edge Microgateway.
3. The `oauthv2` plugin will validate the JWT.
4. If the JWT is valid, then the request will continue to the next plugin, otherwise an error message will be returned to the client application.

## Prerequisites
Please complete the following tasks before you use this plugin.  

1. [Install Edge Microgateway](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure Edge Microgateway](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)

3. [Create entities on Apigee Edge](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part2)


## Plugin configuration properties
You can set the following properties in the `oauthv2` stanza in the Edge Microgateway configuration file.

```yaml
oauth:
  # Header name used to send the JWT to Edge Microgateway
  # Default: Authorization

  authorization-header: "x-custom-auth-header"

  # Set to true if you want to send the Authorization header to the target server;
  # Set to false if you want this plugin to remove the header after it is validated.
  # Default: false

  keep-authorization-header: true

  # Grace period is the number of seconds before the token is removed from the cache.
  # If you set this to 5 (seconds), then MG will check if the difference between the expiry time and the current time [absoluteValue(expiry_time - current_time)] is less than or equal (<=) to the grace period.
  # If true, then Edge Microgateway will remove the token from the cache.  
  # Default: 0 seconds

  gracePeriod: 5

  # Set to true to enable Edge Microgateway to check against the resource paths only.  
  # In this case it ignores the proxy name check.
  # Default: false, which enables Edge Microgateway to check if the proxy name is included in the product.

  productOnly: true

  # Note: if you set the tokenCacheSize, then you should also enable it (tokenCache: true).
  # Set tokenCache to true if you want to cache the access token (JWT) locally.
  # Default: false - Access token is not cached.

  tokenCache: true

  # Set the maximum number of tokens cached locally.
  # Default: 100

  tokenCacheSize: 150
  
  # Skips the oauth if this is present and urls are mentioned. Example as:
  
  skipurl:
     - /healthcheck
     - /ping  
```

## Enable the plugin
In the EM configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauthv2
      # other plugins can be listed here
```

## Configure the plugin
In the same configuration file you also need to configure the `oauthv2` plugin if you want to change the default behavior.  The example below changes the `oauthv2` to enable access token caching and changes the `tokenCacheSize` to 150 tokens.    

```yaml
oauthv2:
  tokenCache: true
  tokenCacheSize: 150
```

## Caching
JWT token caching is **not** enabled by default.  

### Cache Headers
The `oauthv2` plugin **does not** observe the [`cache-control`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching) header.
The only way to cache the JWT is to set the `tokenCache` property to `true`.

### Access Tokens
Access tokens (JWTs) can also be cached in Edge Microgateway to avoid validating the JWT on every request.  Access tokens are only cached if you set `tokenCache` to `true` and the JWT is valid.  

## Best Practices for configuring this plugin
* The `oauthv2` plugin is typically listed first in the plugin sequence.  
* If you need one set of APIs to be validated by JWTs and another set to be validated by API Keys (lower security), then consider the following:
  * Create two products, one for API Key validation and one for access token (JWT) validation, then configure two sets of  Edge Microgateway instances - one set for API key validation and one set for access token validation.
  * Use an HTTPS load balancer to determine which Edge Microgateway should receive the traffic.  
