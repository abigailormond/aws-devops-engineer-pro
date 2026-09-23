
## Scenario - Animals4life
- Small on-prem datacenter
- Based in brisbane, AUS
- 3 other offices: london, NYC, seattle
- Current problems:
	- legacy on prem hardware failing
	- lack of HA and scalability
- Ideal outcomes:
	- fast performance for all field workers
	- able to quickly deploy to new regions
	- low cost & scalable

## AWS Accounts

- hold identities & resources
	- identities -> users
	- resources -> provisioned 
- require unique email address, and a credit card
- Account Root User -> unique to this account, can only access this one account
	- initial user created
	- full control over the entire account!! access can't be restricted
	- BE CAREFUL
	- best practice is to only use this for initial account setup, then use iam identity after that
- IAM -> identity and access management, can be created and given full or limited access rights
	- Users, groups, roles

## MFA
- Factors -> different pieces of evidence that prove identity
- Types of factors:
	- knowledge --> something you know
		- usernames, passwords
	- possession --> something you have
		- bank card, mfa device/ app
	- inherent --> something you are
		- fingerprint, face, voice, iris
	- location --> where you are
		- physical location, what network logged into 
- In AWS, activate MFA for a user
	- aws generates secret key & additional user information, this generates qr code, you scan that on mfa application, which adds key & info to app, now app can generate mfa codes to use 

## IAM Basics
- Least Privilege Access
- IAM = globally resilient service (any data always secure across all aws regions!)
	- every account has their own dedicated instance of IAM
- 3 types IAM identity objects:
	- User = represent humans or applications that need access
	- Group = collection of related users
	- Role = can be used by aws services, or for granting external access to your account
		- generally used when # of entities granted access is uncertain (as opposed to users or groups)
		- ex: role allowing `S3` access, grant that to `EC2` instances
- Policy or policy document
	- allow or deny access to AWS services ONLY when attached to IAM users, groups, and roles
- 3 jobs of IAM
	- manage identities -- ID provider (IDP)
	- authenticate the identites 
	- authorize -- allow or deny access to resources
- basics
	- no cost
	- global service & globally resilient
	- ALLOW or DENY the identities on tis own account
	- no direct control on external accounts or users 
	- Identity federation & MFA
		- identity federation -- use existing identities like workplace identities, facebook, etc

## IAM Access Keys
- Type of Long-Term Credentials
- all long-term credentials used with IAM users
- generally users use user/pass on console UI, access key in CLI
- long-term indicates doesn't rotate or change often
- IAM users have 1 username 1 password
	- password optional! some iam users don't log in 
- IAM users can have 2 access keys (useful for rotating keys)
- access keys 
	- can be created, deleted, inactive, or active
		- can't be modified -- if compromised, delete, make new one (**Rotate access key**)
	- 2 parts:
		- access key id --> public part
		- secret access key --> private part
- 

## IAM Identity Policies
- set of statements granting or denying access to aws resources, granting this to specific identities
- attached to specific identities
- IAM Policy Document ---> 1+ statements
	- satements do the allow/denying
- Elements of statement
	- `Sid` -- statement ID -- describes whats going to be in the statement
	- `Effect` -- `allow`/`deny`
	- `Action`
		- format `[service:resource]`
			- can list specific action, or wildcard `*` to match any action/operation
		- or can be list of many specific actions
	- `Resource` -- can be wildcard, or specific resource using an AWS ARN
- It is possible to be allowed and denied at same time!! 
	- ex: full `S3` allow, the statement afterwards has deny for specific bucket
	- what happens? --> both of the statements are applied.
	- RULES: (order of priority)
		1. EXPLICIT DENY overrules everything else
		2. EXPLICIT ALLOW -- these take effect unless there is also an explicit deny. The deny takes priority over the allow
		3. DEFAULT DENY -- implicit -- denied unless specifically granted access
