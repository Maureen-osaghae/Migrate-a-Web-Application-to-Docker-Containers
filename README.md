<h1>Migrate a Web Application to Docker Containers</h1>
<h2>Lab overview and objectives</h2>

In this lab, you will migrate a web application to run on Docker containers. The application is installed directly on the guest operating systems (OSs) of two Amazon Elastic Compute Cloud (Amazon EC2) instances. You will migrate the application to run on Docker containers.

After completing this lab, you should be able to:

<ol>
<li>Create a Dockerfile.</li>
<li>Create a Docker image by using a Dockerfile.</li>
<li>Run a container from a Docker image.</li>
<li>Interact with and administer your containers.</li>
<li>Create an Amazon Elastic Container Registry (Amazon ECR) repository.</li>
<li>Authenticate the Docker client to Amazon ECR.</li>
<li>Push a Docker image to Amazon ECR.</li>
</ol>

</h2>Business Scenario</h2>

The café owners have noticed how popular their gourmet coffee offerings have become. Customers cannot seem to get enough of their cappuccinos and lattes. Meanwhile, the café owners have been challenged to consistently source the highest quality coffee beans. Recently, the owners learned that one of their favourite coffee suppliers wants to sell her company. Frank and Martha jumped at the opportunity to buy the company.
The acquired coffee supplier runs an inventory tracking application on an AWS account. Sofía has been tasked to understand how the application works and then create a plan to integrate the application into the café's existing application infrastructure.
In this lab, I will again play the role of Sofía, and work to migrate the application to run on containers.

The following diagram shows the architecture that was created for me in AWS at the beginning of the lab: 

<img width="416" alt="image" src="https://github.com/user-attachments/assets/44cbcaec-2221-4877-8102-159b780cdb7e" />

By the end of this lab, I have migrated the application and the backend database to run as Docker containers, as shown in the following diagram: 

<img width="421" alt="image" src="https://github.com/user-attachments/assets/765ceecd-d8f1-40db-8711-7f346bf7b3db" />

I will register these two containers in Amazon ECR to make them available to deploy as needed.

Let's get started!

<h2>Preparing the development environment</h2>

<h3>Connect to the VS Code IDE.</h3>

 Copy the LabIDEURL and LabIDE Password
In a new browser tab, paste the value for LabIDEURL to open the VS Code IDE.

On the prompt window Welcome to code-server, enter the value for LabIDEPassword you copied to the editor earlier, choose Submit to open the VS Code IDE.

<img width="785" alt="image" src="https://github.com/user-attachments/assets/64e1e463-6df0-4f80-b1f1-79393f064b09" />

Download and extract the files that you will need for this lab. In the same terminal, run the following command:

    wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACCDEV-2-44869/06-lab-containers/code.zip -P /home/ec2-user/environment

The code.zip file is downloaded to the VS Code IDE. The file is listed in the left navigation pane.

Extract the file:
    
    unzip code.zip

<img width="461" alt="image" src="https://github.com/user-attachments/assets/a4acf207-4caa-45eb-b050-1dd339c88c00" />


Run a script that will upgrade the version of Python and the AWS CLI that are installed on the VS Code IDE.

Set permissions on the script so that you can run it, then run it:

    chmod +x ./resources/setup.sh && ./resources/setup.sh

Verify that the script completed without error. Verify the version of AWS CLI installed.
      In the VS Code IDE Bash terminal (at the bottom of the IDE), run the following command:

aws –version
The output should indicate that version 2 is installed. Verify that the SDK for Python is installed. Run the following command:
          
          pip3 show boto3

<h2>Task 2: Analysing the existing application infrastructure</h2>

In this task, you will analyze the current application infrastructure. Open the existing coffee supplier application in a browser tab. Return to the browser tab labeled Your environments, and navigate to the EC2 console. Choose Instances.
Notice that three instances are running.

<img width="959" alt="image" src="https://github.com/user-attachments/assets/430351f3-e843-4cbf-ac04-c85bad8b3969" />

