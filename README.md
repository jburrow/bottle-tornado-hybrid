# bottle-tornado-hybrid
Hybrid approach for existing wsgi python bottle application to use asyncio tornado handlers for websocket

# Background

I have worked number of bottle applications using various backends (cherrypy, gunicorn, tornado). 

Some application would like to support web-socket.

This example explores how you can directly add TornadoWebSocketHandlers to the underlying TornadoServer whilst still supporting an existing application that is written using bottle.

![screenshot](https://github.com/jburrow/bottle-tornado-hybrid/blob/master/docs/bottle-tornado-websocket.PNG?raw=true)

```python

import bottle
import tornado
import tornado.websocket
import tornado.web


@bottle.route("/")
def bottle_homepage():
    return """
<ul>
    <li><a href='bottle-hello'>bottle-hello</a></li>    
    <li><a href='tornado-hello'>tornado-hello</a></li>
    <li><a href='test-ws'>test-ws</a></li>
</ul>"""


@bottle.route("/bottle-hello")
def bottle_hello():
    return "Hello from bottle"


@bottle.route("/test-ws")
def bottle_test_websocket():
    return """<script>
var ws = new WebSocket('ws://localhost:8080/tornado-ws')
ws.onmessage = (msg)=>console.debug(msg);
ws.onopen = (msg)=>{console.debug(msg);ws.send('Hello Hybrid');}

    </script>
    """


class WSHandler(tornado.websocket.WebSocketHandler):
    def open(self):
        print("new connection")
        self.write_message("New Connection")

    def on_message(self, message):
        print("message received %s" % message)
        self.write_message("Hello World")

    def on_close(self):
        print("connection closed")


class TornadoHelloHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello from tornado")


class TornadoServerWithHandlers(bottle.ServerAdapter):
    def run(self, bottle_app):
        import tornado.wsgi
        import tornado.httpserver
        import tornado.ioloop

        wsgi_app = tornado.wsgi.WSGIContainer(bottle_app)

        options = self.options.get("options") or {}
        tornado_handler_routes = options.get("tornado_handler_routes") or []

        tornado_handler_routes = list(tornado_handler_routes)
        tornado_handler_routes.append(
            (".*", tornado.web.FallbackHandler, dict(fallback=wsgi_app))
        )

        tornado_app = tornado.web.Application(tornado_handler_routes)
        server = tornado.httpserver.HTTPServer(tornado_app)
        server.listen(self.port, address=self.host)
        tornado.ioloop.IOLoop.instance().start()


if __name__ == "__main__":
    tornado_handler_routes = [
        ("/tornado-hello", TornadoHelloHandler),
        ("/tornado-ws", WSHandler),
    ]
    bottle.run(
        host="localhost",
        port=8080,
        server=TornadoServerWithHandlers,
        options=dict(tornado_handler_routes=tornado_handler_routes),
    )
```