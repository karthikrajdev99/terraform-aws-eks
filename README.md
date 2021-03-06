### Create Kubernetes cluster in AWS.
This module will create an EKS cluster in the AWS account managed by the current credential.

Features:
* Creates required VPC, Subnets, and Security Group
* Configures the EKS cluster to use it for dynamic worker nodes.  

* Installs Helm Tiller
* Installs ALB Ingress controller
* Configures AWS IAM Role mapping

***
## Inputs:
| Variable | Type | Description | Default |
|--------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| name | string | Name of the Kubernetes cluster. |  |
| kubernetes_version | string | Version of Kubernetes control plane to deploy. |  |
| vpc_cidr | string | CIDR for vpc |  |
| private_subnets | list | Private subnet CIDRs |  |
| public_subnets | list | Public subnet CIDRs |  |
| min_workers | number | The minimum number of instances to maintain in the worker pool | 1 |
| max_workers | number | The maximum number of instances to maintain in the worker pool | 10 |
| desired_capacity | number | The number of instances to launch and maintain in the cluster | 1 |
| map_roles | list | List of role mapping data structures to configure the AWS IAM Authenticator.,See: https://github.com/kubernetes-sigs/aws-iam-authenticator#full-configuration-format,[{,rolearn = "arn:aws:iam::000000000000:role/KubernetesAdmin",username = "kubernetes-admin",groups = ["system:masters"],}],EOT | [] |

***
## Outputs:
| Output | Description |  |  |
|---------------------|----------------------------------------------------------------------------|---|---|
| kubeconfig | Content of a `kubeconfig` file that can be used to connect to this cluster |  |  |
| kubeconfig_filename | The filename of the generated kubectl config. |  |  |
| ssh_private_key | Private key for SSH access to worker nodes. |  |  |

***
## Example:
```terraform

provider "aws" {
  version = "~> 2.33"
}

provider "kubernetes" {
  version     = "~> 1.9"
  config_path = module.k8s_cluster.kubeconfig_filename
}

provider "helm" {
  version         = "~> 0.10"
  service_account = "tiller"
  kubernetes {
    config_path = module.k8s_cluster.kubeconfig_filename
  }
}

provider "tls" {
  version = "~> 2.1"
}

module "k8s_cluster" {
  source = "../../"

  name               = "test-cluster"
  kubernetes_version = "1.14"
  vpc_cidr           = "10.0.0.0/16"
  private_subnets    = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets     = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
}

output "kubeconfig" {
  value = module.k8s_cluster.kubeconfig
}

output "ssh_private_key" {
  value     = module.k8s_cluster.ssh_private_key
  sensitive = true
}
```

## Notes:

Fetch kubeconfig from state:
```
terraform output kubeconfig > kubeconfig_<cluster-name>
```

Connect to cluster:
```
kubectl --kubeconfig kubeconfig_<cluster-name> ...
```

