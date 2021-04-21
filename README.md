# Kubernetes-Jenkins-deployment
This document is for personal use. If you want to use it, you are welcome to follow the same steps.

Deployment using Declarative object configurations.

create three yaml files: namespace.yaml, deployment.yaml and service.yaml 

# namespace.yaml file:
1.    you need to create a target namespace. The required yaml has mentioned below.
  
---
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins-deployment
  labels:
   app: jenkins-image
  annotations:
   type: test

 1. apiversion is chosen carefully on per object requirement from the followwing link.
 https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-apiversion-definition-guide.html
 
 2. in metadata the requested namespace name is mentioned and further tagged with the appropriate label.
 
 3. This script can be run using two commands:
    i. kubectl create -f namespace.yam           # Creating a fresh namespace
    ii. kubectl apply -f namespace.yamk          
    
# Jenkins. Yaml file.

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins-deployment
  labels:
    app: jenkins-image
  annotations:
     monitoring: "true"
spec:
  selector:
    matchLabels:
      app: jenkins-image
  replicas: 2
  template:
    metadata:
      labels:
        app: jenkins-image
    spec:
       containers:
        - image: jenkins:latest
          name:  jenkins
          ports:
          - containerPort: 80
          resources:
      
 1. The deployment kind  along with relevant apiversion is chosen from the same above link.
 2. metadata add the unique identities of the application and add the same namespace as created from namespace.yaml script
 3. spec: is very important part of this file. It has three important parameters to define: template,replicas and selector
    i. replicas defines the number of pods need to be created of jenkins container
    ii. template gives the name of the pods, the image, and ports. 
    iii. targetPort is container internal port and port 80 is external port
    iv: image parameter is extracting only the latest image from docker hub. Remember you can create your own image by writing Dockfile.
        you could use this file for creating image and eventually building a container. here I am polling it from docker hub repository

 # service.yaml file
---
apiVersion: v1
kind: Service
metadata:
    name: jenkins-deployment
    namespace: jenkins-deployment
spec:
    ports:
     - name: http 
       nodePort: 30600
       port : 80
       protocol: TCP
       targetPort: 80
    selector:
       app: jenkins
    type: NodePort 
  
1. Api is chosen from the aformentioned link. for writing serbice
2. Here namespace is same as mentioned in namespace scripts
3. in the Spec: portion. nodeport will be external accessible port. it will be replicated on port 80. Poryocol will be TCP and Continaer    inner port is 80 port, which is target port.
4. I chose NodePort routing method as I have build my cluster on local VMs. In Cloud Loadbalancer is used for which you also buy a domain to point it out

After deploying these files your jenkkins will be ready

NOte : for Pods use Fasboard or switch your context to watch out the process. 

 
