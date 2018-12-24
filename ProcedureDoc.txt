Kubernetes Procedure Document
-----------------------------
Github repository [Read this first]
Kubernetes releases minor version updates of its distribution every 3 months.The changes are often very minor, the API is very stable. Often API versions like v1betaX change to v1betaX+1 or to v1 (stable).

All the scripts you can find in the repository should work with the latest version of Kubernetes, if you have any issues, contact me through one of the channels listed below

Questions?
Send me a message

Use Q&A

**Download Kubectl**
Linux:[https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/linux/amd64/kubectl]

MacOS:[https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/darwin/amd64/kubectl]

Windows:[https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/windows/amd64/kubectl.exe]

Or use a packaged version for your OS: see https://kubernetes.io/docs/tasks/tools/install-kubectl/

**Minikube**
Project URL:[https://github.com/kubernetes/minikube]

Latest Release and download instructions: [https://github.com/kubernetes/minikube/releases]

VirtualBox: http://www.virtualbox.org

>kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
>kubectl expose deployment hello-minikube --type=NodePort

minikube service hello-minikube --url

<open a browser and go to that url>

**Kops**
Project URL [https://github.com/kubernetes/kops]

**Choose for subdomain hosting**
Enter the AWS nameservers given to you in route53 as nameservers for the subdomain

http://www.dot.tk/ provides a free .tk domain name you can use and you can point it to the amazon AWS nameservers

Namecheap.com often has promotions for tld’s like .co for just a couple of bucks


Kubernetes from scratch
------------------------
You can setup your cluster manually from scratch

If you’re planning to deploy on AWS / Google / Azure, use the tools that are fit for these platforms

If you have an unsupported cloud platform, and you still want Kubernetes, you can install it manually

CoreOS + Kubernetes: ###a href="https://coreos.com/kubernetes/docs/latest/getting-started.html">https://coreos.com/kubernetes/docs/latest/getting-started.html

doc: [https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/]

Docker
You can download Docker Engine for:

Windows: https://docs.docker.com/engine/installation/windows/

MacOS: https://docs.docker.com/engine/installation/mac/

Linux: https://docs.docker.com/engine/installation/linux/


Cluster Commands
----------------
> kops create cluster --name=kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --zones=us-east-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.majumder-jyotisekhar.me
> kops edit cluster kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state

> kops update cluster kubernetes.majumder-jyotisekhar.me --yes --state=s3://jm-kops-state

> kops delete cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state

> kops delete cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --yes
> kops get clusters --state=s3://jm-kops-state

 * validate cluster: kops validate cluster --state=s3://jm-kops-state
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.kubernetes.majumder-jyotisekhar.me


new ssh-key upload
------------------
> kops create secret sshpublickey admin -i ~/.ssh/beast_rsa.pub --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state
>kops delete secret SSHPublicKey admin1 a2:58:eb:30:5b:41:83:4a:c6:39:d4:33:65:5d:ec:9c --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state
>kops update cluster kubernetes.majumder-jyotisekhar.me --yes --state=s3://jm-kops-state
>kops rolling-update cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --yes


[ https://www.devops.buzz/kubernetes-start-and-stop-cluster-using-kops/ ]

> aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,PrivateIpAddress,PublicIpAddress,Tags[?Key==`Name`].Value[]]' --output json | tr -d '\n[] "' | perl -pe 's/i-/\ni-/g' | tr ',' '\t' | sed -e 's/null/None/g' | grep '^i-' | column -t


deploy hello-minikube app in ec2 k8s cluster
---------------------------------------------
> kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
> kubectl expose deployment hello-minikube --type=NodePort
> kubectl get services


docker image/registry/push n deploy app in minikube
---------------------------------------------------
> cd /home/opc/jm-docker-demo
> docker build -t jmajumde/nodejs-helloworld-v1 .
> docker image list
> docker run -p 3000:3000 -it 674b6a3b4d71  -> to test

push the above image 

> docker login
> docker tag imageid jmajumde/nodejs-helloworld-v1 ===> not required, if -t is used in docker build
> docker push jmajumde/nodejs-helloworld-v1

Now deploy the first-app helloworld in minikube
------------------------------------------------
> minikube status

You can point your docker client to the VM's docker daemon by running. So that everytime kubectl does not try to pull the image from remote registry which speeds up pod provisioning. We need to mention "**imagePullPolicy: IfNotPresent**" in the pod provisioning yml specification. 

eval $(minikube docker-env)

But didnt work for me.

Alternatively we can create a secret for using remote registry like below, and mention "__imagePullSecrets:__" flag in the pod provisioning yml specification. This needs to be done for each pod that is using a private registry.
However, setting of this field can be automated by setting the imagePullSecrets in a serviceAccount resource. Check Add ImagePullSecrets to a Service Account [ttps://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account]for detailed instructions.

> kubectl create secret docker-registry mydockerhubregistry --docker-server=https://hub.docker.com --docker-username=<user> --docker-password='<passwd>' --docker-email="<email>"
> kubectl get serviceaccounts
> kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "mydockerhubregistry"}]}'
> kubectl get serviceaccounts default -o json

> kubectl create -f ~/mywork/k8s-nonsense/first-app/helloworld.yml
> kubectl describe pod nodehelloworld.example.com

==== But this also does not work for me ====

to test the pod and the service initially we can test it by port-forward
> kubectl port-forward nodehelloworld.example.com 8081:3000

But its better to create a service for the pod. To create a service in minikube 
> kubectl expose pod nodehelloworld.example.com --type=NodePort --name nodehelloworld-service
> minikube service nodehelloworld-service --url


Deploy service witl ELB in aws
-------------------------------

ClusterIp    -> Expose service through **k8s cluster** with ip/name:port
NodePort     -> Expose service through **Internal network VM's** also external to k8s ip/name:port
LoadBalancer -> Expose service through **External world** or whatever you defined in your LB.

> kops validate cluster kubernetes.majumder-jyotisekhar.me --state s3://jm-kops-state
> kubectl create -f /vagrant_data/first-app/helloworld.yml
> kubectl create -f /vagrant_data/first-app/helloworld-service.yml
> kubectl describe services helloworld-service


Replication Controller
----------------------
> kubectl create -f data/replication-controller/helloworld-repl-crtl.yaml
> kubectl describe ReplicationController/helloworld-controller
> kubctl get pods

If there are mutiple replicas of same pod and then one pod is deleted [kuctl delete pod <name>] then repliocation controller automatically deploys another pod which delivers the HA feature. 

Pods can be horizontally scale (up/down) using replicas like below
> kubectl scale --replicas=4 -f data/replication-controller/helloworld-repl-crtl.yaml
> kubectl get rc
> kubectl scale --replicas=1 rc/helloworld-controller

Using replication controller pods can be scaled horizontically _ONLY_ if the pods are _STATELESS_, in case of _STATEFUL_ pods we cannot scale the pods like this. 




