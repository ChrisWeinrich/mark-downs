# 3 Microservices Elements

## Building a Monolith

webApp
Model

Pros: Simple to develop,build,test,deploy,scale  
Cons: New Team Member, Growing Teams, Code Hard to understand, No Emerging technologies, scale for bad reasons, overloaded container, huge database

## Building Microservices

Domain Driven Design

SubDomains
- User
- Order
- Product

## Organization

- Team per subdomain
- independent
- Responsible
- Agile and DevOps
- Versioning is important !

## Datastore

- Each Service has independent Data store
- Different requirements
- Relational
- NoSQL
- Graph/LDAP
- No distributed transaction
- Immediately consistent
- Eventual consistency
- Event Sourcing: Akka, Kafka, Rabbit MQ


## User Interface

- Independent Teams
- Oen set of components
- Unique UI
- Singe Application
- UI composition
- Server Side Composition
- Client Side Composition

## Services

### Remote PRocedure Invocation
- RPC
- Request/reply
- Sync
- Async
- Rest, SOAP, gRPC

### Messaging
- Message or Event
- Broker or channel
- Publish
- Subscribe
- Loosely Couple
- Kafka, Rabbit MQ

### Protocol Format Exchange
- Text
- XML,JSON,YAML
...

### APIs and Contracts

- Application program interface
- contract
- Sopa,mRest,gprec
- WSDL,Swagger,IDL

### APIS Contract per Device
- Different devices
- different needs
- different APIs
- different Contracts

## Distributed Services

### Service Registry

- Location Change
- Phone book of services
- Self registration
- Discovery
- Invocation
- Eureka, Zookeeper consule

### CORS

- Same-origin policy
- same protocol,server,port
- Restrict cross-origin
- additional http headers

### Circuit Breaker

- network failure
- heavy load
- domino effect
- Hystrix, Jrugged

Invoke via Proxy

### Gateway
- Singly Entry point
- Unified interface
- Unified interface
- cross-cutting concerns
- api translation
- Zuul,Netty, Finagle

### Security
- identity and access management
- Single Sign-on
- Kerberos, OpenID, Oauth 2.0,SAML
- Okta, Keylock, shiro
- ...

### Scalability and Availability
- Vertical
- Horizontal
- Ribbon,Meraki

=> Registry with load balancing

- Be operational
- Possible Single point of Failure: Gateway, Broker, Registry, IAM

### Monitoring
- many moving parts
- many machines   
- centralized
- visual
- Kibana, Grana, Splun

#### Health Check
- Service running
- Incapable handling requests
- Health check API
    - Database status
    - Host stats
    - ..
- heart bits

#### Log Aggragation
- Aggregate Logs
- LogStash, Splunk, PaperTrail

#### Exception Tracking
- Errors
- Throw an exception
- Record exceptions
- investigated and resolved

#### Metrics
- System slowing down
- performance issues
- gather statistics
- aggregate metrics
- DropWizard,Actuator, Prometheus

#### Auditing
- Behavior

#### Rate Limiting
- Third-party access
- Control API usage
- Limit traffic

#### Distributed Tracing
- Request span services
- Trace entire request
- logs, chain of calls
- ...
- Dapper, HTrace,Zipkin

### Deployment
- Physical Server
- VM
- On Premise
- In the cloud

#### Services per Host
- microservice per host

#### Containers
- Container image
- With dependencies
- packing microservice
- easy to move from environment
- scale up and down
- Docker, rkt

#### Orchestrator
- Multiple containers
- multiple machines
- start at the right time
- failed containers
- Kubernetes

#### CD
- Automate deployment
- cost-effective
- Quick
- Reliable
- Build, test, deploy
- ...

#### Environments
- production
- dev, test,QA, staging
- integration
- versioning

#### External configuration
- difference envirnments
- different configutation
- activate functionalty
---



























































