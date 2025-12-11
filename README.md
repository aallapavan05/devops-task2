Overview

The Safe Code Executor is a secure and isolated Python code execution system built using Flask and Docker.
It allows users to send Python (and JavaScript in easy-level) code to an API, and the code is executed inside a restricted Docker container.

This setup ensures that untrusted or harmful scripts cannot affect the host system.
The main purpose of this project is to understand how to safely execute external code using Docker’s isolation and resource-control features.

work flow diagram

<img width="1536" height="1024" alt="ChatGPT Image Dec 11, 2025, 05_05_19 PM" src="https://github.com/user-attachments/assets/bcbb6107-2bad-4bc5-9e93-2ce466bdad48" /> 

<img width="449" height="572" alt="Screenshot 2025-12-11 194853" src="https://github.com/user-                                          attachments/assets/27f2023e-7c4e-4056-a1c3-c95083e1d386" />



Setup Instructions
1. Create Project Folder

I created a folder named:

safe-code-executor


All project files (app.py, Dockerfile, test scripts, etc.) were stored inside this folder.

2. Install Required Software

Before starting, I installed:

Python 3.10+

Docker Desktop

Visual Studio Code

WSL2 (for Windows)

These tools are required to build Docker images, run containers, and host the Flask API.

3. Install Python Dependencies

Inside the project folder, I installed Flask:

pip install flask

4. Run the Flask API

After the setup, I started the API using:

python app.py


This enabled the /run endpoint that receives and executes Python code safely.

5. Test the API Using Postman

URL:

http://localhost:5000/run


Body (JSON):

{ "code": "print('Hello World')" }


The output displayed:

Hello World


This confirmed that the API and Docker container execution were working correctly.

<img width="1355" height="767" alt="image" src="https://github.com/user-attachments/assets/56d23b3a-6c5f-4740-a530-8a3f19404ee2" />

<img width="1832" height="951" alt="Screenshot 2025-12-11 130922" src="https://github.com/user-attachments/assets/31dd86e8-e513-435e-8e79-88b2140e1635" />


<img width="1465" height="1004" alt="Screenshot 2025-12-11 131110" src="https://github.com/user-attachments/assets/4477d2f1-0908-45e0-b108-25d2eab993e5" />

Security Features Implemented

To secure the execution environment, I implemented multiple Docker-based protection layers.

1. Timeout Limit (10 seconds)

This prevents infinite loops such as:

while True:
    pass


The container automatically stops after 10 seconds.

<img width="902" height="685" alt="image" src="https://github.com/user-attachments/assets/74f8d19f-eacf-4922-92c0-9a15f7839ed8" />
2. Memory Restriction (128 MB)

This stops programs that consume excessive memory, such as:

x = "a" * 1000000000


Docker terminates the container with exit code 137, meaning the memory limit was exceeded.

This demonstrated how Docker prevents memory-overflow attacks and protects the server.

<img width="1920" height="1008" alt="Screenshot 2025-12-10 160205 (1)" src="https://github.com/user-attachments/assets/fe0d256b-abda-43ba-8519-bdfe4d71ed4d" /> <img width="1126" height="225" alt="image" src="https://github.com/user-attachments/assets/c0155813-e727-41db-9cbd-ebebde1fd2b5" />
3. Network Disabled

Network access is blocked inside the container.
Requests like:

import requests
requests.get("http://example.com")


are not allowed.

This prevents malicious scripts from connecting to the internet.

<img width="1920" height="1008" alt="Screenshot 2025-12-09 145237" src="https://github.com/user-attachments/assets/70542933-b562-41e1-9f17-a671e82e30c3" /> <img width="1116" height="734" alt="image" src="https://github.com/user-attachments/assets/42f0c131-114a-4b2a-98bb-aecefb0047ae" />
4. Read-Only Filesystem

Enabling a read-only filesystem prevents scripts from writing or modifying files inside the container.

5. Automatic Cleanup

Every container is started with:

--rm


which removes the container immediately after execution.
This keeps the environment clean and avoids storage accumulation

3. Understanding Docker Security
Goal

