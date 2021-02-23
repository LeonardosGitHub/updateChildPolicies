### The purpose of this repo is to provide a POC for updating child policies who have "declined" inheritence for HTTP compliance

_This was tested on BIG-IQ version 7.1.0.2_


The logic and work flow for the ansible code is as follows:

* Provide name of parent policy as input to the playbook
* Identifying the parent policy id and name
    * GET request to /mgmt/cm/asm/working-config/policies/?$filter=name%20eq%20%27{{parentPolicy}}%27&$select=id,name
        * Filter response to only include policies that match name provided
        * Only provide the name and id of the parent policy in response
    * Using above information create a variable with id of parent policy

* Identifying the setting for "Header name with no header value" in the parent policy, this sets the value to apply for the children who have declined inheritance for "HTTP Protocol Compliance"
    * GET request to /mgmt/cm/asm/working-config/policies/{{parentPolicyID}}/http-protocols/4dafc3c8-1e91-39ba-a20d-46c47cffa34d?$select=enabled
        * Filter response to return only the value of "enabled" for the "Header name with no header value" in the parent policy
    * Using above information create a variable with the "enabled" value

* Identify all child policies associated with parent above
    * GET request to /mgmt/cm/asm/working-config/policies?$filter=parentPolicyReference/id eq '{{parentPolicyID}}'&$expand=sectionReference,httpProtocolsReference
        * Filter response to include only child policies that are associated with previously discovered parent
        * Expand the 'sectionReference' json blob to be able to determine if the child policy has declined the "HTTP Protocol Compliance" property
    * Using above information create a dictionary that includes Child Policy name, policy ID, enabled value of "Header name with no header value" and compliance declined status

* Submit a PATCH to update enabled value for "Header name with no header value" to match parent policy setting
    * PATCH request to /mgmt/cm/asm/working-config/policies/{{item.value.policyID}}/http-protocols/4dafc3c8-1e91-39ba-a20d-46c47cffa34d
    * patch body = {"enabled": {{ stateOfHeadNoValue }} } 

