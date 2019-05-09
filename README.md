# bottle-tornado-hybrid
Hybrid approach for existing wsgi python bottle application to use asyncio tornado handlers for websocket

#Context

I have worked number of bottle applications using various backends (cherrypy, gunicorn, tornado). 

Some application would like to support web-socket.

This example explores how you can directly add TornadoWebSocketHandlers to the underlying TornadoServer whilst still supporting an existing application that is written using bottle.

![screenshot](https://github.com/jburrow/bottle-tornado-hybrid/blob/master/docs/bottle-tornado-websocket.png?raw=true)
