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
```
kops create cluster --name=kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --zones=us-east-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.majumder-jyotisekhar.me
kops edit cluster kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state

kops update cluster kubernetes.majumder-jyotisekhar.me --yes --state=s3://jm-kops-state

kops delete cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state


kops delete cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --yes
kops get clusters --state=s3://jm-kops-state
```
 * validate cluster: kops validate cluster --state=s3://jm-kops-state
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.kubernetes.majumder-jyotisekhar.me


New ssh-key upload
------------------
```
kops create secret sshpublickey admin -i ~/.ssh/beast_rsa.pub --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state

kops delete secret SSHPublicKey admin1 a2:58:eb:30:5b:41:83:4a:c6:39:d4:33:65:5d:ec:9c --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state

kops update cluster kubernetes.majumder-jyotisekhar.me --yes --state=s3://jm-kops-state

kops rolling-update cluster --name kubernetes.majumder-jyotisekhar.me --state=s3://jm-kops-state --yes

```
[ https://www.devops.buzz/kubernetes-start-and-stop-cluster-using-kops/ ]

> aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType,PrivateIpAddress,PublicIpAddress,Tags[?Key==`Name`].Value[]]' --output json | tr -d '\n[] "' | perl -pe 's/i-/\ni-/g' | tr ',' '\t' | sed -e 's/null/None/g' | grep '^i-' | column -t


deploy hello-minikube app in ec2 k8s cluster
---------------------------------------------
```
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
kubectl get services
```

docker image/registry/push n deploy app in minikube
---------------------------------------------------
```
cd /home/opc/jm-docker-demo
docker build -t jmajumde/nodejs-helloworld-v1 .
docker image list
docker run -p 3000:3000 -it 674b6a3b4d71  -> to test
```

push the above image 

```
docker login
docker tag imageid jmajumde/nodejs-helloworld-v1 ===> not required, if -t is used in docker build
docker push jmajumde/nodejs-helloworld-v1
```
Now deploy the first-app helloworld in minikube
------------------------------------------------
> minikube status

You can point your docker client to the VM's docker daemon by running. So that everytime kubectl does not try to pull the image from remote registry which speeds up pod provisioning. We need to mention "**imagePullPolicy: IfNotPresent**" in the pod provisioning yml specification. 

> eval $(minikube docker-env)

But didnt work for me.

Alternatively we can create a secret for using remote registry like below, and mention "__imagePullSecrets:__" flag in the pod provisioning yml specification. This needs to be done for each pod that is using a private registry.
However, setting of this field can be automated by setting the imagePullSecrets in a serviceAccount resource. Check Add ImagePullSecrets to a Service Account [ttps://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account]for detailed instructions.
```
kubectl create secret docker-registry mydockerhubregistry --docker-server=https://hub.docker.com --docker-username=<user> --docker-password='<passwd>' --docker-email="<email>"
kubectl get serviceaccounts
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "mydockerhubregistry"}]}'
kubectl get serviceaccounts default -o json

kubectl create -f ~/mywork/k8s-nonsense/first-app/helloworld.yml
kubectl describe pod nodehelloworld.example.com
```

Minikube works fine with docker hub registry. But somehow trying to poiting to local registry even with "imagePullPolicy" settings does not. This is a TODO item to try out later

to test the pod and the service initially we can test it by port-forward
> kubectl port-forward nodehelloworld.example.com 8081:3000

But its better to create a service for the pod. To create a service in minikube 
> kubectl expose pod nodehelloworld.example.com --type=NodePort --name nodehelloworld-service

service url 

> minikube service nodehelloworld-service --url


Deploy service with ELB in AWS
-------------------------------

ClusterIp    -> Expose service through **k8s cluster** with ip/name:port
NodePort     -> Expose service through **Internal network VM's** also external to k8s ip/name:port
LoadBalancer -> Expose service through **External world** or whatever you defined in your LB.

```
kops validate cluster kubernetes.majumder-jyotisekhar.me --state s3://jm-kops-state
kubectl create -f /vagrant_data/first-app/helloworld.yml
kubectl create -f /vagrant_data/first-app/helloworld-service.yml
kubectl describe services helloworld-service
```

##### Stop the running cluster in Dev/Test scenario
There is no straught forward way to stop the running K8 cluster in aws using kops. While playing arounf k8 in aws using kops, if we keep the cluster instances running then aws incure cost for that. So to avid that we can terminate the cluster without actually delete it. This way the resources will be terminated, but the service would be still configured. Modify the instance group for the cluster master node first

```
kops get ig --state s3://jm-kops-state
kops edit ig master-us-east-1a --state s3://jm-kops-state
```
> Modify maxSize, minSize parameter to 0 (zero)

Do the same for the nodes
> kops edit ig nodes --state s3://jm-kops-state

Let’s save our new config. This step will not “run” anything yet, we are just committing our cluster’s config file.
> kops update cluster --yes --state s3://jm-kops-state

Now update the cluster in rolling-update fasion

```
# kops rolling-update cluster --name kubernetes.majumder-jyotisekhar.me --state s3://jm-kops-
state --cloudonly --force --yes
vagrant@ubuntu-xenial:/vagrant_data$ kops rolling-update cluster --name kubernetes.majumder-jyotisekhar.me --state s3://jm-kops-state --cloudonly --force --yes
NAME			STATUS	NEEDUPDATE	READY	MIN	MAX
master-us-east-1a	Ready	0		0	0	0
nodes			Ready	0		2	2	2
W1224 22:34:49.265207   11911 instancegroups.go:152] Not draining cluster nodes as 'cloudonly' flag is set.
I1224 22:34:49.265385   11911 instancegroups.go:280] Stopping instance "i-0067798ffb5227677", in group "nodes.kubernetes.majumder-jyotisekhar.me" (this may take a while).
W1224 22:38:49.659046   11911 instancegroups.go:184] Not validating cluster as cloudonly flag is set.
W1224 22:38:49.659120   11911 instancegroups.go:152] Not draining cluster nodes as 'cloudonly' flag is set.
I1224 22:38:49.659141   11911 instancegroups.go:280] Stopping instance "i-0b30ecce28e24fba9", in group "nodes.kubernetes.majumder-jyotisekhar.me" (this may take a while).
W1224 22:42:50.088648   11911 instancegroups.go:184] Not validating cluster as cloudonly flag is set.
I1224 22:42:50.088755   11911 rollingupdate.go:184] Rolling update completed for cluster "kubernetes.majumder-jyotisekhar.me"!

```

Now check from aws cli or from ec2 console, that all k8s cluster instances are gone. To start the cluster reverse the config for "MaxSize, MinSize as desired value and start the cluster.




