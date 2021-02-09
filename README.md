Project Name :
Function as a Service (FaaS) for Internet of Things (IoT)
Function as a Service is cutting-edge service model that has developed with the current advancement of cloud computing. Cloud functions allow custom code to be executed in response to an event. In most cases, developers need only worry about their actual code, as event queuing, underlaying infrastructure, dynamic scaling, and dispatching are all handled by the cloud provider. FaaS is a great fit for data and data processing in the Internet of things. The goal of the project is to study the existing open-source platform that enables FaaS, and build a proof of concept using the IOT LORA platform of EURECOM

Keywords
FaaS, Cloud, IOT, Edge computing .

Table of contents
General info
Overview
Installation
Usage
Authors
Resources
General info
In order to have an idea about Faas platforms, you should read this files.

https://dbermbach.github.io/publications/2020-icfc-tinyfaas.pd
https://www.lebigdata.fr/faas-definition
Overview
To get started with serverless computing, you only need three ingredients:

A Kubernetes cluster
Kubeless
Some code.
Once you have verified that your Kubernetes cluster is running, you will need to perform the following steps:

Start the Kubernetes Dashboard on port 8080
Install Kubeless
Write a function
Register the function with Kubeless
Call the function
Installation
Step 1 : The installation of Kubeless :
Brief idea about Kubeless :

Kubeless is an application designed to be
deployed on top of a Kubernetes cluster.
It accepts commands to register, delete,
and list functions that you would
like to run.

This below is the case of deploying kubeless to a Kubernetes cluster with RBAC available ( to have more details about differents cases of Kubeless installation, you can see here ).

curl -L https://github.com/kubeless/kubeless/releases/download/0.0.20/kubeless_linux-amd64.zip > kubeless.zip
unzip kubeless.zip
sudo cp bundles/kubeless_linux-amd64/kubeless /usr/local/bin/
Step 2 : Kubeless deployment in our Kubernetees cluster :
In order to create a kubeless namespace , we used these commands :

$ export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
$ kubectl create ns kubeless
$ kubectl create -f https://github.com/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
To make sure that the deployment of kubeless was a success , you should run these commands:

$ kubectl get pods -n kubeless
$ kubectl get deployment -n kubeless
$ kubectl get customresourcedefinition
Usage
Step 1 : Write a Hello function :
Now Kubeless is running, which means you can forget about containers, VMs, Kubernetes, etc… All you need is some code.
So , in the next steps you will try a sample function in order to test your application is working .
In this example, you will use Python runtime to write a Hello function.

def hello(event, context):
  print event
  return event['data']
Note : Functions in Kubeless have the same format regardless of the language of the function or the event source.
In general, every function:

Receives an object event as their first parameter. This parameter includes all the information regarding the event source. In particular, the key ‘data’ should contain the body of the function request.
Receives a second object context with general information about the function.
Returns a string/object that will be used as response for the caller.
Now, you can save your function where you want in order to register it with Kubeless ( for example you can save it at ~/Downloads/test.py).

Step 2 : Register the function with Kubeless :
It’s all easily done using a reasonably simple command called kubeless function deploy. Here it is( supposing you saved the function at ~/Downloads/test.py):

$ kubeless function deploy hello --runtime python2.7 --handler test.hello --from-file ~/Downloads/test.py
Registering the function with Kubeless involves telling Kubeless these bits of information :

kubeless function deploy hello : tells Kubeless to register a new function named hello . The function will be accessible over the Web using this name. Note that this doesn’t need to be the same as the function name used inside the code (we’ll specify that a little further along using the --handler option).
–trigger-http tells Kubeless that the function will be invoked over HTTP. It’s also possible to trigger the function in other ways, but that is not covered here.
–runtime python2.7 tells Kubeless to use Python 2.7 to execute the code.
–handler test-hello : This specifies the file and the exposed function that will be used when receiving requests. In this example we are using the function hello from the file test.py.
–from-file ~/Downloads/test.py : This is the file containing the function code. It is supported to specify a zip file as far as it doesn’t exceed the maximum size for an etcd entry (1 MB).
Step 3 : Checking :
Once the command has completed executing, check that it’s registered with Kubeless using the kubeless function ls command, as shown below:

$ kubeless function ls
Step 4 : Call the function :
Calling the function from the Kubeless CLI :

$ kubeless function call hello --data 'Hello world!'
--> Hello world!
$ kubeless function call hello --data 'Hi user!'
--> Hi user!
Step 5 : Expose the Kubeless function :
By default, a deployed function will be matched to a Kubernetes service using ClusterIP as the service. That means that the function is not exposed publicly. Because of that, you must run the kubeless trigger http command that can make the function publicly available.

Deploy the function with Kubeless CLI :
Once you have a Ingress Controller running you should be able to start deploying functions and expose them publicly. First deploy the hello function:

$ cd examples
$ kubeless function deploy get-python \
                    --runtime python2.7 \
                    --handler helloget.foo \
                    --from-file python/helloget.py
$ kubectl get po
$ kubectl get svc
Expose the function /
In order to expose the function, it is necessary to create a HTTP Trigger object. The Kubeless CLI provides the commands required to do so:

kubeless trigger http create hello --function-name hello
This command will create an ingress object:

$ kubectl get ing
Kubeless creates a default hostname in form of …nip.io. Alternatively, you can provide a real hostname with --hostname flag or use a different --path like this:

$ kubeless trigger http create get-python --function-name get-python --path echo --hostname example.com
$ kubectl get ing
But you have to make sure your hostname is configured properly.

Now, you can test the created HTTP trigger with the following command:

curl --data '{"hello":"world"}' localhost:8080/api/v1/namespaces/default/services/hello:8080/proxy/ --header "Content-Type:application/json"
Step 6 : Develop an application using FaaS :
Now, we will develop an App Client (Python) which:

 Takes as entry the date of birth.
Sends the request to the Kubeless function which:

 Takes as input the date of birth
 Output the age
Writing the function :
Using Python:

from datetime import date 

def calculateAge(event ,context):
    print (event)
    today = date.today() 
    age = today.year - event.year - ((today.month, today.day) <  (event.month, event.day)) 
    return age
You can save the code at ~/Downloads/test.py.

Registering the function with Kubeless :
$ kubeless function deploy age --runtime python2.7 --from-file ~/Downloads/test.py --handler test.age 
To show all functions, you can run this command:

$ kubeless function ls 
Create a http trigger to age function :
$ kubeless trigger http create age --function-name age
Call the function :
$ kubeless function call age --data '1997,12,7'
Step 9 : Clean up :
You can delete the functions and uninstall Kubeless:

$ kubeless function delete hello
$ kubeless function delete age
$ kubeless function ls
$ kubectl delete -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
Authors
Name	Email
Ksentini Adlen	
Trabelsi Mohamed Idriss	[email protected]ecom.fr
Belhadj Mohamed Monsef	[email protected]
Rached Haythem	[email protected]
Resources
Documentation
see Kubeless Installation
see Get Started with Serverless Computing on Kubernetes with Minikube and Kubeless
see kubeless-http-triggers
