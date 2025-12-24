1.INSTALL OIDC provider which helps k8 to connect to other sefvies in AWS
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster <your-cluster-name> \
    --approve

2. Download an IAM policy for the LBC ---> using AWS cli we will create one policy by giving it one name:

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.17.0/docs/install/iam_policy.json


3. Create an IAM policy: ---> using AWS cli we will create one policy by giving it one name:

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

When you run this command, AWS will:
Read the JSON file containing permission definitions
Create a new managed policy in your AWS account
Make it available to attach to IAM roles or users
Return the policy's ARN (Amazon Resource Name) that you can reference later

4. Create Service account and IAM role. map the above policy
# this is our service accont name : aws-load-balancer-controller 
# this our policy name: AWSLoadBalancerControllerIAMPolicy

eksctl create iamserviceaccount \
--cluster=roboshop-dev \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::319255955334:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve

Breaking down the command
eksctl create iamserviceaccount - Creates both a Kubernetes service account AND an IAM role, linking them together using OIDC
--cluster=roboshop-dev - The name of your EKS cluster where this will be created
--namespace=kube-system - The Kubernetes namespace where the service account will be created (kube-system is where system components run)
--name=aws-load-balancer-controller - The name of the Kubernetes service account being created
--attach-policy-arn=arn:aws:iam::319255955334:policy/AWSLoadBalancerControllerIAMPolicy - Attaches the IAM policy you created earlier to the IAM role, granting those permissions
--override-existing-serviceaccounts - If a service account with this name already exists, it will be updated/replaced
--region us-east-1 - The AWS region where your EKS cluster is located
--approve - Automatically approves and executes the changes without prompting for confirmation
#POLICY ARN: arn:aws:iam::319255955334:policy/AWSLoadBalancerControllerIAMPolicy

5. Install drivers

Install helm

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

6. Add EKS chart repo

helm repo add eks https://aws.github.io/eks-charts

7. Install Load balancer controller ---> using helm we got 2 pods  AWS LB CONTROLLER for our cluster.

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=roboshop-dev --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller



8.kubectl get pods -n kube-system

These are two pods running the AWS Load Balancer Controller in your EKS cluster.
check drivers are running

NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
kube-system   aws-load-balancer-controller-84fb88bb55-mwnlc   1/1     Running   0          20s
kube-system   aws-load-balancer-controller-84fb88bb55-rqsjg   1/1     Running   0          20s 

What These Pods Do
The AWS Load Balancer Controller watches your Kubernetes cluster for:
Ingress resources - When you create an Ingress, it automatically provisions an AWS Application Load Balancer (ALB)
Service resources with type LoadBalancer - When you create a LoadBalancer service with proper annotations, it can provision either an ALB or Network Load Balancer (NLB)
Target group binding - It manages the connection between your Kubernetes pods and AWS load balancer target groups