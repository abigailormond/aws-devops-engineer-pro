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
    - vpc-based lambda functions don't actually run within your vpc??