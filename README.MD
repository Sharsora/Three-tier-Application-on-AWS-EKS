


1. Source Code & Dockerization
Sabse pehle, Developer se humein Application code milta hai (React frontend, Node.js backend). Isme package.json (dependencies ke liye) aur index.js (entry point) jaisi files hoti hain.

Dockerfile Kyu? Iska use hum Easy Deployment ke liye karte hain. Docker application ko "Package" kar deta hai (Library, OS, Code sab ek sath). Isse "it works on my machine" wali problem khatam ho jati hai; aap ise kisi bhi environment (Dev, QA, Prod) mein bina kisi error ke chala sakte hain.

Process: Dockerfile se hum Image banate hain aur us image ko AWS ECR (Elastic Container Registry) mein push karte hain, jo hamara private storage cloud hai.

2. Kubernetes Configuration (YAML Files)
Jab image cloud (ECR) par chali jati hai, tab hum Kubernetes (K8s) ko batate hain ki ise kaise chalana hai.

deployment.yaml: Iska kaam hai Pods ko manage karna. Ye batata hai ki kitne Replicas (copies) chalani hain aur kaunsi image use karni hai.

service.yaml: Iska kaam hai Pods ko networking dena. Pods temporary hote hain (delete hote rehte hain), Service unhe ek Permanent Name/IP deti hai taaki internal communication ho sake.

Services ke Types:

ClusterIP (Default): Sirf cluster ke andar baat karne ke liye (Jaise Backend ko DB se baat karni ho).

NodePort: Har worker node par ek port open kar deta hai (Testing ke liye theek hai).

LoadBalancer: Seedha ek Cloud Load Balancer create kar deta hai (Mehenga hota hai agar har service ke liye alag banayein).

3. Access & Networking (Ingress & ALB)
Application chal to gayi, lekin user ise internet se kaise dekhega? Yahan Ingress ka concept aata hai.

Ingress Resource vs ALB:

Ingress Resource: Ye ek YAML file hai jisme aap "Rules" likhte hain (e.g., shiva.com/app par jaye to frontend dikhao). Ye sirf ek Document/Rulebook hai, ye khud traffic handle nahi karta.

ALB (Application Load Balancer): Ye actual AWS ka hardware/resource hai jo traffic ko receive karta hai.

AWS Load Balancer Controller ka role:
Ye sabse important part hai. Kubernetes aur AWS do alag duniya hain. Ingress Resource (K8s) ko dekh kar AWS par ALB banana, ye kaam AWS Load Balancer Controller karta hai.

Kyu chahiye? Agar controller nahi hoga, to aapko Manually AWS console par ja kar ALB banana padega, jo automation ke khilaf hai.

Difference: Controller ek "Manager" hai jo Ingress file ko padhta hai aur automatically AWS par ALB create/update kar deta hai.

Summary Flow 

Code liya -> Docker Image banayi -> ECR mein push kiya.

Deployment file se Pods chalaye.

Service (ClusterIP) se internal connection kiya.

Ingress Resource file likhi (Rules define kiye).

ALB Controller ne wo file dekhi aur AWS par ALB create kar diya.

Ab User ALB ke URL se aapki application access kar sakta hai.

Setup EKS Cluster

```
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```
Install AWS Load Balancer
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2
```

Deploy AWS Load Balancer Controller
```
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f full_stack_lb.yaml
```