<ol>
<li>One instance is the VS Code IDE that you used in the previous task.</li>
<li>The other two instances (MysqlServerNode and AppServerNode) support the application that you will containerize in this lab.</li>
<li>Choose the AppServerNode instance, and copy the Public IPv4 address value.</li>
<li>Open a new browser tab and navigate to the IP address.</li>
</ol>
 The coffee suppliers website displays.

 <img width="821" alt="image" src="https://github.com/user-attachments/assets/c8028790-ffef-4654-9f53-b206714db299" />

 Note: The page uses http:// instead of https://. Your browser might indicate that the site is not secure, because it does not have a valid SSL/TLS certificate. You can ignore the warning in this development environment. 

<ol>
  <li>Choose List of suppliers and then choose Add a new supplier.</li>
  <li>Fill in all of the fields with values. For example:</li>
  <li>Name: Nikki Wolf</li>
  <li>Address: 100 Main Street</li>
  <li>City: Anytown</li>
  <li>State: CA</li>
  <li>Email: nwolf@example.com </li>
  <li>Phone: 4155551212</li>
  <li>Choose Submit.</li>
</ol>

The All suppliers page displays and includes the record that you submitted.

<img width="947" alt="image" src="https://github.com/user-attachments/assets/6bf06088-54ca-4435-9329-0f0fa2a5320b" />

 •  Choose edit and change the record (for example, modify the phone number).
      
 •  To save the change, choose Submit.
      
 •  Notice that the change was saved in the record.

Analyze the web application code.

A copy of the application code that is installed on the AppServerNode EC2 instance is also available in your VS Code IDE. Return to the VS Code IDE browser tab.
In the file browser in the left navigation pane, expand the resources directory, and then expand the codebase_partner directory to see the application code.

<h3>Optional:</h3>
If you are interested to know how the code is configured on the AppServerNode, you can connect to the instance and view the code there as well. 

To do this, in the terminal next to these instructions, run the following commands, to connect to the EC2 instance and see the files that are installed (replace <public-ip-address> with the actual IPv4 address of the AppServerNode instance).

    ssh -i ~/.ssh/labsuser.pem ubuntu@<public-ip-address>
    cd resources/codebase_partner
    ls -l

For this lab, it is not necessary for you to understand the details of how the application was built. However, the following details might be of interest to you:
        
◦  The application was built with Express, which is a framework for building web applications.
          
◦ The application runs on port 80 and is coded in node.js.
          
◦ To install the application directly on the guest OS of the AppServerNode Ubuntu Linux EC2 instance, node.js and the node package manager (npm) were installed. 

Then, the code that you can see in resources/codebase_partner was placed on the server, and the connection to the application's database was configured.

<h2>Task 3: Migrating the application to a Docker container</h2>

In this task, you will migrate an application that is installed directly on the guest OS of an Ubuntu Linux EC2 instance to instead run in a Docker container. The Docker container is portable and could run on any OS that has the Docker engine installed.
For convenience, you will run the container on the same EC2 instance that hosts the VS Code IDE that you are using. You will use this IDE to build the Docker image and launch the Docker container.

The following diagram shows the migration that you will accomplish in this task:

<img width="404" alt="image" src="https://github.com/user-attachments/assets/39af85bd-8876-4ffc-80dc-f9573973ff8e" />

Create a working directory for the node application, and move the source code into the new directory.

In the VS Code IDE, to create and navigate to a directory to store your Docker container code, run the following commands:

    mkdir containers
    cd containers

To create and navigate to a directory named node_app inside of the containers directory, run the following commands:
     
      mkdir node_app
      cd node_app
To move the code base, which you copied earlier, into the new node_app directory, run the following command:
     
      mv ~/environment/resources/codebase_partner 
      ~/environment/containers/node_app

<h3>Create a Dockerfile.</h3>
<span>Note:</span>
A Dockerfile is where you provide instructions to Docker to build an image. A Docker image is a template that has instructions to create a container.
To create a new Dockerfile named Dockerfile in the node_app/codebase_partner directory, run the following command:

    cd ~/environment/containers/node_app/codebase_partner
    touch Dockerfile

In the left navigation pane, browse to and open the empty Dockerfile that you just created.

Copy and paste the following code into the Dockerfile: 

    FROM node:11-alpine
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    COPY . .
    RUN npm install
    EXPOSE 3000
    CMD ["npm", "run", "start"]

