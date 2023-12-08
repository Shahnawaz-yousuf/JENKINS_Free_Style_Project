JENKINS free style - PROJECT (Steps):

# CONTINUOUS INTEGRATION (CI):=========

### Step-1: Go to AWS Console and create 2 instances:
 
        (a)instance1 - Dev-server (AMI-Ubuntu 20.04 LTS,t2-micro,), security group (ssh-port22, and TCP custome-port8080)
        (b)instance2 - QA-server (AMI-Ubuntu 20.04 LTS,t2-micro,), security group (ssh-port22, and TCP custome-port8080)


### Step-2: take ssh-connetion of Dev-server => run the command:

         (a) sudo apt-get update -y (hit enter)
         (b) Install java - sudo apt install openjdk-11-jdk -y (hit enter)
         (c) Install maven & git - sudo apt-get install maven git -y (hit enter)
         (d) Install JENKINS - wget https://get.jenkins.io/war-stable/2.426.1/jenkins.war (hit enter)
         (e) Now extract the JENKINS file (jenkins.war) - java -jar jenkins.war (hit enter)
         (f) Now to open the JENKINS GUI on browser-copy the public IP of Dev-server and paste on the browser with :8080 (enter)
         (g) Now copy the required password from Dev-server terminal and paste in JENKINS GUI (Administrator password:)
         (h) Now on JENKINS GUI select-Install suggested Plugins and wait for process
         
>> Note: Whenever JENKINS GUI page is not reachable on browser just go to Dev-server terminal and execute the same command: java -jar jenkins.war (hit enter)
         (i) Again go to JENKINS GUI browser and refresh it and login again with your credentials:(user_name & password)

### Step-3: take ssh-connetion of QA-server => run the command:
          
         (a) sudo apt-get update -y (hit enter)
         (b) Install tomcat9 server:sudo apt-get install -y tomcat9 (hit enter)
         (c) Install tomcat9 admin : sudo apt-get install -y tomcat9-admin (hit enter)
         (d) Now check tomcat9 installed or not: copy public IP of QA-server and paste in browser with :8080
         (e) Now in QA-server gitbash terminal go to cd /etc/tomcat9 (hit enter) => sudo nano tomcat-users.xml (hit enter) 
         (f) Here copy this : <user username="training" password="nawaz123" roles="manager-script,manager-status,manager-gui"/> then => Cntrl+O=>enter=>Cntrl+X
         (g) Now : cd ~ and comeback to home and again restart tomcat9 : sudo systemctl restart tomcat9 (hit enter)

### Step-4: Now copy public IP of QA-server and paste in browser with :8080/manager 

         (a) Here fill the credentials : username - training & password - nawaz123 (OK) ==> tomcat9 GUI Page will open on the browser
         
### Step-5: Now instance3 - Prod-server (AMI-Ubuntu 20.04 LTS,t2-micro,), security group (ssh-port22, and TCP custome-port8080)

         (a) sudo apt-get update -y (hit enter)
         (b) Install tomcat9 server:sudo apt-get install -y tomcat9 (hit enter)
         (c) Install tomcat9 admin : sudo apt-get install -y tomcat9-admin (hit enter)
         (d) Now check tomcat9 installed or not: copy public IP of Prod-server and paste in browser with :8080
         (e) Now in Prod-server gitbash terminal go to cd /etc/tomcat9 (hit enter) => sudo nano tomcat-users.xml (hit enter)
         (f) Here copy this : <user username="learning" password="nawaz123" roles="manager-script,manager-status,manager-gui"/> then => Cntrl+O=>enter=>Cntrl+X
         (g) Now : cd ~ and comeback to home and again restart tomcat9 : sudo systemctl restart tomcat9 (hit enter)
         (h) Now copy public IP of QA-server and paste in browser with :8080/manager  
         (i) Here fill the credentials : username - learning & password - nawaz123 (OK) ==> tomcat9 GUI Page will open on the browse 


### Step-6: Now Go to JENKINS GUI dashboard:

         (a) Create Job: ==> Item_name: development, ==> select:Freestyle Project =>OK
         (b) Description: Development Project ==> Source Code Management:select:Git ==> Repository URL:https://github.com/samiuddin-imaad/maven
         (c) Build steps:click on:Add Build steps:select:envoke top-level Maven targets ==>Goals:Package ==>Apply & save
         (d) Now Go to dashboard:development job is ready ==> click on:run the job (it will start running and pulling the code from GitHub)
         (e) Now after completeion Green tick will reflect before the job_name development
         (f) To see the output : click on Last success No.#1:select=>console output

