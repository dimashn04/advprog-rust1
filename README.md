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

## Commit 3  
I've added a ```404.html``` file:  
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>
<body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
    <p>Rust is running from Dimas's machine.</p>
</body>
</html>
```  

And from the given refference from the instruction in Chapter 20, part: Validating the Request and Selectively Responding, I've modified my ```handle_connection``` function to this:  
```rust
fn handle_connection(mut stream: TcpStream) { 
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );
        stream.write_all(response.as_bytes()).unwrap();
    }
}
```  
The modified ```handle_connection``` function acts as a simple HTTP server. It reads the request line from the client's HTTP request and checks if it's a ```GET / HTTP/1.1``` request, indicating a request for the root path (```/```). Based on this check, it responds with different content:  
    1. If the request is for the root path (```GET / HTTP/1.1```), it responds with a status line ```HTTP/1.1 200 OK```, reads the content of a file named ```hello.html```, calculates its length, constructs an HTTP response containing this content, and sends it back to the client.  
    2. If the request is not for the root path (indicating that the requested resource is not found), it responds with a status line ```HTTP/1.1 404 NOT FOUND```, reads the content of a file named ```404.html```, calculates its length, constructs an HTTP response containing this content, and sends it back to the client.  
In summary, this modified ```handle_connection``` function serves different content based on the request. If the request is for the root path, it serves ```hello.html```, otherwise, it serves a ```404.html``` page indicating that the requested resource is not found.  

I've also refactor the ```handle_connection``` function using ternary operations so there are no duplicates:  
```rust
fn handle_connection(mut stream: TcpStream) { 
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );
    stream.write_all(response.as_bytes()).unwrap();
} 
```  
![Commit 3 screen capture](/assets/images/commit3.png)  

## Commit 4  
In this modified ```handle_connection``` function, additional functionality is introduced to handle a specific type of request, namely ```GET /sleep HTTP/1.1```. Here's what the modification does:  
    1. It still reads the request line from the client's HTTP request as before.  
    2. Instead of directly checking the request line, it uses pattern matching (match) to match against different types of request lines.  
    3. If the request line matches ```GET / HTTP/1.1```, indicating a request for the root path (```/```), it sets the status_line to ```HTTP/1.1 200 OK``` and filename to ```hello.html```.  
    4. If the request line matches ```GET /sleep HTTP/1.1```, indicating a request to sleep for a certain period before responding, it introduces a delay of 10 seconds using ```thread::sleep(Duration::from_secs(10))```. After sleeping, it sets the status_line to ```HTTP/1.1 200 OK``` and filename to ```hello.html```.  
    5. If the request line doesn't match any of the predefined patterns, it sets status_line to ```HTTP/1.1 404 NOT FOUND``` and filename to ```404.html```.  
    6. After determining the appropriate ```status_line``` and filename, it reads the content of the file indicated by filename using ```fs::read_to_string()```.  
    7. It calculates the length of the content string.  
    8. It constructs an HTTP response containing the determined status_line, length, and contents.  
    9. Finally, it writes the constructed HTTP response back to the client over the TCP stream.  
In summary, this modification extends the functionality of the server to handle a special case where if the client requests ```GET /sleep HTTP/1.1```, the server sleeps for 10 seconds before responding with the content of ```hello.html```. Otherwise, it behaves as before, serving either ```hello.html``` for root requests or ```404.html``` for requests to non-existent resources.  

## Commit 5  
### How does ```ThreadPool``` work?  
Here's how it works:  
    1. ThreadPool Initialization (```ThreadPool::new```):  
        - It initializes a new thread pool with a specified size. The size determines the number of worker threads in the pool.  
        - It creates a channel (```mpsc::channel```) for communication between the main thread (which submits jobs) and the worker threads.  
        - It creates a vector to hold the worker threads.  
        - For each worker thread, it creates a new Worker struct, passing it the ID and a clone of the receiver end of the channel.  
    2. Worker Creation (```Worker::new```):  
        - Each worker thread is created with a closure that runs in an infinite loop.  
        - Inside the loop, the worker waits for a job to be sent over the channel.  
        - Once a job is received, the worker executes it.  
    3. Job Execution (```ThreadPool::execute```):  
        - To execute a job, the main thread places the job inside a box and sends it over the channel to one of the worker threads.  
        - The job is then executed by the worker thread.  
    4. Concurrency and Communication:  
        - The ```Arc<Mutex<mpsc::Receiver<Job>>>``` is shared among all worker threads. This allows multiple threads to access the receiver end of the channel safely.  
        - The ```Mutex``` ensures that only one worker thread can receive a job at a time to prevent data races.  
        - The ```mpsc``` (multi-producer, single-consumer) channel allows multiple threads to send jobs to worker threads concurrently while ensuring that each job is only processed by one worker.  
In summary, the thread pool manages a fixed number of worker threads. Jobs are submitted to the pool and executed concurrently by the worker threads, providing a simple mechanism for parallel execution of tasks.  

## Commit 6  
https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html suggests using the ```build``` method instead of ```new``` when initializing the ```ThreadPool```, arguing that ```new``` may cause an error if the number of threads given is too small. However, this argument is incorrect because the expectation of the ```new``` method is success. Therefore, it is recommended to replace the ```new``` method with ```build```, which returns ```Result```. Then, when the value is returned to the caller, it can be unwrapped to get the value of the execution result.  