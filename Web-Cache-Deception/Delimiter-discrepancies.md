# Delimiter discrepancies

Delimiters specify boundaries between different elements in URLs. The use of characters and strings as delimiters is generally standardized. For example, ? is generally used to separate the URL path from the query string. However, as the URI RFC is quite permissive, variations still occur between different frameworks or technologies.

Discrepancies in how the cache and origin server use characters and strings as delimiters can result in web cache deception vulnerabilities. Consider the example /profile;foo.css:

    The Java Spring framework uses the ; character to add parameters known as matrix variables. An origin server that uses Java Spring would therefore interpret ; as a delimiter. It truncates the path after /profile and returns profile information.
    Most other frameworks don't use ; as a delimiter. Therefore, a cache that doesn't use Java Spring is likely to interpret ; and everything after it as part of the path. If the cache has a rule to store responses for requests ending in .css, it might cache and serve the profile information as if it were a CSS file.

The same is true for other characters that are used inconsistently between frameworks or technologies. Consider these requests to an origin server running the Ruby on Rails framework, which uses . as a delimiter to specify the response format:

    /profile - This request is processed by the default HTML formatter, which returns the user profile information.
    /profile.css - This request is recognized as a CSS extension. There isn't a CSS formatter, so the request isn't accepted and an error is returned.
    /profile.ico - This request uses the .ico extension, which isn't recognized by Ruby on Rails. The default HTML formatter handles the request and returns the user profile information. In this situation, if the cache is configured to store responses for requests ending in .ico, it would cache and serve the profile information as if it were a static file.

Encoded characters may also sometimes be used as delimiters. For example, consider the request /profile%00foo.js:

    The OpenLiteSpeed server uses the encoded null %00 character as a delimiter. An origin server that uses OpenLiteSpeed would therefore interpret the path as /profile.
    Most other frameworks respond with an error if %00 is in the URL. However, if the cache uses Akamai or Fastly, it would interpret %00 and everything after it as the path.

# Exploiting delimiter discrepancies

You may be able to use a delimiter discrepancy to add a static extension to the path that is viewed by the cache, but not the origin server. To do this, you'll need to identify a character that is used as a delimiter by the origin server but not the cache.

Firstly, find characters that are used as delimiters by the origin server. Start this process by adding an arbitrary string to the URL of your target endpoint. For example, modify /settings/users/list to /settings/users/listaaa. You'll use this response as a reference when you start testing delimiter characters.
Note

If the response is identical to the original response, this indicates that the request is being redirected. You'll need to choose a different endpoint to test.

Next, add a possible delimiter character between the original path and the arbitrary string, for example /settings/users/list;aaa:

    If the response is identical to the base response, this indicates that the ; character is used as a delimiter and the origin server interprets the path as /settings/users/list.
    If it matches the response to the path with the arbitrary string, this indicates that the ; character isn't used as a delimiter and the origin server interprets the path as /settings/users/list;aaa.

Once you've identified delimiters that are used by the origin server, test whether they're also used by the cache. To do this, add a static extension to the end of the path. If the response is cached, this indicates:

    That the cache doesn't use the delimiter and interprets the full URL path with the static extension.
    That there is a cache rule to store responses for requests ending in .js.

Make sure to test all ASCII characters and a range of common extensions, including .css, .ico, and .exe. We've provided a list of potential delimiter characters to get you started in the labs, see the Web cache deception lab delimiter list. Use Burp Intruder to quickly test these characters. To prevent Burp Intruder from encoding the delimiter characters, turn off Burp Intruder's automated character encoding under Payload encoding in the Payloads side panel.

You can then construct an exploit that triggers the static extension cache rule. For example, consider the payload /settings/users/list;aaa.js. The origin server uses ; as a delimiter:

    The cache interprets the path as: /settings/users/list;aaa.js
    The origin server interprets the path as: /settings/users/list

The origin server returns the dynamic profile information, which is stored in the cache.

Because delimiters are generally used consistently within each server, you can often use this attack on many different endpoints.
Note

Some delimiter characters may be processed by the victim's browser before it forwards the request to the cache. This means that some delimiters can't be used in an exploit. For example, browsers URL-encode characters like {, }, <, and >, and use # to truncate the path.

If the cache or origin server decodes these characters, it may be possible to use an encoded version in an exploit.

# Delimiter list
<pre>
    !
"
#
$
%
&
'
(
)
*
+
,
-
.
/
:
;
<
=
>
?
@
[
\
]
^
_
`
{
|
}
~
%21
%22
%23
%24
%25
%26
%27
%28
%29
%2A
%2B
%2C
%2D
%2E
%2F
%3A
%3B
%3C
%3D
%3E
%3F
%40
%5B
%5C
%5D
%5E
%5F
%60
%7B
%7C
%7D
%7E
</pre>
