### The purpose of this repo is to provide a POC for updating child policies who have "declined" inheritence for HTTP compliance

_This was tested on BIG-IQ version 7.1.0.2 with (1) Parent Policy and (7) child policies; of the 7 child policies 4 have declined the "HTTP Protocol Compliance" property_


The logic and work flow for the ansible code is as follows:
* Identifying the parent policy
    * GET request to /mgmt/cm/asm/working-config/policies/?$filter=type%20eq%20%27parent%27&$select=id,name
        * Filter response to only include policies that are listed as a parent
        * Only provide the name and id of the parent policy in response
    * Using above information create a variable with id of parent policy
* Identify all child policies associated with parent above
    * GET request to /mgmt/cm/asm/working-config/policies?$filter=parentPolicyReference/id eq '02f94b85-515b-3c73-a8d8-2cde271b10be'&$expand=sectionReference
        * Filter response to include only child policies that are associated with previously discovered parent
        * Expand the 'sectionReference' json blob to be able to determine if the child policy has declined the "HTTP Protocol Compliance" property
    * Using above information create a dictionary that includes Child Policy name, policy ID, and compliance declined status
* Identify the id of the "Header name with no header value"
    * GET request to /mgmt/cm/asm/working-config/policies/<child_policy_id>/http-protocols
    * Using the response and a jsonFilter, find the json blob associated with "Header name with no header value", and get the associated id
    * Using above information add the new id to the existing Child Policy dictionary which now includes: name, policy ID, compliance declined status, and the id for "Header name with no header value"
* Submit a PATCH to turn-off checking for "Header name with no header value"
    * PATCH request to /mgmt/cm/asm/working-config/policies/<child_policy_id>/http-protocols/<Header_name_with_no_header_value_id>
    * patch body = {"enabled": false} 