### Step-7: Now copy the path from output console: /home/ubuntu/.jenkins/workspace/development/webapp/target/ and open one more new ssh-connection terminal for Dev-server ==>establish connection.

         (a) paste the path: cd /home/ubuntu/.jenkins/workspace/development/webapp/target (hit enter)
         (b) Now do:ls => maven-archive surefire webapp webapp.war
         

### Step-8: Go to JENKINS GUI Dashboard ==> Manage Jenkins ==> Plugins ==> Available Plugins ==> search for "Deploy to container" ==>select it ==> click on: install ==> Go back to the top page


        (a) Go to JENKINS GUI Dashboard ==> development ==> configure ==> Post Build ==> click on: Add post build action ==> select: Deploy war/ear to container ==> WAR/EAR files: **/*.war ==> Context path: QA-env ==> Add container=>select:tomcat9x Remote ==> tomcat URL:http//private IP of QA-server:8080

        (b) Click on: Add credentials ==> jenkins ==>username:training ,password:nawaz123 => click on:Add ==> credentials: click on:arrow:select:training/****
        (c) Apply & save
        (d) Now Go to development ==> run again & refresh the jenkins page 
        (e) To check it just Go to the QA-server tomcat GUI and refresh the page : will display in Application column => Path => /QA-env
        (f) Now To see the Deployment result copy the Public IP of QA-server and paste in new browser with :8080/QA-env (enter) : O/P = Hello World


# CONTINUOUS DEPLOYMENT & QA/QC along with TESTING :==============


### Step-9: Now Testing: Got to Jenkins dashboard :

        (a) Create Job: ==> Item_name: testing, ==> select:Freestyle Project =>OK 
        (b) Description: testing Project ==> Source Code Management:select:Git ==> Repository URL:https://github.com/samiuddin-imaad/test-repo
        (c) Build steps:click on:Add Build steps:select:execute shell ==> Command: echo "Test is passed"==> Apply & save
        (d) Now Go to testing job ==> run the job => refresh the jenkins page ==> see the console output ==> SUCCESS


# CONTINUOUS DELIVERY (CD):=======


### Step-10: Production:  Here we will integrate Testing with Production via jenkins

        (a) Go to Jenkins dashboard ==> Manage Jenkins ==> Plugins ==> Available Plugins ==> search for "copy Artifact" select and install

        (b) Go to Jenkins dashboard ==> development ==> configure => Add post build actions: select: Build other project ==> Post-build Actions => Projects to build : testing Project => trigger only if build is stable ==> Add and save ==> run the development job.

        (c) Go to Jenkins dashboard ==> development ==> configure => Add post build actions: select:Archive the artifacts => Files to archive: **/*.war => apply & save.

        (d) Go to Jenkins dashboard ==> testing project ==> configure ==> Add build step ==> select: copy artifacts from another project ==> Project name:development => which build: Latest successful build ==> untick: fingerprint 

        (e) Add post build action: Deploy WAR/EAR files to a container ==> WAR/EAR files: **/*.war ==> context path: Prod-env => Add container:tomcat9.x Remote ==> Credentials==> select: learning/**** ==> tomcat URL: http://Private IP of Prod-server:8080 ==> Apply & save

        (f) Run the developmant job & refresh the jenkins page : Once get successful run 


        (g) Go to Prod-server ==> copy public IP ==> Paste on any browser with :8080/Prod-env : O/P = Hello World 


 
<img width="440" alt="jenkins-1" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/93fcc5eb-c19c-4d21-897d-528c320ae5ee">


<img width="644" alt="jenkins-2" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/9aabf07a-ed5c-49fb-9cae-e01e9ad424ea">


<<<<<<<<===<img width="647" alt="jenkins-3" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/231f6ed6-f478-45d5-8a1c-3093b10cfd89">

<img width="932" alt="jenkins-4" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/b3064ad2-8815-4163-ab07-ff1879b40269">


<img width="941" alt="jenkins-5" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/f2dbc285-2cc9-4eb6-b048-09a9aac8186e">

<img width="932" alt="jenkins-6" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/e396853a-2ff6-4aa5-9333-06d999174e2d">

<img width="656" alt="jenkins-7" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/7369bffd-9cb4-4019-b901-c8ffd520fa2c">


<img width="359" alt="jenkins-8" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/6cf17110-068a-41eb-99a9-6a5d0c5a358b">


<img width="937" alt="jenkins-10" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/db029278-1204-4d30-87b7-c9837202e2c6">


<img width="948" alt="jenkins-9" src="https://github.com/Shahnawaz-yousuf/JENKINS_Free_Style_Project/assets/146080117/5e0241e5-94d8-4b27-9fb7-ee2cacf98309">


========>>>> # CONGRATULATIONS YOU HAVE DONE SUCCESSFULLY <<<<<============ 




