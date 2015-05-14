Web Socks is an HTML5 Web Socket server written in pure PHP.  It implements the version 76 handshake and is compatible with later versions of Chromium.  There is a simple OO API which aims to make writing "stateful" connection handlers easy, the gubbins of handling the lower level TCP connection is hidden and an event based API is used to deliver messages to and from remote clients.

Because the Web Socket protocol is currently still in heavy development I've kept the server as simple as possible to make implementing future changes as easy as possible.  I expect to be able to handle the upcoming new framing features and even Multiplexing (if that ever happens).  My plan is to track new features as they are included in Chromium and/or Firefox.

Here's an example chat client handler

```
class SimpleChatClientHandler extends WsClient
{
    private static $Chatters = array();
    private $myPos;

    function onConnect() {
        $this->myPos = array_push(self::$Chatters, $this) - 1;
        $this->info("[SChat] client connected");
    }
    function onHandshake(WebSocketHandshake $hs, $errNo = 0) {
        $this->info("[SChat] client handshake complete");
    }

    function onRead($buff, $errNo = 0) {
        $this->info("[SChat] client is delivered %d bytes", strlen($buff));
        foreach (self::$Chatters as $chatee) {
            if ($chatee !== $this) {
                // Deliver this client's message to all others
                $chatee->write($buff);
            }
        }
    }

    function onWrite($bw, $errNo = 0) {
        $this->info("[SChat] client, %d bytes written to network", $bw);
    }

    function onClose($errNo = 0) {
        unset(self::$Chatters[$this->myPos]);
        $this->info("[SChat] Bye-bye client ;-)");
    }
}
```

Please note this script requires php 5.3+