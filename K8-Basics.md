##### Replication Controller

```
kubectl create -f data/replication-controller/helloworld-repl-crtl.yaml
kubectl describe ReplicationController/helloworld-controller
kubctl get pods
```
If there are mutiple replicas of same pod and then one pod is deleted [kuctl delete pod <name>] then repliocation controller automatically deploys another pod which delivers the HA feature. 

Pods can be horizontally scale (up/down) using replicas like below
```
kubectl scale --replicas=4 -f data/replication-controller/helloworld-repl-crtl.yaml
kubectl get rc
kubectl scale --replicas=1 rc/helloworld-controller
```
Using replication controller pods can be scaled horizontically _ONLY_ if the pods are _STATELESS_, in case of _STATEFUL_ pods we cannot scale the pods like this. 


##### Relication Sets
Replication Ser is the next generation Replication Controller.It's supports a new selector that can do selection based on filtering according to a set of values. For instance, the selection could be an environment that is either going to be "dev" or "qa",

not only based on equality like the Replication Controller in the Replication Controller, you can only say an

"environment == dev", you couldn't do anything more complex. With the replication set you can do more complex matching. This Replica Set rather than Replication Controller is used 
by the deployment object.

##### Deployment Object
A deployment is a declaration in Kubernetes that allows you to do app deployments and updates. 
- When using the deployment object, you define the state of your application. Kubernetes will then make sure the cluster matches your desired state.A state can just be running this application and make sure it is replicated five times. 
- Just the way to describe this to Kubernetes is most easily done now with the Deployments.

It would just be too much manual work to then do updates and rollbacks and so on. The deployment object is much easier to use and gives you more possibilities. So, what are those possibilities with a deployment object.

- You can create a deployment, for instance, just deploying an app, update a deployment, deploy a new version of your app.
- You can also do rolling updates, which means zero downtime deployments. You're not immediately going to update your whole app, but you are going to do it in steps, so that you will have zero downtime during this deployment.
- You can also easily rollback to a previous version of your app, you can pause and resume a deployment as well. For instance if you want to roll out to only a certain percentage of your running pods.

> kubectl get deployments.

> kubectl get rs - get info about replica set.

> kubectl rollout status deployment/helloworld-deployment.

> kbectl set image deplpyment/helloworld-deplpyment k8s-demo=k8s-demo:2

> kubectl rollout history deployment/helloworld-deployment

If you release a new version of your app that's a version 3 and you want to go back to version 2 then you can do a "Kubectl rollout undo" and this rolls back to the previous version. If you want to rollback to another version, you can see all the versions using rollout history if you want them rollback to version 1, for instance, you can just specify doing "Kubectl rollout undo --to revision=n" and the revision number.

> kubectl rollout undo deployment/helloworld-deployment



