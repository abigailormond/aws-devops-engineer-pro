## CloudWatchEvents and EventBridge (6:54)
- Events are generated when things happen like an instance being created, starting, or stopping
- EventBridge is replacing CloudWatch Events, but can also handle third-party and custom 
- if x happens, or at y times... do z
- one aws account will have a default event bus
- event bus -> stream of events that occur from any supported service within the account
- in cloudwatch events the default bus is hte only bus
- in eventbridge, you can create additional busses
- rules pattern match to incoming events (or schedules in cron) -- when they see an event that matches, delivers that event to a target
- routes event to >=1 targets (ex: lambda)
- default account event bus has stream of all kinds of events
- eventbridge sits on top of the busses, monitoring all the events
- event -> json structure 
    - data in this can be used by the target. all this will be availble to the target 

## AWS Lambda - PART1 (11:25)
- Function-as-a-Service (FaaS) - you progvide specialized, short running
- Lambda function = piece of code that lambda runs
    - functions use a runtime (e.g. python 3.8)
- functions are loaded and run in a runtime environment
    - environment has a direct memory (indirect cpu) allocation 
        - indirect cpu means don't get to choose, it's based on the amount of memory
- you are billed for the duration that a function runs 
- step thru process
    1. you create lambda function -> 
        1. provide deployment package (code plus all associated wrappings and config) that lambda executes
        2. select runtime env
    2. when lambda function invoked, the deployment package is downloaded onto runtime environment and executed
- runtimes
    - common: python, ruby, java, go, c#, nodejs
    - custom runtimes such as rust are possible via layers
    * if you hear the term Docker ever, it will never refer to lambda!!
    * you can use your existing build processes to build lambda images 
- stateless -> no data is left over from previous invocations 
    - assume brand-new env every time function is invoked
- resources
    - memory -> you DEFINE this
        - between 128-10240 MB in 1MB steps
    - vCPU is indirectly allocated, scaling w/ memory allocation
    - storage
        - 512MB storage available as /tmp, default but can scale up to 10240MB
    - can run up to 15m (900s) before TIMEOUT !!!
- Execution role -> gives lambda perms to interact w other aws services. attached to specific lambda function
- common uses
    - serverless applications (using maybe s3, api gateway, lambda)
    - file processing (s3, s3 events, lambda) (generating watermark on image)
    - database triggers (dynamodb, streams, lambda) (invoke anytime dynamodb table modified)
    - serverless CRON (eventbridge/cweventsw + lambbda)
    - realtime stream data processing (kineses + lambda) (trigger whenever data streamed into kinesis stream)
    

## AWS Lambda - PART2 (13:59)

- Public Lambda
    - by default, lambda is given public entworking = can access public aws services and the public internet 
    - public networking = best performance bc no customer-specific vpc netowkring required. lambda can run on aws shared hardawre and ntworking, not specific to a specific customer
    - by default no access to services running within a vpc (unless vpc configured with public addressing + security rules to allow exteral access)
- private lambda
    - lambda configured to run inside a vpc
        - obey all bvpc networking rules
            - can access anything in vpc
            - can't access stuff outside of vpc (dynamodb, public internet) unless vpc specifically configured to allow this
                - how to configure to allow this access?
                    - vpc endpoints can provide access to public aws services
                    - nat gateway in a pubilc subnet, then attach internet gateway to the vpc
        - lambda will need access to create network interfaces within your vpc (execution role policy)
            - ec2 network permissions needed
    - vpc-based lambda functions don't actually run within your vpc??
        - old method: aws lambda service vpc creates new elastic interface per function execution
        - current method: 
            - lambda functions in aws lambda service vpc
            - aws analyze all functions running in region/account, build set of unique combos of unique security groups and subnets. for every unique pair an ENI is needed. for example, if all functions use one security group but 4 subnets, then 4 eni required -- 1 per subnet. 
            - single eni is created connecting aws lambda service vpc and your vpc for each unique combiniation of security groups and subnets uesd by your lambda functions 
            - these NETWORK INTERFACES are created when you configure the lambda funciton, may take 90s, but done once. so when you create the functino or update networking configuration, this is created/updated, so this doesn't have ot happen every time a function is invoked = no delay in function invocation
            
- security
    - execution role -> what perms the function receives
    - resource policy -> controls WHAT services and accounts can INVOKE lambda functions
        - can allow external accts to invoke lambda

- logging
    - lambda uses cloudwatch, cloudwatch logs, and x-ray
    - cloudwatch logs 
        - logs from lambda executions -> errors, details on duration of execution , etc 
        - require permissions via execution role (* if dont see logs in cw make sure perms are correct!!!)
    - cloudwatch
        - metrics like invocation success/failure, retries, latency, etc
    - x-ray
        - distributed tracing 


