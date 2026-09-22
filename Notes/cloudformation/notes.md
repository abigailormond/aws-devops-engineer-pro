CloudFormation Physical & Logical Resources
- CloudFormation Template
    - JSON or YAML
    - contains logical resources
    - Templates used to create stacks (any #)
    - Reusable across regions and accounts
- Stacks
    - create physical resources from the logical
    - If a stacks template changes -> physical resources are changed
    - if a stack is deleted -> physical resources are deleted
- Template
    - Has Logical Resource NAME and TYPE
    - Contains resource properties
        - Once the logical resource moves to create_complete (physical resource is active) THEN the template logical resource can query attributes of the physical resource, like the ec2 ID
- Creating a stack
    - CreateStack uses template, parameters, and options to create a stack
- A Stack creates, updates, deletes physical resources based on logical resources in the template


CloudFormation Template and Pseudo Parameters


CloudFormation Intrinsic Functions


CloudFormation Mappings


CloudFormation Outputs