- When an identity tries to access a resource, AWS gathers all statements that are related to that identity (user, any group policies, any service policies) and evaluates them all together 
- Types of Policies
	- Inline Policies -- applied to individual identity 
		- used for special or exception access rights
	- Managed policies -- create policy then attach the policy to any # of identities 
		- reusable
		- low management overhead
		- 2 types
			- AWS managed policies
			- custom


## IAM Users and ARNs


IAM Users
- Identity used for anything requiring long-term AWS access (humans, applications, or service accounts)
- Principal -- entity trying to access an aws account. 
	- must be authenticated and authorized 
	- principal makes requiest to iam to be able to access resources
- Authentication -- principal proves it is an identity that it claims to be
	- long-term credentials: username/password, access keys
- Authorization -- IAM checking statements that apply to that identity 
* You can only have 5,000 IAM Users per account
* an Iam User can be a member of 10 groups
* IAM Roles & Identity Federation can address the above limitations

ARN
- Amazon Resource Name
- uniquely identitfy resources within any AWS accountws
- globally unique
- format:
	- `arn:partition:service:region:account-id:resource-id`
	- `arn:partition:service:region:account-id:resource-type/resource-id`
	- `arn:partition:service:region:account-id:resource-type:resource-id`
	- `arn:aws:s3:::catgifs`
	- don't need to specify region or accountid because s3 bucket names are globally unique
	- resource is the BUCKET
- `arn:aws:s3:::catgifs/*`
	- resource is the OBJECTS IN THE BUCKET not the bucket itself
- wildcard `*` -- refers to all
- double colon `::` -- when something doesn't need to be specified or isn't applicable
- 


## IAM Groups
- containers for organizing IAM users
	- can't log in, don't have credentials
- can have inline and managed policices attached
- No limit on # IAM Users in a group (except that there can only be 5000 users in an account)
- No built-in all-users group in IAM, but you could make one, but you would have to manage it yourself
- No group nesting
- Limit of 300 groups per account, can be increased with support ticket
* groups are not a true identity -- cannot be referenced as a principal in a policy. 
* just organizing users and assigning policies to groups that the iam users will inherit

## DEMO - Perms IAM Grops
- cloudformation outputs vs parameters?


## IAM Roles -- The Tech
- Used when multiple or unknown # of principals are accessing the account
	- Ex: multiple users in a company using the role
	- applications or services using the role
- IAM Roles are **assumed** .. you **become** the role
	- assumed for period of time when authenticated, then stops
- Role represents a level of access within the account rather than representing the principal identity long-term
- 2 types of policies attached to roles
	- Trust Policy
		- what identities are allowed to assume the role 
		- can reference other identities in the same account (users, other roles, services like `EC2`)
		- Can reference identities in other aws accounts
		- can allow other types of identities -- like google credentials
	- Permissions Policy
- When someone assumes a role, **temporary security credentials are generated by AWS STS** (Secure Token Service) -- `sts:AssumeRole`

## When to User IAM Roles
- AWS Services
	- Most common use of roles: for aws services. services operate on your behalf and need access to do so
	- Ex: `Lambda`, like most AWS services, has no permissions inherently
		- `Lambda Execution Role`
			- Trust Policy that trusts the Lambda service --> lambda is allowed to assume that role whenever a function is executed
			- Permissions policy that allows access to other AWS services
		 - When function runs...
			 1. Uses `sts:AssumeRole` operation
			 2. `STS` generates temporary security credentials for the `Lambda` runtime environment
			 3. lambda runtime env uses these credentials to do whatever it needs to do
		- If didn't use a role, would have to hardcode access keys, etc, for the function to use -- less secure
- Emergency, out of usual situations
	- "Break glass"
		- barrier from normal access, but available for emergencies 
- Existing Identities
	- if existing `Active Directory` / `SSO`
	- if >5000 identities
	- External identities can't directly access AWS directly, but can be granted access by allowing a role to be assumed by an external identity. It then generates temp credentials to the external identity can access the AWS resource
	- **Identity Federation** -- system of trust between separate organizations that allows users to access applications/resources using a single set of credentials 