Learn what Docker protects you from and what it does not protect you from.

Experiments Performed
1. Reading system files
with open("/etc/passwd") as f:
    print(f.read())


Result:
Inside the container, this command worked because /etc/passwd exists inside the container’s filesystem, not your host machine.
I learned that containers have their own system files and cannot directly access host OS files.



<img width="1299" height="817" alt="Screenshot 2025-12-11 192235" src="https://github.com/user-attachments/assets/a3478e0c-72d9-473e-9ba3-5fed2b374ce3" />


2. Writing files inside the container
with open("/tmp/test.txt", "w") as f:
    f.write("hacked!")


Result:
This worked initially because a container normally allows writing to its internal filesystem.



<img width="1523" height="597" alt="Screenshot 2025-12-11 192314" src="https://github.com/user-attachments/assets/1a92b3c6-3848-41b6-bb37-1960a296d421" />


Learning:
Even though the container is isolated, user code can still modify files inside the container.

3. Enabling read-only mode

I added this to the Docker command:

--read-only


Result:
Running the write test again failed with an error like:

PermissionError: Read-only file system


Learning:
This experiment showed me how to prevent users from creating or modifying files inside the container.

What I Learned About Docker Security

Docker isolates user code from the host machine.

User code cannot access real host system files, only container files.

File writes are allowed unless you enable --read-only.

Containers still allow reading internal system files unless restricted.

Additional flags like --pids-limit, --network none, and --memory help increase security.

Docker is secure, but it is not a full security boundary—you still need limits and restrictions.

4. Polish & Document the Project
Goal

Make the project clean, understandable, and ready for others to use.

Improvements Added
1. Clear error messages

Instead of showing generic errors like:

Error


I updated them to:

Execution timed out after 10 seconds
Memory limit exceeded
Invalid language selected
Input code too long


This makes debugging easier for users.

2. Input validation

Reject code longer than 5000 characters

Allow only safe languages

Block dangerous keywords inside user input (optional)

Documentation Added

The documentation now includes:

How to install and run the project

Example API requests and responses

Supported languages and limits

Security measures taken:

read-only filesystem

no network access

memory limits

timeout limits

running as non-root user

container destroyed after each run

What I learned during development

5. Basic Frontend UI

I added a simple HTML page:

A text area to paste code

A dropdown for selecting language

A submit button

An output section

This helps users test the API quickly.

6. Testing Everything

I re-ran all test cases including:

Infinite loops

Memory overflow

Syntax errors

Timeout conditions

File read/write tests

Multi-file ZIP tests

Empty input tests

Long input tests

I also had another person use the API to check if the documentation was clear.

Easy Level Enhancements
1. Add JavaScript (Node.js) Support

Extend the executor to run JavaScript code using a Node.js Docker image.
This helps your platform support multiple languages and makes it more useful.
<img width="1634" height="918" alt="Screenshot 2025-12-11 134418" src="https://github.com/user-attachments/assets/02381dc9-065f-4eb7-91da-1e521ca32dc7" />
<img width="1417" height="870" alt="Screenshot 2025-12-11 134234" src="https://github.com/user-attachments/assets/c4c7383e-180c-4545-897e-2a89951fa0c9" />
<img width="1766" height="968" alt="Screenshot 2025-12-11 134000" src="https://github.com/user-attachments/assets/6b3634b2-2133-48e2-b44a-333914be49b7" />
<img width="1496" height="930" alt="Screenshot 2025-12-11 133528" src="https://github.com/user-attachments/assets/2e0ad0b2-a86c-48d1-a422-d30e64afe6cc" />






2. Improve the Web UI

Make the frontend cleaner and easier to use:

Better layout

Larger code box

Clear buttons

Light/Dark mode (optional)

A simple UI upgrade greatly improves the user experience.

<img width="973" height="707" alt="Screenshot 2025-12-11 134654" src="https://github.com/user-attachments/assets/e0ed8ede-b4e0-4b40-be77-461d66bb26ab" />
<img width="939" height="772" alt="Screenshot 2025-12-11 134610" src="https://github.com/user-attachments/assets/b160a694-a5e4-4c9c-bb9e-c46400f67126" />
<img width="878" height="794" alt="Screenshot 2025-12-11 134527" src="https://github.com/user-attachments/assets/b72c626c-7929-4d84-99d3-9a54dcab0e49" />

