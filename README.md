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

