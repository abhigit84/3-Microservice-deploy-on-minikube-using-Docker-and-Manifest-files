
Github used:-
https://github.com/abhigit84/3-Microservice-deploy-on-minikube-using-Docker-and-Manifest-files.git

Objective:-
This is a 3 microservice deployment project using manifest or yaml files and dockerfiles on minikube.


Region:us-west-1

OS used:Amazon linux ami 2,15 gb
t2.medium(for kubernetes to run this is minimum)

AWS Console:-

1)sudo su
yum update -y

2)Install Docker:

amazon-linux-extras install docker

yum install docker -y

systemctl start docker

systemctl enable docker


3)Install Conntrack and Minikube:(conntrack is base for minikube to work)

yum install conntrack -y

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

4)Start your MINIKUBE:-
/usr/local/bin/minikube start --force --driver=docker

5)Install Maven and Java

MAVEN:

cd /opt/
wget http://mirrors.estointernet.in/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar xvzf apache-maven-3.6.3-bin.tar.gz
vi /etc/profile.d/maven.sh [ And add the below both lines ]
export MAVEN_HOME=/opt/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin

GIT:

yum install git -y
JAVA:

yum install java -y

6)Install kubectl:

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl

chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short –client

7)Clone the repo(note u are in /opt and u have to clone here only)

https://github.com/abhigit84/3-Microservice-deploy-on-minikube-using-Docker-and-Manifest-files.git

3 microservices(u can say application or modules) in this Project(every microservice having its own dockerfile)so 3 jars in total,1 for each microservice.

cd kubernetes_java_deployment

cd shopfront

mvn clean install -DskipTests

docker images --no image of microservice there ...only minikube image there.

docker build -t abhinoi84/shopfront:latest .

docker login

docker push abhinoi84/shopfront:latest

kubectl apply -f shopfront-service.yaml

if u face errors start minikube using minikube start command in step way above.

kubectl get pods--u will see pod created.

Hit the below command to start the kubernetes
dashboard in EC2:

/usr/local/bin/minikube dashboard

Now on onother session of same ec2 connect(in new window..please keep the existing session open only on which u were doing everything)

Open the EC2 in new window and set the PROXY
sudo su
kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'

This command whatever incoming request will be there it will be redirecting to local host as each microservice will run in localhost.
0.0.0.0 is localhost and 127.0.0.0 is also localhost

On ec2 instance open security group and open all traffic for ipv4.

you can hit this on browser:-

http://<EC2-IP>:8001/api/v1/namespaces/kubernetes-da
shboard/services/http:kubernetes-dashboard:/proxy/#/po
d?namespace=default

on minikube dashboard:
u can see in deployments-edit-same yml/json file.
go to pods(click on : and go to exec or go inside container)
you can see app.jar after doing ls -lrta as in Dockerfile is adding to current directory of container(here service container)

Hit the below command for each service in
different console of EC2 after stopping that /usr/local/bin/minikube dashboard by ctrl+c .
kubectl port-forward --address 0.0.0.0 svc/shopfront 8080:8010

Repeat above steps for each left 2 microservice.

cd productcatalogue/
mvn clean install -DskipTests
docker build -t abhinoi84/productcatalogue:latest .
docker push abhinoi84/productcatalogue:latest
and repeat above steps in same order.

cd stockmanager/
mvn clean install -DskipTests
docker build -t abhinoi84/stockmanager:latest .
docker push abhinoi84/stockmanager:latest
and repeat above steps in same order.


To see in browser the microservice productcatalog application:

kubectl port-forward --address 0.0.0.0 svc/productcatalogue 8090:8020

http://54.183.85.189:8090/products


To see in browser the microservice stockmanager application:

kubectl port-forward --address 0.0.0.0 svc/stockmanager 9008:8030

http://54.183.85.189:9008/stocks

To see in browser the microservice shopfront application:

kubectl port-forward --address 0.0.0.0 svc/shopfront 8080:8010

http://54.183.85.189:


Note:in 1 yaml file only both kinds deployment and service written here.so 1 yaml file for each microservice residning in kubernetes folder
1 dockerfile for each microservice.
similarly we will have 1 jenkins file for each microservice but this is not pipeline so jenkins not used in this project.
kubectl port-forward --address 0.0.0.0 svc/stockmanager 9008:8030 this is the way to forward to localhost port 9008 from pod port 8030 and access it on ur local machine browser.
Host port(ec2 or localhost):container port
dont get consfused how its printed on kubectl get svc other way around.
















