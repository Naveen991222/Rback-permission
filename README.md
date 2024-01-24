# Rback-permission
If we want master Access on Eks Cluster we need to check aws-auth file
path of aws-auth config namespace kube-system on config maps use below commands for view the aws-auth config file
kubectl get configmap aws-auth -n kube-system -o yaml 
kubectl edit configmap aws-auth -n kube-system   --> for edit purpose  
we need to edit the  bleow file


apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/worker-node-role
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/admin-user
      username: admin-user
      groups:
        - system:masters

Above file we need add the mapusers userarn in aws and username in aws account we need toa add enter control+s automatically its saved 
kubectl apply -f aws-auth.yaml

# Rback for user 
Step 1: Create a ClusterRole
Create a YAML file (e.g., cluster-role.yaml) with the necessary permissions. This is an example of a ClusterRole with broad permissions. Customize it based on your security requirements.

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: full-cluster-access
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]

kubectl apply -f cluster-role.yaml

Step 2: Bind the ClusterRole to the User

Create a ClusterRoleBinding YAML file (e.g., cluster-role-binding.yaml):
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: grant-full-cluster-access
subjects:
- kind: User
  name: "<employee-username>"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: full-cluster-access
  apiGroup: rbac.authorization.k8s.io
  
Replace <employee-username> with the actual username or identity of the new employee.

Apply the ClusterRoleBinding:
kubectl apply -f cluster-role-binding.yaml
now user




For AWS Side 

Creating IAM users and associating them with roles for an Amazon EKS cluster involves several steps. Here's a step-by-step process from scratch:

Step 1: Create IAM User
Open the AWS Management Console.

Navigate to the IAM service.

In the IAM dashboard, click on "Users" in the left navigation pane.

Click "Add user."

Enter a username and choose "Programmatic access."

Attach policies to the user based on the permissions required for your EKS cluster. At a minimum, the user needs the AmazonEKSClusterPolicy to interact with EKS.

Complete the user creation process, and note the Access Key ID and Secret Access Key. You'll need these later.

Step 2: Create IAM Role for EKS Worker Nodes
In the IAM dashboard, click on "Roles" in the left navigation pane.

Click "Create role."

Select "Amazon EKS" as the service that will use this role.

Choose "Amazon EKS - Worker Node" as the use case.

Attach policies based on the permissions required for your worker nodes. At a minimum, attach the AmazonEC2ContainerRegistryReadOnly and AmazonEKSWorkerNodePolicy policies.

Complete the role creation process, and note the Role ARN.

Step 3: Associate IAM User with IAM Role
Edit the IAM user you created earlier.

In the "Permissions" tab, click "Attach policies."

Attach the policies required for the IAM user.

In the "Add inline policy" section, create a policy to assume the EKS role. Example policy:

json
Copy code
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::123456789012:role/eks-worker-node-role"
    }
  ]
}
Replace the Resource ARN with the ARN of the EKS worker node role.

Step 4: Configure kubeconfig for EKS Cluster
Install AWS CLI and configure it with the IAM user's access key and secret.

bash
Copy code
aws configure
Install eksctl if not already installed:

bash
Copy code
brew install eksctl  # For macOS
Use eksctl to create a new EKS cluster:

bash
Copy code
eksctl create cluster --name my-cluster --region us-west-2 --node-type t2.small --nodes 2 --managed
Adjust parameters based on your preferences and AWS region.

Step 5: Update kubeconfig with User and Role
Fetch the cluster's kubeconfig:

bash
Copy code
aws eks --region us-west-2 update-kubeconfig --name my-cluster



