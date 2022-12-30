## IAM Empty role config rule
AWS Config rule to detect IAM roles with no attached policies (both inline and managed policies) created using rule [AWS Config Rules Development Kit](https://github.com/awslabs/aws-config-rdk).

## Deployment
1. Make sure you have [rdk](https://rdk.readthedocs.io/en/latest/getting_started.html#installation) installed
2. Clone this repository
3. Navigate to cloned repository directory
2. run `rdk deploy IAM_EMPTY_ROLE`

This will deploy your rule to AWS Config, and you can track deployment progress in your terminal. For more information, view Deploy in RDK Command Reference documentation.

## Rule properties
### Resources in scope
AWS IAM Role
### Rule triggers
* Configuration item change: runs every time an in-scope resource is changed. For example, when a role is created or modified. Ignores deleted resources. When the rule is triggered by configuration item change, it only assesses the changed resource except for the first run when the rule assesses all the configuration items for resources in scope, stored in AWS config.
* Scheduled notification: every 24 hours, it checks all IAM roles for compliance.

### Parameters
This rule does not take any parameters and will raise an error if you specify a parameter.
### Logic
For configuration item change assessments, configuration item is used to assess compliance. For scheduled assessments, API calls are made to gather infromation and make assessments.

#### Configuration item change assessments:
1. [`evaluate_changetrigger_compliance`](IAM_EMPTY_ROLE/IAM_EMPTY_ROLE.py#L57) function receives the configuration item
2. It checks `['configuration']['attachedManagedPolicies']` (list of all managed policies attached to a role) and `['configuration']['rolePolicyList']` (list of all inline policies attached to a role) lists, and if both are empty if marks the resource non-compliant. 

Here is the [configuration item schema](https://github.com/awslabs/aws-config-resource-schema/blob/master/config/properties/resource-types/AWS::IAM::Role.properties.json) for AWS::IAM::Role resource type.

#### Scheduled assessments:
1. [`evaluate_scheduled_compliance`](IAM_EMPTY_ROLE/IAM_EMPTY_ROLE.py#L30) function gets a list of all rules using boto3 iam client [list_roles](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_roles) method. 
2. For each role in roles list, it lists the names of the inline policies that are embedded in the specified IAM role using iam [list_role_policies](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_role_policies) method
3. It lists all managed policies that are attached to the specified IAM role using iam [list_attached_role_policies](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_attached_role_policies) method. 
4. If both lists are empty it marks the resource non-compliant and moves to the next resource on the list.


