# Introduction
Hello, I'm Nathaniel Petersen, PID A17832207, and this page will explore the webserver "ChatServer", SSH keys and some personal insights I've gained from the week 2 and 3 CSE15L labs.

# Part 1 - ChatServer
## Code for ChatServer

Server.java - Direct copy of Server.java from [Link](https://github.com/ucsd-cse15l-s24/wavelet) repo
```
// A simple web server using Java's built-in HttpServer

// Examples from https://dzone.com/articles/simple-http-server-in-java were useful references

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
import java.net.URI;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

interface URLHandler {
    String handleRequest(URI url);
}

class ServerHttpHandler implements HttpHandler {
    URLHandler handler;
    ServerHttpHandler(URLHandler handler) {
      this.handler = handler;
    }
    public void handle(final HttpExchange exchange) throws IOException {
        // form return body after being handled by program
        try {
            String ret = handler.handleRequest(exchange.getRequestURI());
            // form the return string and write it on the browser
            exchange.sendResponseHeaders(200, ret.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(ret.getBytes());
            os.close();
        } catch(Exception e) {
            String response = e.toString();
            exchange.sendResponseHeaders(500, response.getBytes().length);
            OutputStream os = exchange.getResponseBody();
            os.write(response.getBytes());
            os.close();
        }
    }
}

public class Server {
    public static void start(int port, URLHandler handler) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        //create request entrypoint
        server.createContext("/", new ServerHttpHandler(handler));

        //start the server
        server.start();
        System.out.println("Server Started! If on your local computer, visit http://localhost:" + port + " to visit.");
    }
}
```

ChatServer.java - Modified version of NumberServer.java from [Link](https://github.com/ucsd-cse15l-s24/wavelet) repo
```
import java.io.IOException;
import java.net.URI;

class Handler implements URLHandler {
    
    String messages = "";

    public String handleRequest(URI url) {
        if (url.getPath().equals("/")) {
            return messages;
        }
        else if(url.getPath().equals("/remove")){
            messages = "";
            return "Messages deleted";
        } 
        else {
            if (url.getPath().contains("/add-message")) {
                String[] parameters = url.getQuery().split("&");

                if (parameters.length!=2){
                    return "404 Not Found";
                }

                String[] message = parameters[0].split("=");

                String[] user = parameters[1].split("=");

                if((message[0].equals("s") && user[0].equals("user")) != true){
                    return "Please give message requests in the format of /add-message?s=<your message>&user=<your name>";
                }

                messages += user[1] + ": " + message[1] + "\n";
                return messages;
            }
            return "404 Not Found!";
        }
    }
}

class ChatServer {
    public static void main(String[] args) throws IOException {
        if(args.length == 0){
            System.out.println("Missing port number! Try any number between 1024 to 49151");
            return;
        }

        int port = Integer.parseInt(args[0]);

        Server.start(port, new Handler());
    }
}
```

##First example of /add-message use
![Image](example2.jpg)
![Image](example1.jpg)

In this example when the URL "http/localhost:4000/add-message?s=Joseph Robinet Biden&user=My Name Is" is put into the hotbar, first the handle method of ServerHttpHandler is run, which immediately attempts to the string variable ret to whatever String is returned by `handler.handleRequest(exchange.getRequestURI())`. This block of code calls upon the `handleRequest` method which is found in ChatServer.java. The argument supplied to this method is `exchange.getRequestURI()`, which calls upon the `getRequestURI()` method on exchange which is equal to the URL from earlier. The `URI url` parameter for `handleRequest` is therefore set to the URL from earlier. The `getPath()` method is then run several times on `url` until the url is found to contain `"/add-message"` through the `contains` method. After this, the variable `String[] parameters` is initialized and using the `getQuery()` and `split` methods on the variable `url`, the string array ends up containing the strings `"s=Joseph Robinet Biden"` and `"user=My Name Is"`. Each of these strings are then split into two and stored in the `message` and `user` string arrays so that `message` contains the strings `"s"` and `"Joseph Robinet Biden"` and `user` contains the strings `"user"` and `"My Name Is"`. After a quick test to ensure that `message` contains `"s"` in its 0th index and `user` contains `"user"` in its 0th index using the equals method, we then update the variable `messages` which was initialized to `""` at the creation of the server to `"My Name Is: Joseph Robinet Biden"\n`. This string is returned so that the variable `ret` in the `handle` method of Server.java is set equal to `"My Name Is: Joseph Robinet Biden"` as opposed to `""`. The method `sendResponseHeaders` is then called as `exchange.sendResponseHeaders(200, ret.getBytes().length);`, which sends a response back to the user on the webpage. We then upate the webpage with the variable `OutputStream os` and calling the methods `getResponseBody()`, `write(ret.getBytes())`, and `close()` to update the web server with the information contained in `ret`, updating the web server to include the message "My Name Is: Joseph Robinet Biden".


##Second example of /add-message use
![Image](example3.jpg)
![Image](example4.jpg)

In this example when the URL "http/localhost:4000/add-message?s=no%20your%20not&user=john" is put into the hotbar, first the handle method of ServerHttpHandler is run, which immediately attempts to the string variable ret to whatever String is returned by `handler.handleRequest(exchange.getRequestURI())`. This block of code calls upon the `handleRequest` method which is found in ChatServer.java. The argument supplied to this method is `exchange.getRequestURI()`, which calls upon the `getRequestURI()` method on exchange which is equal to the URL from earlier. The `URI url` parameter for `handleRequest` is therefore set to the URL from earlier. The `getPath()` method is then run several times on `url` until the url is found to contain `"/add-message"` through the `contains` method. After this, the variable `String[] parameters` is initialized and using the `getQuery()` and `split` methods on the variable `url`, the string array ends up containing the strings `"s=no your not"` and `"user=john"`. Each of these strings are then split into two and stored in the `message` and `user` string arrays so that `message` contains the strings `"s"` and `"no your not"` and `user` contains the strings `"user"` and `"john"`. After a quick test to ensure that `message` contains `"s"` in its 0th index and `user` contains `"user"` in its 0th index using the equals method, we then update the variable `messages` which was previously set to `"Luigi: Hi im john\n"` at the creation of the server to `"Luigi: Hi im john\njohn: no your not"`. This string is returned so that the variable `ret` in the `handle` method of Server.java is set equal to `"Luigi: Hi im john\njohn: no your not"` as opposed to `"Luigi: Hi im john\n"`. The method `sendResponseHeaders` is then called as `exchange.sendResponseHeaders(200, ret.getBytes().length);`, which sends a response back to the user on the webpage. We then upate the webpage with the variable `OutputStream os` and calling the methods `getResponseBody()`, `write(ret.getBytes())`, and `close()` to update the web server with the information contained in `ret`, updating the web server to include the message "john: no your not" on a new line.

![Image](example5.jpg)
![Image](example6.jpg)
![Image](example7.jpg)





