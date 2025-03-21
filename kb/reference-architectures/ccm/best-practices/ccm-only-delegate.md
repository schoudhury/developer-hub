---
title: IaC for a CCM Only Delegate
description: IaC code samples for a Harness delegate that is tasked only for CCM related activities.
---

# Provision a CCM Only Delegate via a Helm chart

## Overview
Because Delegates are a Harness platform offering, they can potentially be used for other modules other than CCM. In this example, we are setting specific parameters to ensure that the service account of the Delegate is read only and preventing the running of scripts:

```
--set k8sPermissionsType=CLUSTER_VIEWER \
--set-json custom_envs='[{"name":"BLOCK_SHELL_TASK","value":"true"}]'
```

## Resource Requirements
Gathering fine-grain metrics in the cluster is memory intensive.  In an effort to ensure we don't run out of memory and terminate the pod, the following sizing guidelines are recommended:

| # Nodes in the Cluster | CPU (Cores) | MEM (Mi)  |
| -----------------------| ----------- | --------- |
|        `<= 100`        |      1      |    3814   |
|       `101 - 200`      |      2      |    7629   |
|       `201 - 300`      |      3      |   11444   |
|       `301 - 400`      |      4      |   15258   |
|       `401 - 500`      |      5      |   19073   |

## Helm Chart
```
helm upgrade -i helm-delegate --namespace harness-delegate-ng --create-namespace \
  harness-delegate/harness-delegate-ng \
  --set delegateName=helm-delegate \

  # Can be found in the Account Settings page.
  --set accountId=< Your Harness Account ID > \

  # can be found by going to Account Settings > Account Resources > Delegates > Tokens and either copy the 
  # default_token or by creating a new specific token for this install 
  #(https://developer.harness.io/docs/platform/delegates/install-delegates/overview/#create-a-new-delegate-token)
  --set delegateToken=< Your Harness Delegate Token > \

  # Prod1 = https://app.harness.io/, Prod2 = https://app.harness.io/gratis, Prod3 = https://app3.harness.io
  --set managerEndpoint=https://app3.harness.io \

  # Get latest version inside of Harness UI.  Account Settings -> Delegates
  --set delegateDockerImage=harness/delegate:24.04.82901 \
  
  --set replicas=1 \
  --set upgrader.enabled=true \

  # Sets the service account of the delegate to read only
  --set k8sPermissionsType=CLUSTER_VIEWER \

  # Provisions a cluster role with these permissions https://github.com/harness/delegate-helm-chart/blob/main/harness-delegate-ng/templates/ccm/cost-access.yaml
  --set ccm.visibility=true \
  
  # Prevent the delegate from being used for running scripts
  --set-json custom_envs='[{"name":"BLOCK_SHELL_TASK","value":"true"}]' \

  # Set CPU and MEM resource limits
  --set cpu=1 \
  --set memory=3814
  ```


# Use Terraform to provision Delegate tokens

## Overview
Managing delegate tokens at scale isn't ideal inside the Harness UI.  You can instead manage these with Terraform using the [Harness provider](https://registry.terraform.io/providers/harness/harness/latest/docs/resources/platform_delegate_token).  All CCM functionality is at the account level so ensure you only set the `account_id` parameter and do not set an organization or project.

You can then leverage the token created to provision your delegate with Terraform: `harness_platform_delegatetoken.this.value`.  You can use the [Helm provider](https://registry.terraform.io/providers/hashicorp/helm/latest/docs) to directly reference the delegate token within your [delegate deployment values](https://registry.terraform.io/modules/harness/kubernetes-delegate/harness/latest).

## Terraform Example
```
# Create delegate token for account level 
resource "harness_platform_delegatetoken" "test" {  
  name        = "test token"
  account_id  = "account_id"
}
```