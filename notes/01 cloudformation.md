MISC
- AMI are specific to region

## CloudFormation Physical & Logical Resources
- CloudFormation Template
	- `JSON` or `YAML`
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
		- Once the logical resource moves to `CREATE_COMPLETE` (physical resource is active) THEN the template logical resource can query attributes of the physical resource, like the `EC2` ID
- Creating a stack
	- `CreateStack` uses template, parameters, and options to create a stack
- A Stack creates, updates, deletes physical resources based on logical resources in the template


## CloudFormation Template and Pseudo Parameters
- Template Parameters
	- human or process can provide input when a stack is created or updated
		- CLI
		- API
	- parameters can be referenced within logical resources, influencing physical resources / configuration
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
		- `AWS::Region` -- always references the region the stack is being created in
		- `AWS::StackName` and `AWS::StackId` match specific stack being created
		- `AWS::AccountId` populated by AWS to the actual account ID
- Both template & pseudo parameters can work in tandem
- Best practice
	- use defaults where possible, get values from AWS, minimize manual input into the template parameters

	

## CloudFormation Intrinsic Functions
- allow you to gain access to data at runtime
- `Ref`
	- reference
	- `!Ref` on template or pseudo parameters returns their value.
		- when used with logical resources, physical ID is usually returned.
		- Ex: `!Ref Instance` --> `i-123456asbcdef0`
- `Fn::GetAtt`
	- get attribute
	- both allow you to reference value from one logical resource into another one
	- `!GetAtt LogicalResource.Attribute`
		- retrieve any attribute associated with the resource, like publicIP, or publicdns name
- `Fn::Join` & `Fn::Split`
	- Split
		- takes string and outputs a list
	- Join
		- takes list and joins them to create a string
- `Fn::GetAZs`
	- get list of availability zones in region, select from a list
	- portable templates! hardcoding AZ is no no
	- `!GetAZs "us-east-1"` or `!GetAZs ""` (current region)
- `Fn::Select`
	- allows you to reference an item in a list using an index
- `Conditions` (`Fn::If`, `Fn::And`, `Fn::Equals`, `Fn::Not`, `Fn::Or`)
- `Fn::Base64`
	- base64 encoding, substitute within text
	- ex: if `UserData` requires base64, pass your normal text into `Fn::Base64` then into your `UserData`
- `Fn::Sub`
	- substitute in variables
- `Fn::Cidr`
	- configure subnet ranges
	- pass in:
		- CIDR block
		- how many subnets to generate from input VPC range
		- bits per CIDR 
	- outputs: 

## CloudFormation Mappings
- feature of CloudFormation that makes it easier to design portable templates
- Templates can contain a Mappings object
- Mappings object can contain many mappings
- mappings map keys to values, allowing lookup
- Can have one key, or Top & Second Level 
	- `!FindInMap [mapName, key]`
	- `!FindInMap [mapName, topLevelKey, secondLevelKey]`
- Use `!FindInMap` intrinsic function (commonly used to retrieve an `AMI` for a given region and architecture)

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


## CloudFormation Conditions
- optional 'conditions' section of template -- can contain many conditions
- conditions evaluated true or false
- conditions process before resources are created
- utilize intrinsic functions and, =, if, not, or
- example use cases:
	- control how many AZ's to create resources in, size of instance, etc
- when a condition is attached to a resource, that resource will only be created if the condition evaluates to true
- process:
	1. create a conditions block
	2. when stack is being created, condition block is evaluated -- now that condition, ex `isProd` is evaluated to true or false
	3. proceeds to processing resources. if a condition is present in the resource block, it is compared against the previously evaluated condition. if that condition is true, it will create the resource. if it is false, it will skip

## CloudFormation DependsOn
- CloudFormation naturally does things in parallel (create, update, delete)
	- attempts to determine dependency order automatically (VPC -> subnet -> EC2)
		- implicit dependency: if EC2 references a subnet, CloudFormation knows it needs to make the subnet first
			- also affects deletion order, in reverse
