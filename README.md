# Blackbird Cloud Kubernetes Application Bootstrap
[![blackbird-logo](https://raw.githubusercontent.com/blackbird-cloud/terraform-module-template/main/.config/logo_simple.png)](https://www.blackbird.cloud)

A handy helm template for bootstraping k8s applications. This template creates a space for kubernetes applications and some small bootstrap components such as secrets, namespace, and ArgoCD application.

This repo can be combined with our terraform helm deploy modules for all major cloud provider, [AWS EKS](https://github.com/blackbird-cloud/terraform-aws-eks-helm-release), [Google Cloud GKE](https://github.com/blackbird-cloud/terraform-google-gke-helm-deployment), and [Azure AKS](https://github.com/blackbird-cloud/terraform-azurerm-aks-helm-release). Or maybe BYOP (Bring Your Own Provider) with [terraform helm deploy module](https://github.com/blackbird-cloud/terraform-helm-deployment)

## Terraform Example
```hcl
module "my-application-bootstrap" {
  source  = "blackbird-cloud/deployment/helm"
  version = ">= 1.1.2"

  name        = "my-application"
  description = "my-application"
  namespace   = "argo-cd"

  cleanup_on_fail = true
  force_update    = true
  wait            = true
  wait_for_jobs   = true

  values = [
    yamlencode({
      namespace: {
        name: "my-application"
      },
      role: {
        create: true
      },
      secretproviderclass: {
        create: true,
        spec: {
            provider: "aws",
            secretsObjects: [
                {
                    data: [
                        {
                            key: "DATABASE_PASSWORD",
                            objectName: "DATABASE_PASSWORD"
                        }
                    ],
                    secretName: var.database_secret_name,
                    type: "Opaque"
                }
            ]
        }
      },
      argoapplication: {
        create: true,
        spec: {
            project: default,
            source: {
                repoURL: "https://github.com/my-organization/my-repository"
                targetRevision: "main"
                path: "./deployment/production"
            },
            destination: {
                server: https://kubernetes.default.svc
                namespace: "my-application"
            }
        }
      }
    })
  ]
}
```


## About

We are [Blackbird Cloud](https://blackbird.cloud), Amsterdam based cloud consultancy, and cloud management service provider. We help companies build secure, cost efficient, and scale-able solutions.

Checkout our other :point\_right: [terraform modules](https://registry.terraform.io/namespaces/blackbird-cloud)

## Copyright

Copyright © 2017-2023 [Blackbird Cloud](https://www.blackbird.cloud)
