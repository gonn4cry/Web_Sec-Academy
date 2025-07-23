# Web Cache Deception
Web Cache Deception is a vulnerability where an attacker tricks a caching system into storing and serving sensitive user specific content as if it were public. This happens when the cache treats a URL as static (like ending in .css), but the backend still returns private data based on the user's session.

# Cache keys 
When the cache receives an HTTP request, it must decide whether there is a cached response that it can serve directly, or whether it has to forward the request to the origin server. The cache makes this decision by generating a 'cache key' from elements of the HTTP request. Typically, this includes the URL path and query parameters, but it can also include a variety of other elements like headers and content type.

If the incoming request's cache key matches that of a previous request, the cache considers them to be equivalent and serves a copy of the cached response. 

# Cache rules
 Cache rules determine what can be cached and for how long. Cache rules are often set up to store static resources, which generally don't change frequently and are reused across multiple pages. Dynamic content is not cached as it's more likely to contain sensitive information, ensuring users get the latest data directly from the server.

Web cache deception attacks exploit how cache rules are applied, so it's important to know about some different types of rules, particularly those based on defined strings in the URL path of the request. For example:

    [ Static file extension rules ] - These rules match the file extension of the requested resource, for example .css for stylesheets or .js for JavaScript files.
    [ Static directory rules ] - These rules match all URL paths that start with a specific prefix. These are often used to target specific directories that contain only static resources, for example /static or /assets.
    [ File name rules ] - These rules match specific file names to target files that are universally required for web operations and change rarely, such as robots.txt and favicon.ico.

Caches may also implement custom rules based on other criteria, such as URL parameters or dynamic analysis. 
# Detecting cached responses
    The X-Cache header provides information about whether a response was served from the cache. Typical values include:
        X-Cache: hit - The response was served from the cache.
        X-Cache: miss - The cache did not contain a response for the request's key, so it was fetched from the origin server. In most cases, the response is then cached. To confirm this, send the request again to see whether the value updates to hit.
        X-Cache: dynamic - The origin server dynamically generated the content. Generally this means the response is not suitable for caching.
        X-Cache: refresh - The cached content was outdated and needed to be refreshed or revalidated.
    The Cache-Control header may include a directive that indicates caching, like public with a max-age higher than 0. Note that this only suggests that the resource is cacheable. It isn't always indicative of caching, as the cache may sometimes override this header.



