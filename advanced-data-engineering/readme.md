# Key Terms

- **Queue**: A place where items of work are stored and managed. Items of work represent units of tasks or actions that need to be processed.

- **First In, First Out (FIFO)**: A pattern of behavior for queues where the first item added to the queue is the first one to be processed.

- **Last In, First Out (LIFO)**: A pattern of behavior for queues where the last item added to thequeue is the first one to be processed.

- **Producer**: The entity responsible for generating and adding units of work to the queue.

- **Consumer**: The entity responsible for processing and removing units of work from the queue.

- **Broker/Message broker**: A separate process used to connect Python applications to message queues such as RabbitMQ.

- **Celery**: A middleman package in Python that allows producing items of work in Python and putting them somewhere else, such as RabbitMQ. An asynchronous task queue based on distributed message passing. Allows you to execute tasks asynchronously outside of the request-response cycle. Integrates well with Flask for building asynchronous web applications.

- **AMQP**: Advanced Message Queuing Protocol, a protocol for messaging middleware used by RabbitMQ.

- **Task queue**: Represents a specific type of message queue designed to handle tasks or units of work.

- **Concurrency**: The ability for multiple operations to execute simultaneously.

- **Serialization**: The process of converting complex data types into simpler formats for storage or transmission.

- **Image processing**: A use case for queues involving resizing images and creating down samples at large scales.

- **Publish/Subscribe (Pub/Sub)**: An architectural pattern used in message queuing systems allowing multiple entities to communicate by publishing messages to a queue and subscribing to receive those messages.

- **Request-Reply Pattern**: A communication model between two parties where one party sends a request and expects a response from the second party.

- **Flask**: A lightweight web framework for Python used for building web applications. A popular Python web framework. Used to build the web application and endpoints. Integrates with Celery to offload tasks to worker processes.

- **requirements.txt**: A file used in Python to declare necessary libraries and packages required for running a specific application or program.

- **Units of work**: Represent individual tasks or actions that need to be processed. These could be data, sending emails, or any other type of action.

- **RabbitMQ**: An open source message broker that implements the Advanced Message Queuing Protocol (AMQP). Used by Celery to pass messages between clients and workers for processing tasks asynchronously.

- **Task**: A unit of work in Celery that gets executed asynchronously. Typically invoked on a route or endpoint and then handed off to a worker process to execute.

- **Worker**: A process that Celery starts to execute tasks that are sent through the message queue. Pulls tasks from the queue and executes them. Can run asynchronously in the background.

- **Queue**: A queue that Celery uses to pass messages between clients and workers. Celery uses RabbitMQ and AMQP to implement the queue. Messages represent tasks to be executed.

- **Shared task**: A convenience Celery decorator to create tasks. Automatically connects to a broker and backend based on configuration. Good for simple use cases.

- **Delay**: A Celery method to introduce artificial delays when executing tasks. Used to demonstrate asynchronous behavior since tasks return immediately.

## MySQL Commands
**Export**: Save MySQL data externally. Useful for external processing or backup.
```
# Export table data to a file 
cursor.execute("SELECT * FROM table INTO OUTFILE 'data.csv'")
```

**Import**: Load external data into MySQL like a file or dump. Quick way to ingest data.
```
# Import data from a CSV file
cursor.execute("LOAD DATA LOCAL INFILE 'data.csv' INTO TABLE mytable")
```

**Piping**: Redirect the output of one command to another command. Integrates MySQL and bash.
```
# MySQL query output to grep  
mysql -e "SELECT * FROM users WHERE name LIKE '%John%'" | grep "@gmail"
```

**Batch Processing**: Chain together commands for sequential data processing without code
```
# Query, extract names, count records
mysql -e "SELECT name FROM users" | cut -d, -f1 | wc -l
```

**Parallel Pipelines**: Split data across commands simultaneously to improve performance.
```
# Search 2 columns in parallel 
mysql -e "SELECT name,email FROM users" | parallel --pipe grep {} ::: John ::: gmail
```

# Hacking MySQL

## Setup 
Setup Database
```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'dbpassword';
CREATE DATABASE mydemo;
USE mydemo;
```
Create Table and Insert Data
```mysql
CREATE TABLE users(
                      id INT AUTO_INCREMENT PRIMARY KEY,
                      name VARCHAR(50),
                      email VARCHAR(50)
);
INSERT INTO users (name, email)
VALUES ('John', 'john@gmail.com');

INSERT INTO users (name, email)
VALUES ('Jane', 'jane@yahoo.com');
```

Query Data
```mysql
SELECT * FROM users;
```

## Extend

Extending a directors_notes Table.

```mysql
CREATE TABLE directors_notes (
note_id INT AUTO_INCREMENT PRIMARY KEY,
film_id INT,
director_note VARCHAR(255),
edit_date DATE
);
```

Insert
```mysql

INSERT INTO directors_notes (film_id, director_note, edit_date)
VALUES (1, 'Fix sound in minute 20', '2023-02-01');

INSERT INTO directors_notes (film_id, director_note, edit_date)
VALUES (2, 'Shorten battle scene', '2023-02-03');
```

Select
```mysql
SELECT f.title, n.director_note, n.edit_date
FROM film f
LEFT JOIN directors_notes n
ON n.film_id = f.film_id;

DROP TABLE directors_notes;
```

## Export to Create Python Web Service

```mysql
SELECT * FROM sakila.actor 
INTO OUTFILE '/tmp/actors.csv' 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

```python
import csv 
from http.server import HTTPServer, BaseHTTPRequestHandler

class HTTPHandler(BaseHTTPRequestHandler):

    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        with open('/tmp/actors.csv') as f:
            reader = csv.reader(f)
            self.wfile.write(b"\n".join([','.join(row).encode() for row in reader]))

httpd = HTTPServer(('localhost', 8081), HTTPHandler)
httpd.serve_forever()
```