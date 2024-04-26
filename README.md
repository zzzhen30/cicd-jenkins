# Setup Virtual Environment

```python
conda create -n jenkins-env python=3.10 -y
conda activate jenkins-env
pip install -r requirements.txt   / pip install -r src/requirements.txt 
pip install .    /    pip install src/.   # installing prediction_model

# check whether the model is installed 
python
import prediction_model
from prediction_model.training_pipeline import perform_training  # import a function
perform_training()  # it will show  ' Model has been saved under the name classification.pkl '
```

# build the FASTAPI 
main.py
``` where to check: 
http://localhost:8005/docs
```


# Test the FASTAPI at http://localhost:8005/docs 

```json

{
  "Gender": "Male",
  "Married": "No",
  "Dependents": "2",
  "Education": "Graduate",
  "Self_Employed": "No",
  "ApplicantIncome": 5849,
  "CoapplicantIncome": 0,
  "LoanAmount": 1000,
  "Loan_Amount_Term": 1,
  "Credit_History": "1.0",
  "Property_Area": "Rural"
}

# Test in Postman 
http://localhost:8005/prediction_api



```

# Docker Commands
```
docker build -t loan_pred:v1 .        ## build dockerimage, run cmd in the directory where dockerfile is present ; dot "." in the cmd, refer to current directory 

docker images            ## check whether image is built

docker build -t manifoldailearning/cicd:latest .     ## for instructional purpose, push this to docker help
docker build -t zzzhen1130/cicd:latest .    #

docker push manifoldailearning/cicd:latest    #  pushing image to docker registry, change it to own account repository 
docker push zzzhen1130/cicd:latest     # modify to push to my own account repository named cicd


docker run -d -it --name modelv1 -p 8005:8005 manifoldailearning/cicd:latest bash    ## build container modelv1.  '-it' means interactive mode , change it to my own repository 
docker run -d -it --name modelv1 -p 8005:8005 zzzhen1130/cicd:latest bash  

docker exec modelv1 python prediction_model/training_pipeline.py   # inside modelv1 docker container, excute python 

docker exec modelv1 pytest -v --junitxml TestResults.xml --cache-clear  # once training complete, excute test, "junitxml" mean nomatter what result it is , save it as xml format

docker cp modelv1:/code/src/TestResults.xml .  # save test result to local directory 

docker exec -d -w /code modelv1 python main.py  # run the FASTAPI application in docker container , check http://localhost:8005/ 

docker exec -d -w /code modelv1 uvicorn main:app --proxy-headers --host 0.0.0.0 --port 8005


```

# AWS EC2
1. build a instance 
select "instance" 
select "launch instance" -- a new instance 
give a name to instance 
select operating system "ubutun"
for instance type, select "medium"
for key pair, create new key pair, named it as "aws-mlops", other leave as default 
Download this key aws-mlops.pem, them move it to the MLOPS (OR you determine) folder
allow SSH traffic, HTTPS traffic, HTTP traffic
Storage: 30 GBS
Launch a new instance 

2. Click Connect to instance 
--- elect SSH client --- connect through SSH client 
--- Set permission: In terminal, MLOPS folder(where the key stored) copy and paste,  
```bash    
    chmod 400 "aws-mlops.pem" 
```   
-- After set permission, in Example, copy paste (it will be different everytime you stop and run the instance again)
```bash    
    ssh -i "aws-mlops.pem" ubuntu@ec2-54-237-90-231.compute-1.amazonaws.com
```
    yes
Then it will connect to ubuntun OS in CE2 / AWS cloud


# Installation of Jenkins

```bash
# the instruction is from https://www.jenkins.io/doc/book/installing/linux/#debianubuntu
# note below maybe outdated, check above web for the latest version 

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins


# install java (so jenkins can run)
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version

# validate jenkins is running 
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

# after validate jenkins is running and access jenkins, we need to be able to open the port 8080
```
1. to go to EC2, the instance running 
Select the instance
Click 'security' 
Under 'security group', click the key
Scroll down, click 'edit inbound rule'
'add rule' ---- select 'All TCP', --- select source type 'Anywhere IPV4'
Create one more rule ' add rule' --- 'All TCP' , --- 'Anywhere IPV6'  

Above allowing access to all the ports

Go back to instance
Click 'Networking' 
You will see IPv4 address 54.157.181.152
In browser, type 54.157.181.152:8080 
You will see a login page of "Unlock Jenkins",  /var/lib/jenkins/secrets/initialAdminPassword



Then your jenkins set up OK  



```

``


`
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
`

```

# Setup Docker engine 
1. search Install Docker in Ubutun in google 
Below detail is from 
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository 

```bash
# Add Docker's official GPG key, Note: use the latest code from above website:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources,Note: use the latest code from above website:
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

```


```bash 
# install docker packages,  use the latest code from above website:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# to run the docker cmd, need to add sudo -- permission issue
sudo docker ps

# provide permission to jenkins user to work on Docker 
sudo usermod -a -G docker jenkins
sudo usermod -a -G docker $USER

# after this, when you use docker ps, still got error: 
docker ps
```
```
 permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.45/containers/json": dial unix /var/run/docker.sock: connect: permission denied

```
```
# need to restart CE2 
-- back to instance 
-- instant state: reboot 
```
```bash
# then check permission again 
  docker ps
# CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES ---- it works 
``` 


# setup Github repository for tracking changes 
1. Github account -- create new repository , named as ci-cd-jenkins
2. Open a terminal in local system 
- at path ~/develop/MLOPS  (the main folder)
- copy and paste the link in Quick setup from the new repo 
https://github.com/zzzhen30/cicd-jenkins.git 
```bash 
git clone https://github.com/zzzhen30/cicd-jenkins.git
```
a new folder named cicd-jenkins will show
/home/zzz/develop/MLOPS/cicd-jenkins 

- copy all files from folder ml-ci-cd-jenkins to cicd-jenkins 

# Admin password Jenkins

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

# Payload URL format in github repo webhook

`http://<public-ipv4 address>:8080/github-webhook/` #replace it with ur own public-ipv4 address

```

# Additional Improvements

docker remove $(docker ps -a -q)
docker images --format "{{.ID}} {{.CreatedAt}}" | sort -rk 2 | awk 'NR==1{print $1}'



# Create Stage Branch
`git checkout -b staging`
`git push `


# modify it for demostration purpose 