<img width="610" alt="image" src="https://github.com/user-attachments/assets/b0985c66-042f-4bed-8121-76e85c3d0c80" />

Save the changes.
<h3>Analysis:</h3>
This Dockerfile code specifies that an Alpine Linux distribution with node.js runtime requirements should be used to create the image. The code also specifies that the container should allow network traffic on TCP port 3000. Finally, the code specifies that the application should be run and started when the container launches.

<h3>Build an image from the Dockerfile.</h3>
In the VS Code IDE Bash terminal, run the following command:

    docker build --tag node_app .

The output is similar to the following: 
    
    [+] Building 9.4s (10/10) FINISHED                                                                         docker:default
     => [internal] load build definition from Dockerfile                                                                 0.0s
     => => transferring dockerfile: 225B                                                                                 0.0s
     => [internal] load metadata for docker.io/library/node:11-alpine                                                    0.3s
     => [internal] load .dockerignore                                                                                    0.0s
     => => transferring context: 2B                                                                                      0.0s
     => [1/5] FROM docker.io/library/node:11-alpine@sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d50  3.4s
     => => resolve docker.io/library/node:11-alpine@sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d50  0.0s
     => => extracting sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10                            0.2s
     => => sha256:0119aca4464934fdf40121363b1cf0537e464c07c790df99058ba587b14aaadd 21.66MB / 21.66MB                     0.5s
     => => sha256:40df19605a18a2dc4e3c6c0cb5d0a93fd2f7615e2a92eccd743698c24c23fe84 1.33MB / 1.33MB                       0.1s
     => => sha256:82194b8b4a64bd286671a1464d3fe319832aaa67c4e41c455fdfdd295ae7aa73 279B / 279B                           0.2s
     => => sha256:8bb56bab197299c8ff820f1a55462890caf08f57ffe3b91f5fa6945a4d505932 1.42kB / 1.42kB                       0.0s
     => => sha256:914ff2c2145de019a19c080a9e42b5763c826194110ec8e02c8e92845799fba6 1.16kB / 1.16kB                       0.0s
     => => sha256:f18da2f58c3dabd7d06c02f4a76719ba0c786737aa83d4bfc8f1e5cb0800b96a 5.66kB / 5.66kB                       0.0s
     => => sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10 2.76MB / 2.76MB                       0.1s
     => => extracting sha256:0119aca4464934fdf40121363b1cf0537e464c07c790df99058ba587b14aaadd                            2.6s
     => => extracting sha256:40df19605a18a2dc4e3c6c0cb5d0a93fd2f7615e2a92eccd743698c24c23fe84                            0.1s
     => => extracting sha256:82194b8b4a64bd286671a1464d3fe319832aaa67c4e41c455fdfdd295ae7aa73                            0.0s
     => [internal] load build context                                                                                    0.0s
     => => transferring context: 1.25MB                                                                                  0.0s
     => [2/5] RUN mkdir -p /usr/src/app                                                                                  0.8s
     => [3/5] WORKDIR /usr/src/app                                                                                       0.0s
     => [4/5] COPY . .                                                                                                   0.1s
     => [5/5] RUN npm install                                                                                            4.3s
     => exporting to image                                                                                               0.3s
     => => exporting layers                                                                                              0.3s
     => => writing image sha256:be21636b8618e57c99de2ffef205bb1567e9f326795d58df9dcc8682c14e4fe0                         0.0s
     => => naming to docker.io/library/node_app   
    

Verify that the Docker image was created. To list the Docker images that your Docker client is aware of, run the following command:
      
      docker images

<img width="715" alt="image" src="https://github.com/user-attachments/assets/ae8295f2-2741-484c-bd02-1c664d2c090a" />

<h3>Create and run a Docker container based on the Docker image.</h3>
      
To create and run a Docker container from the image, run the following command:
      
    docker run -d --name node_app_1 -p 3000:3000 node_app

