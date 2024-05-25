# streaming-04-multiple-consumers

## Overview

This project uses RabbitMQ to distribute tasks to multiple workers using a single producer. It provides an overview of three different versions of emitter and listener files, each building in complexity and standard conventions for ease-of-use. The project ends with a custom v3 emitter file that reads tasks from a tasks.csv file and distributes them to multiple active v3_listening_worker files.

## Author

**Derek Graves**  
**Date:** May 24, 2024

## Before You Begin

1. Fork this starter repo into your GitHub.
2. Clone your repo down to your machine.
3. View / Command Palette - then Python: Select Interpreter
4. Select your conda environment. 

## Setup

### Virtual Environment

1. Create a virtual environment:
```bash
py -m venv .venv
```
2. Activate the environment:
```bash
source .venv/Scripts/activate
```
3.  Install required external packages:
```bash
pip install pika
```
or you may choose to install from a requirements.txt file:

```bash
pip install -r requirements.txt
```

### RabbitMQ Installation:
- Follow instructions on https://www.rabbitmq.com/tutorials to learn about installation.
- Ensure the RabbitMQ server is running on your maching once installed.
- Read the [RabbitMQ Tutorial - Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)

### Project Files

1. Ensure tasks.csv is populated with tasks for version 3 files.  Adding a . at the end of each task will make it take longer to complete, whereas removing them will make processing the tasks faster. 

```bash
First task....
Second task....
Third task........
Forth task...
Fifth task...
```

2. Version descriptions:

- Version 1: Sends single tasks directly from the command line.
- Version 2: Improved code organization for easier maintenance. Still sends single tasks directly from the command line.
- Version 3: Reads tasks from a tasks.csv file. Utilizes a logging utility (util_logger.py) for better log management.

### RabbitMQ Admin 

RabbitMQ comes with an admin panel. When you run the task emitter, reply y to open it. 

(Python makes it easy to open a web page - see the code to learn how.)

## Execute the Producer

1. Run emitter_of_tasks.py (say y to monitor RabbitMQ queues if you wish.)  Versions 1 and 2 will only send one message at a time as they are not reading from tasks.csv.  
    - This portion of the code allows messages to be send directly from the command line (emitter): 
```bash
message = " ".join(sys.argv[1:]) or "Second task....."
```

2.  Version 3 emitter files will use the following code to read tasks from tasks.csv instead:
```bash
def read_tasks_from_csv(filename='tasks.csv'):
    """Read tasks from a CSV file and return a list of tasks"""
    tasks = []
    try:
        with open(filename, newline='') as csvfile:
            task_reader = csv.reader(csvfile)
            for row in task_reader:
                task = ' '.join(row)
                tasks.append(task)
    except FileNotFoundError:
        logger.error(f"File {filename} not found.")
        sys.exit(1)
    return tasks
```
- Version 3 files also make use of the logging utility instead of print statements. 
- Producers will terminate on their own (or once they run out of tasks to send e.g. once they finish sending all tasks in tasks.csv.)

## Execute a Consumer / Worker

1. Run listening_worker.py. It will not terminate on its own (as you can see from the produced message) and instead will prompt the user to hit ctrl+c to terminate the connection. Otherwise, it will continue to listen. Version 3 is the most streamlined for efficiency and also uses logging. Multiple consumers can be started by running this script in multiple terminals.
2. Once a task is complete, an acknowledgement message will display (such as "Done") advising the task has been processed and the listener is ready for another task from the queue. 

## Setting up Multiple Anaconda Terminals

1.  Open Anaconda Prompt from the start menu or applications folder.
2.  Once open, use the following commands to navigate to the root directory for the project and activate the virtual environment in the conda terminal:
```bash
cd Documents/streaming-03-rabbitmq
dir  # This will display project folders for easy copy/pasting
conda activate base
.venv\Scripts\activate # Activate virtual environment- conda will show .venv next to command line. 
```

From there, the terminals have been set up and are ready to execute files.  The general workflow would look like: 
1. Run python v3_listening_worker.py in a conda terminal.  For subsequent worker activation, open a new conda window and set it up using the commands above. Run python v3_listening_worker.py again in the new terminal.  
2. Run v3_emitter_of_tasks.  It will begin to send messages to the queue name indicated in the file, and listeners will begin to receive and process the messages one at a time from that queue.  If a worker is busy, a message will be sent to the next available open worker.  Once complete, an acknowledgement message like "Done" will appear in the console, indicating the task is complete and the worker is ready for the next task in the queue. 

## Reference

The original repository and inspiration for this project can be found at: 
- https://github.com/denisecase/streaming-04-multiple-consumers

- [RabbitMQ Tutorial - Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)


## Screenshot

See a running example with at least 3 concurrent process windows here:

![Screenshot](images/EmittertoMultiListeners%202024-05-24%20155857.png)