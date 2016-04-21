---
layout: post
title: First study on WSGI and PASTE
description: ""
category: study
tags: [study,openstack,WSGI]
imagefeature:
comments: true
share: true
---


# **1. WSGI**

Basically WSGI is lower level than CGI which you probably know. But in difference to CGI, WSGI does scale and can work in both multithreaded and multi process environments because it's a specification that doesn't mind how it's implemented.

For me, the python WSGI is like a powerful tool that working between web server('gateway') and python methods('application'). I cut and paste the Introduction of WSGI from **Python Web Server Gateway Interface** [http://cloud.centos.org/centos/7/images/](http://cloud.centos.org/centos/7/images/)

The server('gateway') side invokes a callable object that is provided by the application side. The specifics of how that object is provided are up to the server or gateway. It is assumed that some servers or gateways will require an application's deployer to write a short script to create an instance of the server or gateway, and supply it with the application object. Other servers and gateways may use configuration files or other mechanisms to specify where an application object should be imported from, or otherwise obtained.

In addition to "pure" servers/gateways and applications/frameworks, it is also possible to create "middleware" components that implement both sides of this specification. Such components act as an application to their containing server, and as a server to a contained application, and can be used to provide extended APIs, content transformation, navigation, and other useful functions.

Throughout this specification, we will use the term "a callable" to mean "a function, method, class, or an instance with a __call__ method". It is up to the server, gateway, or application implementing the callable to choose the appropriate implementation technique for their needs. Conversely, a server, gateway, or application that is invoking a callable must not have any dependency on what kind of callable was provided to it. Callables are only to be called, not introspected upon.

## 1.2. The Application/Framework Side
The application object is simply a callable object that accepts two arguments. The term "object" should not be misconstrued as requiring an actual object instance: a function, method, class, or instance with a __call__ method are all acceptable for use as an application object. Application objects must be able to be invoked more than once, as virtually all servers/gateways (other than CGI) will make such repeated requests.

(Note: although we refer to it as an "application" object, this should not be construed to mean that application developers will use WSGI as a web programming API! It is assumed that application developers will continue to use existing, high-level framework services to develop their applications. WSGI is a tool for framework and server developers, and is not intended to directly support application developers.)

~~~ python
def simple_app(environ, start_response):
    """Simplest possible application object"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']

~~~

## 1.3. The Server/Gateway Side


The server or gateway invokes the application callable once for each request it receives from an HTTP client, that is directed at the application. To illustrate, here is a simple CGI gateway, implemented as a function taking an application object. Note that this simple example has limited error handling, because by default an uncaught exception will be dumped to sys.stderr and logged by the web server.

~~~ python
import os, sys

def run_with_cgi(application):

    environ = dict(os.environ.items())
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    def write(data):
        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             status, response_headers = headers_sent[:] = headers_set
             sys.stdout.write('Status: %s\r\n' % status)
             for header in response_headers:
                 sys.stdout.write('%s: %s\r\n' % header)
             sys.stdout.write('\r\n')

        sys.stdout.write(data)
        sys.stdout.flush()

    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[0], exc_info[1], exc_info[2]
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]
        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()

~~~
## 1.4 Middleware: Components that Play Both Sides
Note that a single object may play the role of a server with respect to some application(s), while also acting as an application with respect to some server(s). Such "middleware" components can perform such functions as:

   * Routing a request to different application objects based on the target URL, after rewriting the environ accordingly.
   * Allowing multiple applications or frameworks to run side-by-side in the same process
   * Load balancing and remote processing, by forwarding requests and responses over a network
   * Perform content postprocessing, such as applying XSL stylesheets

The presence of middleware in general is transparent to both the "server/gateway" and the "application/framework" sides of the interface, and should require no special support. A user who desires to incorporate middleware into an application simply provides the middleware component to the server, as if it were an application, and configures the middleware component to invoke the application, as if the middleware component were a server. Of course, the "application" that the middleware wraps may in fact be another middleware component wrapping another application, and so on, creating what is referred to as a "middleware stack".

For the most part, middleware must conform to the restrictions and requirements of both the server and application sides of WSGI. In some cases, however, requirements for middleware are more stringent than for a "pure" server or application, and these points will be noted in the specification.

Here is a (tongue-in-cheek) example of a middleware component that converts text/plain responses to pig latin, using Joe Strout's piglatin.py . (Note: a "real" middleware component would probably use a more robust way of checking the content type, and should also check for a content encoding. Also, this simple example ignores the possibility that a word might be split across a block boundary.)

~~~python
from piglatin import piglatin

class LatinIter:

    """Transform iterated output to piglatin, if it's okay to do so

    Note that the "okayness" can change until the application yields
    its first non-empty string, so 'transform_ok' has to be a mutable
    truth value.
    """

    def __init__(self, result, transform_ok):
        if hasattr(result, 'close'):
            self.close = result.close
        self._next = iter(result).next
        self.transform_ok = transform_ok

    def __iter__(self):
        return self

    def next(self):
        if self.transform_ok:
            return piglatin(self._next())
        else:
            return self._next()

class Latinator:

    # by default, don't transform output
    transform = False

    def __init__(self, application):
        self.application = application

    def __call__(self, environ, start_response):

        transform_ok = []

        def start_latin(status, response_headers, exc_info=None):

            # Reset ok flag, in case this is a repeat call
            del transform_ok[:]

            for name, value in response_headers:
                if name.lower() == 'content-type' and value == 'text/plain':
                    transform_ok.append(True)
                    # Strip content-length if present, else it'll be wrong
                    response_headers = [(name, value)
                        for name, value in response_headers
                            if name.lower() != 'content-length'
                    ]
                    break

            write = start_response(status, response_headers, exc_info)

            if transform_ok:
                def write_latin(data):
                    write(piglatin(data))
                return write_latin
            else:
                return write

        return LatinIter(self.application(environ, start_latin), transform_ok)


# Run foo_app under a Latinator's control, using the example CGI gateway
from foo_app import foo_app
run_with_cgi(Latinator(foo_app))
~~~
# 2. PASTE and PASTE Deployer
Pyton PASTE is known as 'a framework for web frameworks', it wrapps WSGI to make it easy to manage and use.  It includes CGI a simple Web server that can generate WSGI request. Paste has seperates to 3 packages:

  * Webob：wrapping request environment in WSGI
  * Paste Deploy： Deploy the WSGI server from configuration file
  * Paste Script, WebTest, ScriptType, INITools, Tempita, WaitForIt, WPHP, WSGIFilter, WSGIProxy。


In PASTE Deploy
* paste.deploy.loadwsgi — — Load wsgi from configure file
* paste.deploy.config — — configure
* paste.deploy.converters — — Convert string configuration

A config file has different sections. The only sections Paste Deploy cares about have prefixes, like app:main or filter:errors – the part after the : is the “name” of the section, and the part before gives the “type”. Other sections are ignored.

The format is a simple INI format: name = value. You can extend the value by indenting subsequent lines. # is a comment.

Typically you have one or two sections, named “main”: an application section ([app:main]) and a server section ([server:main]). [composite:...] signifies something that dispatches to multiple applications
[app:main]:define the name of the WSGI application, 'main' is the name of the given application.

* config: another_config_file.ini#app_name == find the name in other configuration file.
* egg: MyApp --> find tMyApp in egg 。
* call: my.project --> myapplication call the method directly
* use = myotherapp --->in section 'myotherapp' to call app
* user_names = messages  --> pass the user define parameter to app.

## 2.1 PASTE Deploy Examples
Firstly, this is the config.ini file:



~~~ini
[app:home]
paste.app_factory = apps:MyHome.app_factory
;define the application name as home, if this application is called, it will find method in app.py-->class Myhome to invoke
[app:home_use]
use=call:apps:MyHome_USE
;Directly call Myhome object in apps.py. The same as paste.app_factory = apps:MyHome.app_factory
[composite:pub]
;depatch the resquest according to the url. We use egg.Paster#urlmap to depatch different requests to different application according to the URL.
use = egg:Paste#urlmap
/: pub0
;root will be dipatched to the application called pub0
/V1:pub1
/V2:pub2

[app:pub0]
paste.app_factory = apps:Pub0.app_factory
;This is how pub0 handle the request


[filter-app:pub1]
paste.filter_factory = apps:PubFilter.factory
next = subpub
;we use filter-app to filter the requests, the requests which pass the filter will hand over to subpub.


[app:subpub]
paste.app_factory = apps:Pub1.app_factory

[pipeline:pub2]
pipeline = logip logmethod pubv2
;we use pipeline to pass a serial filters
[filter:logip]
paste.filter_factory = apps:LogIPFilter.factory

[filter:logmethod]
paste.filter_factory = apps:LogMethod.factory

[app:pubv2]
paste.app_factory = apps:Pub2.app_factory
#This is an example of passing user_define arguemnts to application
para1 = This is first
para2 = This is second
~~~
Then this is the apps.py source code

~~~python
import os
import eventlet
from eventlet import wsgi, listen
from paste import deploy
from webob import Request
cfg_file='myconfig.ini'
server_list = [('home', 8001),('home_use',8002),('pub',8003)]
# We define 3 serviers here as home home_use and pub. The server can be found in [app:names]


class Pub0(object):
	#The build-in function make the object can be called like a function.
    def __call__(self, environ, start_response):
        start_response('200 OK', {("Content-type", "text/plain")})
        return 'Hello from PUB0\n'
	#cls will be assigned to the object of Pub0 when it is called
    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        return cls()



class Pub1(object):
    def __call__(self, environ, start_response):
        start_response('200 OK', {("Content-type", "text/plain")})
        return 'Hello from PUB1\n'

    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        return cls()



class Middleware(object):
    def __init__(self, app):
        self.app = app

    @classmethod
    def factory(cls, global_conf, **kwargs):
        def filter(app):
            return cls(app)
        return filter

class PubFilter(Middleware):
    #we define the middleware filters
    def __init__(self, app):
        super(PubFilter, self).__init__(app)

    def __call__(self, environ, start_response):
        req = Request(environ)
        if req.method == 'POST':
            start_response('200 OK', {("Content-type", "text/plain")})
            return 'Bad request\n'
        else:
            return self.app(environ, start_response)


    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        return cls()


class Pub2(object):
    def __call__(self, environ, start_response):
        start_response('200 OK', {("Content-type", "text/plain")})
        return 'Hello from PUB2\n'

    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        print "we print thr parameters from ini file: %s" % local_conf
        return cls()


class LogIPFilter(Middleware):
    def __init__(self, app):
        super(LogIPFilter, self).__init__(app)

    def __call__(self, environ, start_response):
        print 'request IP is: %s' % environ['REMOTE_ADDR']
        return self.app(environ, start_response)


class LogMethod(Middleware):
    def __init__(self, app):
        super(LogMethod, self).__init__(app)

    def __call__(self, environ, start_response):
        print 'Method is: %s' % environ['REQUEST_METHOD']
        return self.app(environ, start_response)


class MyHome(object):

    def __call__(self, environ, start_response):
        start_response('200 OK', {("Content-type", "text/plain")})
        return 'Hello from MyHome\n'
    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        return cls()


class MyHome_USE(object):

    def __call__(self, environ, start_response):
        start_response('200 OK', {("Content-type", "text/plain")})
        return 'Hello from MyHome with USE\n'
    @classmethod
    def app_factory(cls, global_conf, **local_conf):
        return cls()

MyHome_USE=MyHome_USE.app_factory

if __name__ == '__main__':
    host = '127.0.0.1'
    servers = []
    for app_name, port in server_list:
        socket = listen((host, port))
        app = deploy.loadapp('config:%s' % os.path.abspath(cfg_file), app_name)

        print "%s is starting" % app_name
        servers.append(eventlet.spawn(wsgi.server, socket, app))

    for server in servers:
        server.wait()
~~~

## 2.2 TESTS
~~~
yb@yb-ThinkPad-T440p:~$ curl http://127.0.0.1:8001
Hello from MyHome
~~~

~~~
yb@yb-ThinkPad-T440p:~$ curl http://127.0.0.1:8001
Hello from MyHome
~~~

~~~
yb@yb-ThinkPad-T440p:~$ curl http://127.0.0.1:8001
Hello from MyHome
~~~


~~~
yb@yb-ThinkPad-T440p:~$ curl http://127.0.0.1:8001
Hello from MyHome

~~~

~~~
yb@yb-ThinkPad-T440p:~$ curl -d "anything" http://127.0.0.1:8003/V1
Bad request
~~~
~~~
yb@yb-ThinkPad-T440p:~$ curl -d "anything" http://127.0.0.1:8003/V1
Bad request
~~~

~~~
yb@yb-ThinkPad-T440p:~$ curl http://127.0.0.1:8003/V2
Hello from PUB2

request IP is: 127.0.0.1
Method is: GET

~~~