- Cross-Account Access


## Service-linked Roles & PassRole
- Service-linked Roles
	- an IAM role linked to a specific AWS service
	- Provides set of permissions that are predefined by the service
	- Providing permissions that a service needs to interact with other AWS services on your behalf
	- How is a service-linked role made? 3 options:
		- the service might create/delete the role itself
		- or the service might allow you to create it during the setup process of the service
		- or could get created within IAM
	- **cannot be deleted until its no longer required** AKA no longer used within that service
	- Management
		- DO NOT TRY TO GUESS THE SERVICE-NAME IN THE ARN OF A RESOURCE STEMENT it can differ and is case sensitive 
	* add example  of role policy statements for service linked role 
- PassRole
	- method in AWS which gives you the ability to implement role sepearation 
	- can be used with service-linked roles 
	- give passrole to user, then user can attach a role to a service? this can let bob pass a role to lambda and that role lambda has may have more permissions than bob himself has. 
	- important aws security architecture


## AWS Organization
- Steps to create
	1. Log into standard AWS account (an account that is not already linked to an org)
	2. Create an organization from within this AWS account. This organization isn't created INSIDE the account, the account is just used to create the organization
	3. that account used to create it now becomes the Management Account (formerly master account) for the organization
	4. Management account can now invite other standard aws accounts into the org
	5. Those standard accounts must approve the invite to join the organization, and now become Member Accounts
- Organization has 1 Management account and >=0 Member Accounts
- Hierarchal structure 
	- top of tree is root container, Organization Root 
		- != account root user !!!!
		- organization root is just container within an organization, which can contain aws accounts (member accounts or management account)
	- Organization root can also contain Organizational Units (OU) that can contain accounts or more OUs
- Consolidated Billing
	- member accounts pass billing thru to the management account "payer account"
		- management account = master account = payer account
- Consolidation of reservations and volume discounts
	- save $
- Service Control Policies (SCPs)
- Can directly create new accounts within an organization -- skip step of invite + accept to become member account
	- just need unique email 
	- also automatically creates role within the member account that allows role switching from the management account: `OrganizationAccountAccessRole`
		- the `OrganizationAccountAccessRole`
			- principal: management AWS account
			- action: `sts:AssumeRole`
			- effect: `allow`
			- can give it whatever permissions you want
		steps to switch role for first time:
			1. click username
			2. click switch roles
			3. type in AWS account of account you want to switch into
			4. type in name of role (default `OrganizationAccountAccessRole` unless you made something diff)
			5. type display name to make shortcut to more easily access this role switch in future
		in future, you can now just click on the shortcut to quickly switch roles
- best architectural practice is to have one aws account handle identities
		1. existing identity group authenticates through identities in one account
		2. role switching to assume roles within other accounts in the org
	- a lot of this happens behind the screen 

## Service Control Policies (SCPs)
- feature of aws organizations
	- SCPs can be applied to organizations ("root container"), `OUs`, or to individual accounts
- Mangagement accounts cannot be affected by SCPs
	- avoid putting resources in here!
- SCPs are account permission boundaries
	- limit what account can do (including account root user)
		- account root user always has full access over entire aws account, but you can restrict the allowed perms on the entire account via SCPs, which in effect also restricts account root user
	- examples
		- only certain size of ec2 instance allowed
		- only certain regions allowed
	- DON'T GRANT ANY PERMISSIONS - defines boundaries and limit permissions
		- control what an account CAN and CANNOT grant via identity policies
- ALLOW list vs DENY list
	- deny list -> DEFAULT ALLOW, scp adds list of "denies"
		- this is the default structure - full access, no boundaries on an aws account
		- lower admin overhead
	- allow list -> DEFAULT DENY , scp adds list of "allows"
	- any additional policies follow (deny, allow, deny) regular policy priorities
- ONLY something that is allowed by an SCP and granted access by identity policy would be actually allowed. middle of venn diagram 
	- 