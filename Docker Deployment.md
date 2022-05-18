CodeCheck<sup>®</sup> 
===============================



## About Codecheck
Codecheck is an anonymous, author-friendly autograder. It is optimized for simple programming assignments that provide practice and build confidence.



![Codecheck - Student view](/home/sergiorojas/Downloads/My%20project%20(1).png)


Program Structure
-----------------

CodeCheck has two parts:

-   A web application that manages submission and assignments. It is
    called `play-codecheck` because it is based on the Play framework.
    It is convenient, but not necessary, to run it in Docker.
-   A Dockerized service that compiles and runs programs, called
    `comrun`.

The `play-codecheck` program has a number of responsibilities:

-   Display problems, collect submissions from students, and check them
-   Manage problems from instructors
-   Manage assignments (consisting of multiple problems)
-   Interface with learning management systems through the LTI protocol

You can find a listing of the supported REST services in the
`app/conf/routes` file.

The `comrun` service is extremely simple. You can find a basic
description in the `comrun/bin/comrun` script.

For local testing of problems, there is also a handy command-line tool.
This tool uses only the part of `play-codecheck` that deals with
checking a problem (in the `com.horstmann.codecheck` package). The tool
is called `codecheck`. It is created by the `cli/build.xml` Ant script.

## Dependencies
  - openjdk-11-jdk [https://openjdk.java.net](https://openjdk.java.net/projects/jdk/11)
  - git [https://git-scm.com](https://git-scm.com/)
  - ant [https://ant.apache.org](https://ant.apache.org/)
  - curl [https://curl.se](https://curl.se/)
  - unzip
  - sbt [https://www.scala-sbt.org](https://www.scala-sbt.org/)
  - docker [https://www.docker.com](https://www.docker.com/)
  - gcloud CLI SDK [https://cloud.google.com](https://cloud.google.com/)
  - AWS CLI [https://aws.amazon.com](https://aws.amazon.com/)

# Build Instructions Build the web application locally

## Step 1: Download Codecheck codebase
Clone the the codecheck repository using git

  ```
  git clone https://github.com/cayhorstmann/codecheck2
  ```

## Step 2: Install Codecheck dependencies
Update the apt package index, and install the latest version of openjdk, git, ant, curl, and unzip

  ```
  sudo apt-get update
  sudo apt install openjdk-11-jdk git ant curl unzip
  ```

## Step 3: Install sbt
[follow the instruction for your environment](https://www.scala-sbt.org/download.html)

1. Add sbt Debian main package repository

  ```
  echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
  echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt_old.list
  curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
```

2. Update the apt package index, and install the latest version of sbt
```
sudo apt-get update
sudo apt-get install sbt
 ```

3. Confirm the installation with the following command
```
sbt --version
```

![sbtversion.png](/home/sergiorojas/Downloads/sbtversion.png)

## Step 4: Setup Codecheck environment

  Create a /opt/codecheck directory and a subdirectory ext that you own

  ```
  sudo mkdir -p /opt/codecheck/ext
  export ME=$(whoami) ; sudo -E chown $ME /opt/codecheck /opt/codecheck/ext
  ```

## Step 5: Build the command-line tool

  1. Install jackson-core, jackson-annotations, and jackson-databind jar files

  From the root directory of the repository, go to cli directory and make a lib directory

  ```
  cd codecheck2/cli
  mkdir lib
  ```

  Install the jar files in the lib directory

  ```
  cd lib
  curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-core/2.6.4/jackson-core-2.6.4.jar
  curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-annotations/2.6.4/jackson-annotations-2.6.4.jar
  curl -LOs https://repo1.maven.org/maven2/com/fasterxml/jackson/core/jackson-databind/2.6.4/jackson-databind-2.6.4.jar
  ```

  2. Install checkstyle, hamcrest-core, and junit jar files

  From the root directory of the repository, go to comrun directory and next go to bin directory, then make a lib directory

  ```
  cd comrun/bin
  mkdir lib
  ```

  Install the jar files in the lib directory

  ```
  cd lib
  curl -LOs https://repo1.maven.org/maven2/com/puppycrawl/tools/checkstyle/8.42/checkstyle-8.42.jar
  curl -LOs https://repo1.maven.org/maven2/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar
  curl -LOs https://repo1.maven.org/maven2/junit/junit/4.13.2/junit-4.13.2.jar
  ```

 3.  From the root directory of the repository, build the command-line tool

  ```
  ant -f cli/build.xml
  ```
![buildcommandlinetool.png](/home/sergiorojas/Downloads/buildcommandlinetool.png)
 4. To verify that it works

  ```
  /opt/codecheck/codecheck -t samples/java/example1
  ```

If you omit the -t, you get a report with your default browser instead of the text report.
![verifycommandlinetool.png](/home/sergiorojas/Downloads/verifycommandlinetool.png)

Debugging the Command Line Tool
-------------------------------

If you are making changes to the part of CodeCheck that does the actual
code checking, such as adding a new language, and you need to run a
debugger, it is easiest to debug the command line tool.

Make directories for the submission and problem files, and populate them
with samples.

In your debug configuration, set:

-   The main class to

        com.horstmann.codecheck.Main

-   Program arguments to

        /path/to/submissiondir /path/to/problemdir

-   VM arguments to

        -Duser.language=en
        -Duser.country=US
        -Dcom.horstmann.codecheck.comrun.local=/opt/codecheck/comrun
        -Dcom.horstmann.codecheck.report=HTML
        -Dcom.horstmann.codecheck.debug

-   The environment variable `COMRUN_USER` to your username

To debug on Windows or MacOS, you have to use the Docker container for
compilation and execution.

    docker build --tag comrun:1.0-SNAPSHOT comrun
    docker run -p 8080:8080 -it comrun:1.0-SNAPSHOT

Point your browser to <http://localhost:8080/api/health> to check that
the container is running.

When debugging, add the VM argument

    -Dcom.horstmann.codecheck.comrun.remote=http://localhost:8080/api/upload

## Step 6: Build the web application locally
Install, following the instructions of the providers,

-   [Eclipse](https://www.eclipse.org/eclipseide/)

  From the root directory of the repository, run the play-codecheck server

  ```
COMRUN_USER=$(whoami) sbt run
  ```

To verify that it works, point the browser to <http://localhost:9000/assets/uploadProblem.html>.
Upload a problem and test it.

---
**Notes**
	The problem files will be located inside the /opt/codecheck/ext directory.
---

![codecheckweb.png](/home/sergiorojas/Downloads/codecheckweb.png)


Debugging the Server
--------------------

Import the project into Eclipse. Run

    sbt eclipse

Then open Eclipse and import the created project.

Make a debugger configuration. Select Run → Debug Configurations,
right-click on Remote Java Application, and select New Configuration.
Specify:

-   Project: `play-codecheck`
-   Connection type: Standard
-   Host: `localhost`
-   Port: 9999

Run the `play-codecheck` server in debug mode:

    COMRUN_USER=$(whoami) sbt -jvm-debug 9999 run

In Eclipse, select Run → Debug Configurations, select the configuration
you created, and select Debug. Point the browser to a URL such as
<http://localhost:9090/assets/uploadProblem.html>. Set breakpoints as
needed.


# Docker Deployment

## Step 1: Install docker
[follow the instruction for your environment](https://docs.docker.com/engine/install/)

1. Update the apt package index and install the latest version of ca-certificates, curl, gnupg, and lsb-release


  ```
   sudo apt-get update
   sudo apt-get install ca-certificates curl gnupg lsb-release
  ```

  2. Add Docker’s official GPG key

  ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  ```

 3. Add docker Debian stable package repository

  ```
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```

4. Update the apt package index, and install the latest version of Docker Engine, containerd, and Docker Compose

  ```
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
  ```

5. Start docker

```
sudo service docker start
```

6. Verify that Docker Engine is installed correctly by running the hello-world image.

```
sudo docker run hello-world
``` 
This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.

![dockerhelloworld.png](/home/sergiorojas/Downloads/dockerhelloworld.png)


## Step 2: Build and run the comrun service Docker container

  1. From the root directory of the repository, build the comrun service Docker container

  ```
  sudo docker build --tag codecheck:1.0-SNAPSHOT comrun
  ```

  2. From the root directory of the repository, run the comrun service Docker container

  ```
  sudo docker run -p 8080:8080 -it codecheck:1.0-SNAPSHOT
  ```

  3. To verify that it works

  ```
  /opt/codecheck/codecheck -l samples/java/example1 &
  ```

  
![verifycomrundocker.png](/home/sergiorojas/Downloads/verifycomrundocker.png)
  

## Step 3: Build and run the play-codecheck Docker container
1. Create a file `conf/production.conf` holding an [application
secret](https://www.playframework.com/documentation/2.8.x/ApplicationSecret):


```
echo "play.http.secret.key=\"$(head -c 32 /dev/urandom | base64)\"" > conf/production.conf
echo "com.horstmann.codecheck.comrun.remote=\"http://host.docker.internal:8080/api/upload\"" >> conf/production.conf
```

---
** Note **
Do not check this file into version control
---


  2. From the root directory of the repository, build the Docker image

  ```
  sudo sbt docker:publishLocal 
  ```

  3. From the root directory of the repository, run the Docker container

  ```
  sudo docker run -p 9090:9000 -it --add-host host.docker.internal:host-gateway play-codecheck:1.0-SNAPSHOT
  ```  

To verify that it works, point the browser to <http://localhost:9090/assets/uploadProblem.html>.
Upload a problem and test it.

  4. Shutdown both containers by running this command in another terminal


  ```
  docker container kill $(docker ps -q)
  ```



  
# Cloud Deployment
## Step 1: Install Google Cloud CLI
[follow the instruction for your environment](https://cloud.google.com/sdk/docs/install#linux)

1. Download Linux Google Cloud SDK archive file

  ```
  curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-373.0.0-linux-x86_64.tar.gz
  ```

 2. Extract the contents of the file to any location on your file system (preferably your Home directory). To replace an existing installation, remove the existing google-cloud-sdk directory and then extract the archive to the same location.

  ```
  tar -xf google-cloud-sdk-373.0.0-linux-x86_64.tar.gz
  ```

3. Run the install script and answer the prompts to add the gcloud CLI tools to your PATH (from the root of the folder you extracted to)

  ```
  ./google-cloud-sdk/install.sh
  ```

4. To initialize the gcloud CLI, run gcloud init

  ```
  ./google-cloud-sdk/bin/gcloud init
  ```

5. Confirm the installation with the following command
```
gcloud --version
```

## Step 2: Install AWS CLI
[follow the instruction for your environment](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

1. Download the AWS CLI installation file

  ```
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  ```

2. Unzip the installer package and creates a directory named aws under the current directory

  ```
  unzip awscliv2.zip
  ```

3. Run the install program

  ```
  sudo ./aws/install
  ```

4. Confirm the installation with the following command

  ```
  aws --version
  ```

5. Configure AWS CLI [instruction](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html)



  ```
$ aws configure
  ```

AWS CLI prompts you for four pieces of information:

  - Access key ID
  - Secret access key
  - AWS Region
  - Output format

![awsconfigure.png](/home/sergiorojas/Downloads/awsconfigure.png)


## Step 3: Authenticate with Google Cloud Container Registry

  [Follow the instruction for your environment](https://cloud.google.com/container-registry/docs/advanced-authentication)

  1. Add the user that you use to run Docker commands to the Docker security group

  ```
  sudo usermod -a -G docker ${USER}
  ```

  2. Log in to gcloud as the user that will run Docker commands.

  ```
  gcloud auth login
  ```

  3. Configure Docker with the following command

  ```
  gcloud auth configure-docker
  ```

  Your credentials are saved in your user home directory

 Linux: 
```
  $HOME/.docker/config.json
  ```

 





Step 4: Comrun Service Deployment {#service-deployment}
-------------------------

There are two parts to the CodeCheck server. We\'ll take them up one at
a time. The `comrun` service compiles and runs student programs,
isolated from the web app and separately scalable.

Here is how to deploy the `comrun` service to [Google Cloud](https://cloud.google.com/).

1. Make a [Google Cloud Run](https://console.cloud.google.com/run?project) project. Define a service `comrun`.

![googlecloudhome.png](/home/sergiorojas/Downloads/googlecloudhome.png)

* Click on Create Service
![clickoncreateservice.png](/home/sergiorojas/Downloads/clickoncreateservice.png)
![](/home/sergiorojas/Downloads/createService.png)

* For Deploy one revision from an existing container image: Click on Test With a Sample Container

![](/home/sergiorojas/Downloads/testWithSampleContainers.png)

  * Change the Service name to Comrun

![](/home/sergiorojas/Downloads/comrunservicename.png)

  * Don't change the Region default value

![](/home/sergiorojas/Downloads/defaultRegion.png)

  * CPU allocation and pricing should be CPU is only allocated during request processing

![](/home/sergiorojas/Downloads/cpuAllocation.png)


  * Authentication should be Allow unauthenticated invocations

![](/home/sergiorojas/Downloads/comrunAuthentication.png)
  
*  Click on Create

![](/home/sergiorojas/Downloads/createComrun.png)

* Success
![](/home/sergiorojas/Downloads/comrunsHome.png)

2. Get the comrun service project name

   Go to https://console.cloud.google.com/run?project and in the URL bar should be the name of the project. 




```
  https://console.cloud.google.com/run?project=xx-xx
  ```



  

  ![conrumprojectnames.png](/home/sergiorojas/Pictures/conrumprojectnames.png)









3. Then run:

  ```
  export PROJECT=your Google project id
 ```

4. Deploy the Comrun service

    docker tag codecheck:1.0-SNAPSHOT gcr.io/$PROJECT/comrun
    docker push gcr.io/$PROJECT/comrun

    gcloud run deploy comrun \
      --image gcr.io/$PROJECT/comrun \
      --port 8080 \
      --platform managed \
      --region us-central1 \
      --allow-unauthenticated \
      --min-instances=1 \
      --max-instances=50 \
      --memory=512Mi \
      --concurrency=40

You should get a URL for the service. Make a note of it---it won\'t
change, and you need it in the next steps. To test that the service is
properly deployed, do this:

    export REMOTE_URL=the URL of the comrun service
    cd path to/codecheck2 
    /opt/codecheck/codecheck -rt samples/java/example1

You should get a report that was obtained by sending the compile and run
jobs to your remote service.

Alternatively, you can test with the locally running web app. In
`conf/production.conf`, you need to add

    com.horstmann.codecheck.comrun.remote= the URL of the comrun service

## Verify comrun service is running


  To check the health of the comrun service, visit this url

  ```
  https://service url/api/health
  ```




## How to scale your comrun service

  Following the [guideline from Google Cloud](https://cloud.google.com/run/docs/about-instance-autoscaling)

  Go to your [google cloud Run](https://console.cloud.google.com/run)

  - Click on a comrun service
  - Click on Edit and Deploy New Revision
  - Under Capacity you can change the Memory and CPU settings
  - Under Capacity you can change the Maximum requests per container (Concurrency)
  - Under Autoscaling you can change the Minimum and Maximum instances










   

  

Step 5: Play Server Deployment {#server-deployment}
----------------------

In Amazon S3, create a bucket whose name starts with the four characters `ext.` and an arbitrary suffix, such as `ext.mydomain.com` to hold
the uploaded CodeCheck problems. Set the ACL so that the bucket owner has all access rights and nobody else has any.






  Create a [Amazon S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html)



  Go to your aws s3 bucket page


  Click on Create Bucket

![Screenshot 5-12-2022 10-25-52 AM.png](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-25-52%20AM.png)
  Name your bucket with ext.

![](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-30-08%20AM.png)

  * Bucket Name: ext.mydomain.com
  * AWS Region: us-west-1

For object ownership set it to ACLs disabled
![](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-38-24%20AM.png)


  * uncheck Block all public access

![](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-47-16%20AM.png)

* Bucket Versioning: Disable
* Set Default encryption to Disable
* Click on Create Bucket




![](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-49-18%20AM.png)
  
Successfully created bucket Message
![](/home/sergiorojas/Downloads/Screenshot%205-12-2022%2010-53-02%20AM.png)

In your [Google Cloud Run](https://console.cloud.google.com/run?project) project, add another service `play-codecheck`.


Add the following to `conf/production.conf`:

    play.http.secret.key= see above
    com.horstmann.codecheck.comrun.remote= Google Cloud comrun host URL/api/upload
    com.horstmann.codecheck.s3.accessKey= your AWS credentials
    com.horstmann.codecheck.s3.secretKey=
    com.horstmann.codecheck.s3bucketsuffix="mydomain.com"
    com.horstmann.codecheck.s3.region=your AWS region such as "us-west-1"
    com.horstmann.codecheck.repo.ext=""
    com.horstmann.codecheck.storeLocation=""



## How to get AWS creditional

  1. Go to your  account

  2. Click on your profile that is on the top right

--
**Note**
comrun remote url is the google cloud run service url. To get the URL of the comrun service. Go to your Google Cloud account and then go to your comrun service and there should be a url.
--


  


---
**Error:** java.util.NoSuchElementException: submissionrun1/_run
 Solution: Update the production.conf file with /api/upload append at the end of the comrun remoute url. ``` com.horstmann.codecheck.comrun.remote="https://comrun-url/api/upload/"```
---

---
**Error:** Unknown Host Exception: host.docker.internal
 Solution: Add this to production.conf ``` com.horstmann.codecheck.storeLocation=""```
---

---
**Error:** Deployment failed: (gcloud.run.deploy)
 Cloud Run Error: Container failed to start. Failed to start and then listen on the port defined by the PORT environment variable. Logs for this revision might contain more information.
Solution: Add this to production.conf ``` com.horstmann.codecheck.storeLocation=""```
---
  










Deploy the `play-codecheck` service:


  After creating a project look for the project id

  ```
  export PROJECT=your Google project name
  ```




    docker tag play-codecheck:1.0-SNAPSHOT gcr.io/$PROJECT/play-codecheck
    docker push gcr.io/$PROJECT/play-codecheck

    gcloud run deploy play-codecheck \
      --image gcr.io/$PROJECT/play-codecheck \
      --port 9000 \
      --platform managed \
      --region us-central1 \
      --allow-unauthenticated \
      --min-instances=1

You will get a URL for the service. Now point your browser to
`https://service url/assets/uploadProblem.html` and upload a problem




  







 
  








     
      

    
     



  


  

  


  

  

  

 








## How to upload a test file to AWS S3 bucket using the CLI

  To upload a file to S3, you’ll need to provide two arguments (source and destination) to the aws s3 cp command.

  ```
  aws s3 cp test.txt s3://<BUCKET-NAME>
  ```




## How to upload multiple zip files to AWS S3 bucket using the CLI

  Open a terminal and go to the directory where the the zip files are located

  Uploads the zip files in the current directory to your AWS bucket

  ```
  for f in $(ls *) ; do aws s3 cp $f s3://ext.yourbucketsuffix.edu; done
  ```

 



  