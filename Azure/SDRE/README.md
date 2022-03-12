# Secure Digital Research Environments (SDRE) on Azure


This is work towards HIPAA compliance on the Azure public cloud. Azure has some incredibly useful machinery 
built in towards this end that can save a research team, together with their IT support staff, a ton of 
time and let them get to the point quicker, which is health research based on clinical data.


Goa: Build a synthetic HIPAA-compliant secure computing environment on Azure. 


Using Azure Blueprint associated with NIST 800-53, *not* 800-66. Clarification on these two NIST documents.


NIST 800-53 is a publication that recommends security controls for federal information systems and organizations and documents 
security controls for all federal information systems, except those designed for national security. The association seems
to be with [FISMA](https://en.wikipedia.org/wiki/Federal_Information_Security_Management_Act_of_2002). 


Moving to HIPAA: 


NIST published "An Introductory Resource Guide for Implementing the Health Insurance Portability and Accountability Act (HIPAA) 
Security Rule (SP 800-66 Revision 1)" in October 2008 to assist covered entities in understanding and properly using the set of
federal information security requirements adopted by the Secretary of Health and Human Services (HHS) under the 
Health Insurance Portability and Accountability Act of 1996 (HIPAA, Public Law 104-191). HIPAA required the Secretary to adopt, 
among other standards, security standards for certain health information. These standards, known as the HIPAA Security Rule, 
were published on February 20, 2003. In the preamble to the Security Rule, 
several NIST publications were cited as potentially valuable resources for readers with specific questions and concerns 
about IT security.


[Here is the link to NIST 800-66.](http://csrc.nist.gov/publications/nistpubs/800-66-Rev1/SP-800-66-Revision1.pdf)


[HIPAA Act](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)


closely related: HITECH act


[HITECH Act](https://en.wikipedia.org/wiki/Health_Information_Technology_for_Economic_and_Clinical_Health_Act)


[Procedural start](https://docs.microsoft.com/en-us/azure/governance/blueprints/samples/nist-sp-800-53-rev4/deploy)


[Here is an overview of this blueprint.](https://docs.microsoft.com/en-us/azure/governance/blueprints/samples/nist-sp-800-53-rev4/index)


[Control mapping (parameters) doc](https://docs.microsoft.com/en-us/azure/governance/blueprints/samples/nist-sp-800-53-rev4/control-mapping)



### Helpful remarks from Clayton Barlow


Links for HIPAA blueprint and related follow. 
 

[HIPAA Blueprint at Azure](https://servicetrust.microsoft.com/ViewPage/HIPAABlueprint)

 
[Azure health blueprint](https://docs.microsoft.com/en-us/azure/security/blueprints/azure-health)


[GitHub on same](https://github.com/Azure/Health-Data-and-AI-Blueprint)



## Notes on Azure SDRE implementation


How long to build? 2.5 days


Automated via ARM (Azure Resource Management) template


- Virtual Network (VPC analog) (VNet)
- Subnets: mainsubnet (private) and bastion (edge server)
    - This mainsubnet is the path to the object storage
- Peering with a separate VNet with access to Express Route
- Network security group (NSG)
- Bastion service


The ARM template is JSON, developed in Visual Studio code using the ARM extension "ARM Tools".
The template file contains IP addresses. 
Azure DevOps repo designated for this purpose. 
Azure organization is XXXXXXXX.


Automated:
- Creation of the VMs 
    - Future: To be improved; for example any secrets / passwords > KeyVault
    - Specifically DSVMs
  
Manual: 
- Creating bock storage as a data disk "managed disk"
- Create a Key Vault 
    - Add DSVM identities here so that they can actually access the data via encryption
- Drive encryption keys
- Powershell: Go encrypt the drives (root and data)
- Add NetID Group containing the external org users. User ~ Group ~ VM: Group added to Admin's group.
- Install DUO software for 2FA
- Each DSVM ip address is static
- Schedule: Shutdown at 9 pm UTC

Created a Custom Role (script) called vmoperator to give them the ability to shutdown / start VMs
(this script is public) Machines are started on demand by researcher 

Automated
- Creation of the AML with dependencies from a separate template
    - "Application Insights"
    - AML Workspace
    - Key Vault (not used)
    - Storage Account (prereq, not used)

Operation: 
- Need quota big enough
- Request compute "NC12S" where 12 is cores; so 12 x 20 nodes = 240; big quota
- Combination of cores per node and number of nodes

Storage: 
- Create two storage accounts 'static' and 'working'
    - As storage account V2: Added features 
    - 'blob only' storage; this will probably change
    - private endpoint, NIC = Network Interface Card
    - for each storage account
        - connection string based on an SAS token
        - connection string is stored in Key Vault
    
Logging mechanism: 
- Storage accounts store data internally: Extract and inject into Log Analytics Workspace
- Access to VMs: Stored in a Log A W in a different table
- Join and create a story!

HIPAA blueprint: Policy, compliance, practices. 
- Enables Security Center to identify whether we are in compliance
- Enrolling the researcher (actual access, training on practices)
  
Also: Storage Recovery Services Vault: To back up the DSVMs 
