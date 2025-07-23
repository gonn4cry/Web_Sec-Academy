# Path mapping discrepancies

URL path mapping is the process of associating URL paths with resources on a server, such as files, scripts, or command executions. There are a range of different mapping styles used by different frameworks and technologies. Two common styles are traditional URL mapping and RESTful URL mapping.

Traditional URL mapping represents a direct path to a resource located on the file system. Here's a typical example:

http://example.com/path/in/filesystem/resource.html

    http://example.com points to the server.
    /path/in/filesystem/ represents the directory path in the server's file system.
    resource.html is the specific file being accessed.

In contrast, REST-style URLs don't directly match the physical file structure. They abstract file paths into logical parts of the API:

http://example.com/path/resource/param1/param2

    http://example.com points to the server.
    /path/resource/ is an endpoint representing a resource.
    param1 and param2 are path parameters used by the server to process the request.

Discrepancies in how the cache and origin server map the URL path to resources can result in web cache deception vulnerabilities. Consider the following example:

http://example.com/user/123/profile/wcd.css

    An origin server using REST-style URL mapping may interpret this as a request for the /user/123/profile endpoint and returns the profile information for user 123, ignoring wcd.css as a non-significant parameter.
    A cache that uses traditional URL mapping may view this as a request for a file named wcd.css located in the /profile directory under /user/123. It interprets the URL path as /user/123/profile/wcd.css. If the cache is configured to store responses for requests where the path ends in .css, it would cache and serve the profile information as if it were a CSS file.

# Exploiting path mapping discrepancies

To test how the origin server maps the URL path to resources, add an arbitrary path segment to the URL of your target endpoint. If the response still contains the same sensitive data as the base response, it indicates that the origin server abstracts the URL path and ignores the added segment. For example, this is the case if modifying /api/orders/123 to /api/orders/123/foo still returns order information.

To test how the cache maps the URL path to resources, you'll need to modify the path to attempt to match a cache rule by adding a static extension. For example, update /api/orders/123/foo to /api/orders/123/foo.js. If the response is cached, this indicates:

    That the cache interprets the full URL path with the static extension.
    That there is a cache rule to store responses for requests ending in .js.

Caches may have rules based on specific static extensions. Try a range of extensions, including .css, .ico, and .exe.

You can then craft a URL that returns a dynamic response that is stored in the cache. Note that this attack is limited to the specific endpoint that you tested, as the origin server often has different abstraction rules for different endpoints. 


# Lab: Exploiting path mapping for web cache deception
1. curl 'https://0ada00640380e01585758164009100b5.web-security-academy.net/my-account/abc.css'
2. Caching Headers
   cache-control: max-age=30
   age: 0
   x-cache: miss
3. Exploit : Craft a Javascript payload and sent it to victim.
4. <script>document.location="http://vulnerable.tld/path/foo..css"</script>

# Automation
<pre>
    #!/bin/bash
    url=$1
    check=$(curl -s -i -k '$url/foo.js')
    echo $check | grep "x-cache:"
    echo $check | grep "cache-control:"
    echo $check | grep "Sensitive"
</pre>
