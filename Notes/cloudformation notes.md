MISC
- AMI are specific to region

## CloudFormation Physical & Logical Resources
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


## CloudFormation Template and Pseudo Parameters
- Template Parameters
	- human or process can provide input when a stack is created or updated
		- CLI
		- API
	- parameters can be referenced within logical resources, influencing physical resources / configuration'
	- Can be configured with:
		- defaults
		- allowedvalues
		- min/max length
		- allowedpatterns
		- noecho (don't want to be visible when typed, like password)
		- variable type
- Pseudo Parameters
	- injected values into the template & stack
	- Examples
		-  AWS:: Region -- always references the region the stack is being created in
		- AWS::StackName and AWS::StackId match specific stack being created
		- AWS::AccountId populated  by AWS to the actual account ID 
- Both template & pseudo parameters can work in tandem
- Best practice
	- use defaults where possible, get values from aws, minimize manual input into the template parameters 

	

## CloudFormation Intrinsic Functions
- allow you to gain access to data at runtime
- Ref 
	- reference
	- !Ref on template or psuedo parameters returns their value. 
		- when used with logical resources, physical ID is usually returned.
		- Ex: !Ref Instance --> i-123456asbcdef0
- Fn::GetAtt
	- get attribute
	- both allow you to reference value from one logical resource into another one
	- !GetAtt LogicalResource.Attribute
		- retrieve any attribute associated with the resourece, like publicIP, or publicdns name
- Fn::Join & Fn::Split
	- Split
		- takes string and outputs a list
	- Join
		- takes list and joins them to create a string
- Fn::GetAZs 
	- get list of availability zones in region, select from a list
	- portable templates! hardcoding AZ is no no
	- !GetAZs "us-east-1" or "" (current region)
- Fn:: Select
	- allows you to reference an item in a list using an index
- Conditions (if, and, equals, not, or)
- Fn::Base64 
	- base64 encoding, substitute within text 
	- ex: if userdata requires base64, pass your normal text into Fn:Base64 then into your userdata
- Fn:: Sub
	- substitute in variables
- Fn::Cidr
	- configure subnet ranges
	- pass in:
		- cidr block
		- how many subnets to generate from input VPC range
		- bits per CIDR 
	- outputs: 

## CloudFormation Mappings
- feature of cloudformation that makes it easier to design portable templates
- Templates can contain a Mappings object
- Mappings object can contain many mappings
- mappings map keys to values, allowing lookup
- Can have one key, or Top & Second Level 
	- !FindInMap \[ mapName, key ]
	- !FindInMap \[ mapName, topLevelKey, secondLevelKey ]
- Use !FindinMap Intrinsic function (commonly used to retrieve AMI for given region, architecture)

## CloudFormation Outputs
- optional, useful for providing status information 
- declare values that will be used as output when using cli or console UI
- can also be accessible from parent stack when using nesting
- can be exported, allowing cross-stack references
- Configuring
	- Description: is visible from CLI and console UI and passed back to parent if in nested stack
	- 
See below image
dynamic parameter fault that will evaluate and show as default to user in parameter field 

![[Pasted image 20260922202654.png]]