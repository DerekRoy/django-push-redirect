# Django HTTP/2 Server Push redirects

This is a test project to utilize HTTP/2 Server Push redirects
with Django and nginx.

This approach requires nginx version 1.3.9 or later.


## Setup

This is a `docker-compose` project, to run this project download and
install [Docker Desktop](https://www.docker.com/products/docker-desktop).


### Certificates

In order to use HTTP/2 a certificate is required.

[`minica`](https://github.com/jsha/minica) is recommended
to generate a certification for the root CA and the localhostL

    minica -domains localhost

Then add the generated root CA file to the OS/browsers trust store
and move the certification for `localhost` to `./conf/certs`.

The expected filenames for the certifications are `cert.pem` and `cert.key`.


## Run the project

Once the certificates have been set up, nginx and Django can be run by using:

    docker-compose up -d
    
Visiting the following urls in a modern browser difference between "normal" redirects and HTTP/2 Server Push redirects can be seen:

* <http://localhost/hello/world> redirects to <http://localhost/hello/world/>
* <https://localhost/hello/world> redirects to and pushes <https://localhost/hello/world/>


### Code

The most important part of nginx is in `conf/nginx/default.conf`:

    server {
      ...
    
      location @python {
        ...
        http2_push_preload on;
      }
    }


The important part of Django is in `push_redirect/http.py`:

    from django.http import HttpResponsePermanentRedirect

    class Http2ServerPushPermanentRedirect(HttpResponsePermanentRedirect):
        def __init__(self, redirect_to, *args, **kwargs):
            super().__init__(redirect_to, *args, **kwargs)
            self['Link'] = f'<{self.url}>; rel=preload'

This `HttpResponsePermanentRedirect` subclass adds a second
header which is used by nginx to push the resource.

The subclass is used as a redirect class in `CommonMiddleware`.


## Inspiration

* <https://twitter.com/simonw/status/1047865898717966337>
* <https://www.ctrl.blog/entry/http2-push-redirects>
* <https://www.nginx.com/blog/nginx-1-13-9-http2-server-push/>
* <https://code.djangoproject.com/ticket/29925>
