### The purpose of this repo is to provide a POC for updating child policies who have "declined" inheritence for HTTP compliance

_This was tested on BIG-IQ version 7.1.0.2 with (1) Parent Policy and (7) child policies; of the 7 child policies 4 have declined the "HTTP Protocol Compliance" property_


The logic and work flow for the ansible code is as follows:

* Identifying the parent policy
    * GET request to /mgmt/cm/asm/working-config/policies/?$filter=type%20eq%20%27parent%27&$select=id,name
        * Filter response to only include policies that are listed as a parent
        * Only provide the name and id of the parent policy in response
    * Using above information create a variable with id of parent policy

* Identifying the setting for "Header name with no header value" in the parent policy, this sets the value to apply for the children who have declined inheritance for "HTTP Protocol Compliance"
    * GET request to /mgmt/cm/asm/working-config/policies/{{parentPolicyID}}/http-protocols/4dafc3c8-1e91-39ba-a20d-46c47cffa34d?$select=enabled
        * Filter response to return only the value of "enabled" for the "Header name with no header value" in the parent policy
    * Using above information create a variable with the "enabled" value

* Identify all child policies associated with parent above
    * GET request to /mgmt/cm/asm/working-config/policies?$filter=parentPolicyReference/id eq '{{parentPolicyID}}'&$expand=sectionReference
        * Filter response to include only child policies that are associated with previously discovered parent
        * Expand the 'sectionReference' json blob to be able to determine if the child policy has declined the "HTTP Protocol Compliance" property
    * Using above information create a dictionary that includes Child Policy name, policy ID, and compliance declined status

* Submit a PATCH to turn-off checking for "Header name with no header value"
    * PATCH request to /mgmt/cm/asm/working-config/policies/{{item.value.policyID}}/http-protocols/4dafc3c8-1e91-39ba-a20d-46c47cffa34d
    * patch body = {"enabled": {{ stateOfHeadNoValue }} } 

