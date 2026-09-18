
Scenario - Animals4life
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

AWS Accounts

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

MFA
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

IAM Basics
- Least Privilege Access
- IAM = globally resilient service (any data always secure across all aws regions!)
	- every account has their own dedicated instance of IAM
- 3 types IAM identity objects:
	- User = represent humans or applications that need access
	- Group = collection of related users
	- Role = can be used by aws services, or for granting external access to your account
		- generally used when # of entities granted access is uncertain (as opposed to users or groups)
		- ex: role allowing s3 access, grant that to ec2 instances
- Policy or policy document
	- allow or deny access to aws services ONLY when attached to IAM users, groups, and roles
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

IAM Access Keys
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

IAM Identity Policies
- set of statements granting or denying access to aws resources, granting this to specific identities
- attached to specific identities
- IAM Policy Document ---> 1+ statements
	- satements do the allow/denying
- Elements of statement
	- Sid -- statement ID -- describes whats going to be in the statement
	- Effect -- allow/deny
	- Action 
		- format [service:resource]
		- can list specific action, or wildcard * to match any action/operation
		- or can be list of many specific actions
	- Resource -- can be wildcard, or specific resource using AWS ARN
- It is possible to be allowed and denied at same time!! 
	- ex: full s3 allow, the statement afterwards has deny for specific bucket
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