<h3>Analysis:</h3>
This command launches a container with the name node_app_1, using the node_app image that you created as the template. The -d argument specifies that it should run in the background and print the container ID. The -p specifies to publish container port 3000 to the host (VS Code IDE) port 3000. The terminal returns the container ID. It is a long string of letters and numbers.

 To view the Docker containers that are currently running on the host, run the following command:
     
      docker container ls

  <img width="959" alt="image" src="https://github.com/user-attachments/assets/dd91fd43-d9ca-42a3-abfe-2985df7db2f7" />

Verify that the node application is now running in the container.

Verify that the node application is now running in the container. To check that the container is working on the correct port, run the following command:
      
      curl http://localhost:3000
      
 <img width="939" alt="image" src="https://github.com/user-attachments/assets/59440489-b35c-4e14-81f8-61b658fff0b6" />

 The webpage looks similar to the following:

     <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="/css/bootstrap.min.css">
        <link rel="stylesheet" href="/css/base.css">
        <title>Coffee suppliers</title>
    </head>
    <body>
    
    <div class="container">
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
            <img src="/img/espresso.jpg" width="200">
            <div><a class="navbar-brand page-title" href="/supplier">Coffee suppliers</a></div>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <ul class="navbar-nav mr-auto">
                    <li class="nav-item active">
                        <a class="nav-link" href="/">Home</a>
                        <a class="nav-link" href="/suppliers">Suppliers list</a>
                    </li>
                </ul>
            </div>
        </nav>    <div class="container">
            <h1>Welcome</h1>
            <p>Use this app to keep track of your coffee suppliers</p>
            <p><a href="/suppliers">List of suppliers</a></p>
        </div>
    </div>
    <script src="/js/jquery-3.6.0.min.js"></script>
    <script src="/js/bootstrap.min.js"></script>
    </body>

<h3>Analysis: </h3>
This demonstrates that the application is running and available on TCP port 3000. However, you want to interact with this as a website, and you don't yet know if the application is able to connect to the database. 

Adjust the security group of the VS Code IDE instance to allow network traffic on port 3000 from your computer.
       
Note: Because you are using the VS Code IDE instance to run the container, you must open TCP port 3000 for inbound traffic.
<ol>
<li>Return to the AWS Management Console browser tab, and navigate to the EC2 console.</li>
<li>Locate and select the Lab IDE instance.</li>
<li>Choose the Security tab, and choose the security group hyperlink.</li>
<li>Choose the Inbound rules tab, and choose Edit inbound rules.</li>
<li>Choose Add rule and configure the following:</li>
<li>Type: Custom TCP</li>
<li>Port range: 3000</li>
<li>Source: My IP</li>
<ol>

<img width="959" alt="image" src="https://github.com/user-attachments/assets/c3fb4d45-9a39-4b96-94cc-9a7b328afb81" />

◦ Choose Save rules.

Access the web interface of the application, which is now running in a container.

◦ In the EC2 console, choose Instances and choose the Lab IDE instance.
 
◦ On the Details tab, copy the Public IPv4 address value.

◦ Open a new browser tab. Paste the IP address into the address bar, and add :3000 at the end of the address.
 
The web application loads in the browser. You have seen this page before; however, you previously accessed the application that was running directly on the AppServerNode EC2 instance guest OS. This time you accessed the application running in a container on the VS Code IDE instance's Docker hypervisor.

<img width="803" alt="image" src="https://github.com/user-attachments/assets/6c7605b1-5bd4-4174-ab22-e921c5d83c4c" />

<h3>Summary of how you have used Docker so far</h3>

You just completed a series of steps with Docker. The following diagram summarizes what you have accomplished with Docker so far.

<img width="389" alt="image" src="https://github.com/user-attachments/assets/40b23b2c-4bb0-4ccd-8590-6ad2f3d2f94e" />

You copied the code base into a directory, which acted as your build area [a]. You also created a Dockerfile that provided instructions for how to create a Docker image. That Dockerfile specified a FROM instruction that identified a starter image to use. You then ran the docker build command [b]. Docker read the Dockerfile and requested the starter image from an image repository [c]. The image repository returned the starter image file [d]. The docker build command then finished building the image according to the instructions in the Dockerfile, which resulted in the Docker image [e]. Finally, you ran the docker run command [f] to run a Docker container [g].














