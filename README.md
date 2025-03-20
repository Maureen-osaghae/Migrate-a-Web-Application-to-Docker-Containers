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








