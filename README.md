# bottle-tornado-hybrid
Hybrid approach for existing wsgi python bottle application to use asyncio tornado handlers for websocket

#Context

I have worked number of bottle applications using various backends (cherrypy, gunicorn, tornado). 

Some application would like to support web-socket.

This example explores how you can directly add TornadoWebSocketHandlers to the underlying TornadoServer whilst still supporting an existing application that is written using bottle.

![screenshot](https://github.com/jburrow/bottle-tornado-hybrid/blob/master/docs/bottle-tornado-websocket.PNG?raw=true)

```python

import bottle
import tornado
import tornado.websocket
import tornado.web

@bottle.route('/hello/<name>')
def index(name):
    return 'hello' + name

class WSHandler(tornado.websocket.WebSocketHandler):
    def open(self):
        print('new connection')
        self.write_message("Hello World")
      
    def on_message(self, message):
        print('message received %s' % message)
        self.write_message("Hello World")
 
    def on_close(self):
        print( 'connection closed' )

class HelloHandler(tornado.web.RequestHandler):
    def get(self):
        self.write('Hello from tornado')

class TornadoServerWithHandlers(bottle.ServerAdapter):
    def run(self, bottle_app, tornado_handler_routes=None):
        import tornado.wsgi
        import tornado.httpserver
        import tornado.ioloop

        wsgi_app = tornado.wsgi.WSGIContainer(bottle_app)

        tornado_handler_routes = list( tornado_handler_routes or [] )
        tornado_handler_routes.append(('.*', tornado.web.FallbackHandler, dict(fallback=wsgi_app)))
        
        tornado_app = tornado.web.Application(tornado_handler_routes)
        server = tornado.httpserver.HTTPServer(tornado_app)
        server.listen(self.port,address=self.host)
        tornado.ioloop.IOLoop.instance().start()

if __name__ == "__main__":
    tornado_handler_routes=[('/hello-tornado', HelloHandler),
                            ('/ws',WSHandler),]
    bottle.run(host='localhost', port=8080, server=TornadoServerWithHandlers, tornado_handler_routes=tornado_handler_routes)
```