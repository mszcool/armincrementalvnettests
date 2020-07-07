# Tests for idempotent ARM-based deployments of Networking Resources with DINE Policies and Blueprints

This repository contains a few templates which we used for testing the effect of re-deploying templates on existing infrastructure related to networking. Unfortunately, per July 2020, the behavior of re-deploying a lot of networking-related resources, especially VNETs, leads to a re-creation (attempt) of those resources. This repository contains a few workarounds to avoid these re-creation (attempts) to allow implementing i.e. upgrades of infrastructure without intrusive/breaking existing deployments.

**--- Work in Progres ---**

## A few deployment hints

Deploying Azure Policy via ARM templates
* Make sure to escale template-functions that should be evaluated during policy execution and NOT during ARM template execution:
   ```json
   "existenceCondition": {
    "allOf": [
        {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        {
            "field": "name",
            "equals": "[[parameters('networkRgName')]"
        },
        {
            "field": "type",
            "equals": "Microsoft.Network/virtualNetworks"
        },
        {
            "field": "name",
            "equals": "[[parameters('networkName')]"
        }
    ]
   }
   ```

* Deploying an Azure Policy as ARM template using the new `az deployment sub create` capability:
   ```bash
    az deployment sub create --location westeurope --template-file ./arm.policy.vnet.json

    # Note: if you want to deploy the policy to a Management Group rather than subscription, you can use the command below. But, the subscription()-template function will not work in that case and you need to reference roles which are available at the management group level.
    az deployment mg create --management-group-id yourtestmg --name deploymentname --location westeurope --template-file ./arm.policy.vnet.json
   ```

* Test deploying the VNET templates and the impact goes as follows:
  * Deploy the ARM template for the VNET
  * Make some changes after deployment through i.e. the portal
  * Re-deploy the template, again, and observe impact by trying out both, `true` and `false` for the parameter `isNewDeployment` when using the flavor `arm.vnet.deploy.1.json`in your tests.