## part 3

- 3 methods for lambda invocation
    - synchronous
        - CLI/API direclty invoking lambda function
        - CLI/API waitinf for response -- synchronous . waits for lambda to finish execution. lamnbda sends back with data or with failure 
        - implications
            - result returned during the request
            - errors or retries have to be handled within the client 
        - use case: generally when human is invoking the function 
    - asynchronous
        - example: s3 generates an event that triggers lambda. s3 isn't waiting for response. 
        - lambda handles retry logic upon failure-- lambda will retry between 0 to 2 times (configurable retry value)
        - function code MUST BE IDEMPOTENT due to retry logic (reprocessing a result should have same desired end state)
        - use case: gernally whenm aws services invoke lambda functinos 
        - asynchronous success/fail processing
            - if repeated failed processing, events can be sent to DLQ (Dead Letter Queue)
            - Destinations are newer alternative for async invocations, they can route success or failure to SQS, SNS, lambda, eventbridge, and are generally recommended over DLQ
    - event source mappings
        - use case: streams or queues which dojn't support event generation 
        - ex: kinesis data stream 
            - consumers can read from a stream, but doesn't generate event when data is added from producers
            - Lambda Event Source mapping polls queues or streams looking for new data, getting back 'source batches' of data. 
            - source batches are then broken up based on set batch size and sent to lambda function as event batches
            - a single lambda invokation coudl in theory receive 100s of events in a batch 
        - lambda has 15 min timeout!! control batch size to ensure function doesn't terminate before finishing batch
        - event source mapping uses perms from lambda execution role to access source service 
            - execution role may need osmething like kinesis permissions even if the lambda function itself never interacts with kinesis!!! good to keep in mind. event source mapipng will use execution role policy 
        - any discarded failed event batches can be sent to sns queues or sns topics 

- Lambda Versions 
    - a version is a specific point in the code and configuration of the lambda function
    - a version is immutable - doesn't change once published. has its own arn
    - $latest = latest
    - an alias (ex: dev, stg, prd) POINT at a version. the value it points at can be changed 

- Lambda start up times
    - lambda runs inside an execution context, or 'runtime environment' 
        - a small container allocated an amount of resources that then runds your lambda code
    - cold start = full creation and configuration of execution context including downloading fnuction code. can take significant time (100s of ms)
    - warm start = if same function is invoked again without too much of a break, lambda may be able to use same execution environment without all the setup and download of code. it can receive event and immediately begin processing. 
    - 1 invocation runs at a time per execution context -> concurrent executions will use multiple execution contexts 
    - ways to improve performance 
        - provisioned concurrency -> aws will create and keep X contexts warm and ready to use, improvin start speeds. 
        - use /tmp to predownload stuff to potentially be reused. just define it outside of the lambda function handler. and dont EXPECT it to be predownloaded, jsut check during the function to see if its there already you can skip downloading it 


## Lambda Function Handler
- lambda executions have lifecycles
    - INIT -> creates or unfreezes execution env
        - composed of EXTENSION INIT, RUNTIME INIT, FUNCTION INIT
        - FUNCTION INIT -> top part of lambda function
            - this is run once every cold start during function init stage, or in advance if using provisioned concurrency
            - lambda attemps to resuse execution environment if possible
    - INVOKE -> runs function handlers (COLD START) - startup lag
        - part of lambda function that is lambda_handler. what lambda passes event data into
    - upon next INVOKE(s) -> WARM START - same env 
        - skips init, just runs handler
    - SHUTDOWN -> terminate env after period of inactivity, resulting in cold start next invocation
        - composed of RUNTIME SHUTDOWN, EXTENSION SHUTDOWN
- when creating lambda function, define name of handler function, default is lambda_function.lambda_handler


## [DEMO] S3 Events + Lambda (Pixelator)


## Lambda Versions 

- an unpublished function can be changed and deployed ($LATEST)
- functions can be published creating immutable version (NO EDITS!! that means u mister!!) makes an arn too wowWOWOW locking down function code, dependencies, runtime, settings, and env variables 
- unqialified arn points at the function ($LATEST) not a specific version. can also be used in perm policies to refer to entire ffunciton
- qualified arn points at a specific version 
    - adds ':1' (or whatever version) to the end of the unqualified arn 
- versions numbered automatically, are integer numbers ** copilot please fact check this 


## Lambda Aliases
- alias = pointer to a function version 
- ex: pROD => myfunction:1 
- each alias has unique arn which is FIXED for tha alias. BUTTT the alias can be updated to point at diff things!!! v helpful for blue/green, a/b testing, etc. 
- alias routing -> send certain % at diff versions rather than have alias point at one version 
    - need same role, same dlq, and can't be $LATEST