- DependsOn lets you explicitly define dependencies 
	- why/ when?
		- elastic IP requires an IGW attached to a VPC to work, but there may not be an inherent internal dependency (!Ref) for that, best to add explicit dependency to prevent an error

## CloudFormation Wait Conditions & cfn-signal
- CloudFormation
	logical resources in template -> stack -> stack creates physical resources -> tells logical resource CREATE_COMPLETE
- problem: need more detailed signalling
### cfn-signal
- configure CloudFormation to wait for # success signals, then CREATE_COMPLETE flagged to signal to the logical resource
	- if failure signal received (max 12H) -> creation fails
	- if timeout reached -> creation fails
### Creation Groups
- applies signal requirement -- stack needs # signals and has # amount of time to receive them
- more detailed requirements for CREATE_COMPLETE or CREATE_FAILED
### Wait Conditions
- allow PAUSE AND WAIT between resource creation
- its own logical resource that will have its own CREATE_COMPLETE
- can depend on other resources, and other resources can depend on it
- implicit depends on another resource -- WaitHandle
	- WaitHandle is its own resource
	- generates presigned URL for resource signals to be sent to 


## CloudFormation Nested Stacks
- isolated CloudFormation stack
	- has all the resources within itself and they all share a lifecycle
	- LIMITS
		- 500 resources per stack
		- can't easily reuse resources e.g. VPC (can't reference in another stack)
		- 
- Nested Stack
	- root stack -> created first, manually
	- parent -> parent of any stacks that it immediately creates
	- root stack will also be the parent of any nested stacks
- Can create nested stack within template of parent stack
	- must supply parameters for any values used to create child stack
	- must supply URL to template ( * template is reusable )
- outputs of child stack are returned to parent stack 
- parent stack can't reference logical resources of child stack but CAN reference outputs from the child stack (same for sibling stacks)
- nested stacks can depend on other sibling nested stacks to affect order of creation
- root/parent stack won't be marked CREATE_COMPLETE until all its children are CREATE_COMPLETE
- use nested stacks when 
	- want to overcome 500 resource limit 
	- want to modularize templates for code reuse
	- make stack installation easier (apply root stack -> automatically create many nested stacks)
* only use nested stacks when everything is lifecycle linked 
	- created together, deleted together
* nested stacks allow for reuse of templates

## CloudFormation Cross-Stack References
* can use when things are not lifecycle linked
	- not created and deleted together
	- ex: long lifecycle VPC, shorter lifecycle applications running on it
- cfn stacks are designed to be isolated and self-container
	- normally outputs are not visible between stacks (except nested stacks)
- cross-stack references allow reuse of resources between stacks
- outputs can be exported making them visible from other stacks 
- exports must have unique name within the region
- export VPC IDs, instance ID, etc
- to use export, instead of using Ref!, use Fn::ImportValue with the export name to get the value from the export
- exports are listed under outputs
* cross-stack references allow reuse actual physical resources

## CloudFormation Stack Sets
- deploy cfn stacks across many accounts and region, without having to separately authenticate into each
- StackSet = container in an admin account, container for stack instances
	- ...contain stack instances, which reference one particular account in one particular region in one particular AWS account
	- if stack fails to create, stack instance remains
	- stack instance = container for 1 stack
- concurrent accounts: defined value, the more you set, the faster resources are deployed. How many accounts can be deployed to at the same time
	- ex: if you're deploying stackset into 10 accounts, concurrent account set to 2, then will be deploying 5 sets of 2 accounts at a time
- failure tolerance: amount of individual deployments which can fail before stackset itself is viewed as failed
- retain stacks: remove stack instances from a stackset, by default will delete the actual stacks themselves, but can change settings to keep the actual stack
- uses
	- enable AWS config
	- create IAM roles for cross-account access
	- AWS config rules - MFA, EIPS, EBS encryption