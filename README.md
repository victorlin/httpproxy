httpproxy
=========

httpproxy is a simple transparent HTTP proxy library.

Example
=======

With httpproxy, it's very easy to implement a transparent proxy that can manipulate both outgoing request and incoming response. Following is a Hodor proxy example, it replaces all words with `hodor` in the response.

```python
from __future__ import unicode_literals
import re

from httpproxy import HTTPProxyApplication


class HodorProxy(object):
    word_pattern = re.compile(r'(\w+)')

    def __init__(self):
        self.scheme = 'http'
        self.host = 'example.com'

    def egress_handler(self, uri, method, status, headers, data):
        hodor = 'hodor'
        data = self.word_pattern.sub(hodor, data)
        headers['Content-Length'] = str(len(data))
        return dict(
            status=status,
            headers=headers,
            data=data,
        )

if __name__ == '__main__':
    proxy_app = HTTPProxyApplication(__name__)
    proxy_app.config['DEBUG'] = True
    proxy_app.config['HTTP_PROXY_FACTORY'] = lambda request: HodorProxy()
    proxy_app.run()

```

It works like this, the first thing is to create the `HTTPProxyApplication` application, then you set the `HTTP_PROXY_FACTORY` value in the app
configuration. It should be an callbal object accepts request as argument and
return a proxy object. A proxy object should provides

 - scheme
 - host

attributes. The `scheme` is target HTTP server scheme to be used, can be either
`http` or `https`. And the `host` is the host name of target server. Proxy can
also provides `ingress_handler` and `egress_handler` methods for manipulating
ingress requests and egress requests. In this example, you can run it, and
hit the proxy like this

```
curl localhost:5000 -H "Host: example.com"
```

then you should be able to see the beautiful Hodor web content like this:

```html
<!hodor hodor>
<hodor>
<hodor>
    <hodor>hodor hodor</hodor>

    <hodor hodor="hodor-hodor" />
    <hodor hodor-hodor="hodor-hodor" hodor="hodor/hodor; hodor=hodor-hodor" />
    <hodor hodor="hodor" hodor="hodor=hodor-hodor, hodor-hodor=hodor" />
    <hodor hodor="hodor/hodor">
...
```
