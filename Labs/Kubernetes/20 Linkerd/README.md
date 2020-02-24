<img src="../img/logo.png" alt="Chmurowisko logo" width="200" align="right">
<br><br>
<br><br>
<br><br>

# Lab title

## LAB Overview

#### Lab description.....

## Task 1: .....
Short description

1. 
```
linkerd check --pre
```
2. 
```
linkerd install | kubectl apply -f -
```
3. 
```
linkerd check
```
4. 
```
kubectl -n linkerd get deploy
```
1. 
```
kubectl create ns simple-service
```
## Task 2:
2. 
```
kubectl apply -f 1_deployment.yaml -n simple-service
```
3. 
```
linkerd dashboard
```
4. 
```
kubectl apply -f 2_debug.yaml
```
for i in {1..10};do curl versions-app1.simple-service.svc.cluster.local:5000; echo; done
for i in {1..10};do curl versions-app2.simple-service.svc.cluster.local:5000; echo; done
5. 
kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -
6. 
```
kubectl apply -f 4_deployment.yaml -n simple-service
```
for i in {1..10};do curl versions-app-error.simple-service.svc.cluster.local:5000; echo; done
7. 
```
kubectl apply -f error-split.yaml -n simple-service
```
8. 
```
kubectl apply -f 6_profile.yaml
```
## END LAB

<br><br>

<center><p>&copy; 2019 Chmurowisko Sp. z o.o.<p></center>