<img width="870" height="731" alt="Screenshot 2025-12-11 135002" src="https://github.com/user-attachments/assets/fec0ef1f-6412-4b4e-a5f1-9caa883a7a7a" />
<img width="857" height="734" alt="Screenshot 2025-12-11 134923" src="https://github.com/user-attachments/assets/70cd9680-5709-493c-a627-3f7338e3e1c7" />


3. Add Execution History

Store the last 5–10 code runs:

Save code

Save output

Show timestamps

This helps users review previous attempts without retyping code.

Medium Level Enhancements
1. Support Multi-File Projects (ZIP Upload)

Allow users to upload a .zip file containing multiple files (like real coding platforms).
Your API should:

Extract the ZIP in a temporary container folder

Validate file types

Execute correctly based on an entry file

This makes your executor more powerful and closer to real-world tools.

<img width="1255" height="586" alt="Screenshot 2025-12-11 193705" src="https://github.com/user-attachments/assets/69574695-9212-4207-8766-366d7d7dbe89" />


2. Add Syntax Highlighting

Integrate a library like:

CodeMirror

Monaco Editor (VS Code editor engine)

This gives users:

Colored syntax

Auto-indent

Better code editing experience

3. Run Multiple Containers in Parallel

Allow up to 5 containers to run at the same time.
This teaches:

Handling concurrent requests

Pooling resources

Managing queue systems

And it makes the platform scalable.

What I Learned

Building this project helped me understand several important DevOps, security, and containerization concepts.
Each experiment showed how real systems protect themselves from unsafe or untrusted code.

1. How Docker isolates processes

I learned that Docker provides a completely isolated environment where untrusted code can run without impacting the host machine.
Even if the code is harmful, it stays confined inside the container.

2. How to control system resources

I implemented CPU limits, memory restrictions, and execution timeouts.
Through testing infinite loops and memory-heavy programs, I understood how these limits prevent system crashes and resource abuse.

3. How to block network access

By disabling network capabilities inside the container, I saw how scripts attempting to access external websites or APIs immediately failed.
This taught me how network isolation protects the backend from external attacks.

4. Difference between container filesystem and host filesystem

I discovered that reading system files inside the container (like /etc/passwd) only exposes container-specific data—not the host’s real files.
This helped me understand exactly what Docker isolates, and what it cannot fully protect.

5. How to integrate Docker with Flask

I learned how to make Flask execute Docker commands, manage containers, capture execution logs, and return clean JSON responses.
This helped me understand how backend systems interact with containerized sandboxes.

6. How to test and debug using Postman

Testing with Postman helped me observe how different code behaves:

Normal scripts

Infinite loops

Memory overflow attempts

File write attempts

Network requests

I improved debugging by reading error codes, container exit statuses, and timeout logs.

7. Importance of secure code execution

This project showed why executing user code directly on a server is extremely dangerous.
Docker’s isolation, filesystem separation, and resource controls make it safe to run unknown programs.

8. Understanding Docker security limitations

Through experiments like reading /etc/passwd and attempting to write files, I learned:

Containers can still read internal system files

Containers can write files unless the filesystem is set to read-only

Adding --read-only stops write operations completely

This helped me understand real Docker security boundaries.

9. How file uploads and ZIP-based execution work

By testing ZIP uploads, I learned how backend servers handle:

File extraction

Directory isolation

Running multi-file code inside containers

This taught me how real coding platforms process user project files.

10. Why proper error handling is important

I learned that user-friendly error messages improve the developer experience.
Instead of returning just “Error”, clear messages like “Execution timed out after 10 seconds” help users understand what went wrong.

11. How to make APIs production-friendly

Through this project, I gained practical experience in:

Limiting request size

Validating user input

Securing endpoints

Cleaning up containers automatically

This helped me understand how professional APIs are structured.
