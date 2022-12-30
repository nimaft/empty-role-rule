# IAM Empty role config rule
AWS Config rule to detect IAM roles with no attached policies (both inline and managed policies) created using rule [AWS Config Rules Development Kit](https://github.com/awslabs/aws-config-rdk).

## Resources in scope
AWS IAM Role
## Rule triggers
* Configuration item change: runs every time an in-scope resource is changed. For example, when a role is created or modified. Ignores deleted resources. When the rule is triggered by configuration item change, it only assesses the changed resource except for the first run when the rule assesses all the configuration items for resources in scope, stored in AWS config.
* Scheduled notification: every 24 hours, it checks all IAM roles for compliance.

# Parameters
This rule does not take any parameters and will raise an error if you specify a parameter.
## Logic
For configuration item change assessments, configuration item is used to assess compliance. For scheduled assessments, API calls are made to gather infromation and make assessments.

### Configuration item change assessments:
[`evaluate_changetrigger_compliance`](IAM_EMPTY_ROLE/IAM_EMPTY_ROLE.py#L57) function receives the configuration item, it checks `['configuration']['attachedManagedPolicies']` (listing all managed policies attached to a role) and `['configuration']['rolePolicyList']` (listing all inline policies attached to a role) lists, and if both are empty if marks the resource non-compliant. Here is the [configuration item schema](https://github.com/awslabs/aws-config-resource-schema/blob/master/config/properties/resource-types/AWS::IAM::Role.properties.json) for AWS::IAM::Role resource type.

### Scheduled assessments
[`evaluate_scheduled_compliance`](IAM_EMPTY_ROLE/IAM_EMPTY_ROLE.py#L30) function gets a list of all rules using boto3 iam client [list_roles](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_roles) method. For each role in roles list, it lists the names of the inline policies that are embedded in the specified IAM role using iam [list_role_policies](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_role_policies) method, and lists all managed policies that are attached to the specified IAM role using iam [list_attached_role_policies](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_attached_role_policies) method. If both lists are empty it marks the resource non-compliant and moves to the next resource on the list.


