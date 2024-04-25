this is a lab report


```
import java.io.IOException;
import java.net.URI;

class Handler implements URLHandler {
    
    String messages = "";

    public String handleRequest(URI url) {
        if (url.getPath().equals("/")) {
            return messages;
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
