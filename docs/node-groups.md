# Node Groups

The framework uses dedicated sub modules for creating [AWS Managed Node Groups](modules/aws-eks-managed-node-groups), [Self-managed Node groups](modules/aws-eks-self-managed-node-groups) and [Fargate profiles](modules/aws-eks-fargate-profiles). These modules provide flexibility to add or remove managed/self-managed node groups/fargate profiles by simply adding/removing map of values to input config. See [example](deploy/eks-cluster-with-new-vpc/main.tf).

The `aws-auth` ConfigMap handled by this module allow your nodes to join your cluster, and you also use this ConfigMap to add RBAC access to IAM users and roles.
Each Node Group can have dedicated IAM role, Launch template and Security Group to improve the security.

Please refer this [full example](deploy/eks-cluster-with-new-vpc/main.tf)

## Managed Node Groups

The below example demonstrates the minimum configuration required to deploy a managed node group.

```hcl
    # EKS MANAGED NODE GROUPS
    managed_node_groups = {
        mg_4 = {
            node_group_name = "managed-ondemand"
            instance_types  = ["m4.large"]
            subnet_ids      = [] # Mandatory Public or Private Subnet IDs
        }
    }
```

The below example demonstrates advanced configuration options for a manged node group.

```hcl
    managed_node_groups = {
      mg_m4 = {
        # 1> Node Group configuration
        node_group_name             = "managed-ondemand"
        create_launch_template      = true              # false will use the default launch template
        launch_template_os          = "amazonlinux2eks" # amazonlinux2eks or windows or bottlerocket
        public_ip                   = false             # Use this to enable public IP for EC2 instances; only for public subnets used in launch templates ;
        pre_userdata           = <<-EOT
                yum install -y amazon-ssm-agent
                systemctl enable amazon-ssm-agent && systemctl start amazon-ssm-agent"
            EOT
        # 2> Node Group scaling configuration
        desired_size    = 3
        max_size        = 3
        min_size        = 3
        max_unavailable = 1 # or percentage = 20

        # 3> Node Group compute configuration
        ami_type       = "AL2_x86_64"             # AL2_x86_64, AL2_x86_64_GPU, AL2_ARM_64, CUSTOM
        capacity_type  = "ON_DEMAND"              # ON_DEMAND or SPOT
        instance_types = ["m4.large"]             # List of instances used only for SPOT type
        disk_size      = 50

        # 4> Node Group network configuration
        subnet_ids  = []                          # Mandatory - # Define private/public subnets list with comma separated ["subnet1","subnet2","subnet3"]
        k8s_taints = []
        k8s_labels = {
          Environment = "preprod"
          Zone        = "dev"
          WorkerType  = "ON_DEMAND"
        }
        additional_tags = {
          ExtraTag    = "m4-on-demand"
          Name        = "m4-on-demand"
          subnet_type = "private"
        }
        create_worker_security_group = false
      }
    }
```

## Self-managed Node Groups

The example below demonstrates how you can customize and deploy a self-managed node group for your cluster.

```hcl
  self_managed_node_groups = {
    self_mg_4 = {
      node_group_name    = "self-managed-ondemand"
      create_launch_template = true
      launch_template_os = "amazonlinux2eks"       # amazonlinux2eks  or bottlerocket or windows
      custom_ami_id      = "ami-0dfaa019a300f219c" # Bring your own custom AMI generated by Packer/ImageBuilder/Puppet etc.
      public_ip          = false                   # Enable only for public subnets
      pre_userdata       = <<-EOT
            yum install -y amazon-ssm-agent \
            systemctl enable amazon-ssm-agent && systemctl start amazon-ssm-agent \
        EOT

      disk_size     = 20
      instance_type = "m5.large"
      desired_size = 2
      max_size     = 10
      min_size     = 2
      capacity_type = "" # Optional Use this only for SPOT capacity as  capacity_type = "spot"

      k8s_labels = {
        Environment = "preprod"
        Zone        = "test"
        WorkerType  = "SELF_MANAGED_ON_DEMAND"
      }

      additional_tags = {
        ExtraTag    = "m5x-on-demand"
        Name        = "m5x-on-demand"
        subnet_type = "private"
      }
      subnet_ids  = [] # Mandatory Public or Private Subnet IDs
      create_worker_security_group = false
    },

  }
```

### Fargate Profile

The example below demonstrates how you can customize a Fargate profile for your cluster.

```hcl
    fargate_profiles = {
      default = {
        fargate_profile_name = "default"
        fargate_profile_namespaces = [{
          namespace = "default"
          k8s_labels = {
            Environment = "preprod"
            Zone        = "dev"
            env         = "fargate"
          }
        }]

        subnet_ids = [] # Mandatory - # Define private subnets list with comma separated ["subnet1","subnet2","subnet3"]

        additional_tags = {
          ExtraTag = "Fargate"
        }
      }
    }
```
