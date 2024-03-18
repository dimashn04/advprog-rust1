# REFLECTION  

## Commit 1   
### What does ```handle_connection``` do?  
The ```handle_connection``` function is responsible for processing incoming TCP streams. Here's a breakdown of what it does:  
    1. It takes a mutable reference to a ```TcpStream``` as its argument.  
    2. It creates a ```BufReader``` wrapping the given TCP stream. This helps with buffered reading from the stream efficiently.  
    3. It reads lines from the buffered reader (```BufReader```) until it reaches an empty line. This is commonly how HTTP requests are formatted, with headers followed by an empty line.  
    4. It collects these lines into a vector, representing the HTTP request.  
    5. Finally, it prints out the HTTP request for inspection.  
In summary, ```handle_connection``` reads the incoming TCP stream line by line until it encounters an empty line, indicating the end of the HTTP request headers. Then it prints out the collected lines as the HTTP request.  

## Commit 2  
### What does the additional lines of code in ```handle_connection``` do?  
The additional code in the ```handle_connection``` function modifies it to send an HTTP response back to the client. Here's what the new code does:  
    1. It defines a status_line indicating that the HTTP response is ```200 OK```. This means the request was successful.  
    2. It reads the contents of a file named ```hello.html``` into a string using ```fs::read_to_string()```. This assumes that there is a file named ```hello.html``` in the same directory as the executable.  
    3. It calculates the length of the contents string.  
    4. It formats the HTTP response, including the status line, content length, and the contents of the ```hello.html``` file.  
    5. It writes the response back to the TCP stream, sending it to the client using ```write_all()```.  
So, in summary, the modified ```handle_connection``` function reads the content of ```hello.html``` file, constructs an HTTP response with a ```200 OK``` status line, and sends it back to the client over the TCP stream.  
![Commit 2 screen capture](/assets/images/commit2.png)  

