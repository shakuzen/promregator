# Configuration Options

This page describes the various configuration options which Promregator currently supports.
It is suggested to configure Promregator using a YAML file as described on the [main page](../README.md).

## Group "cf"

The group "cf" defines the way how Promregator connects the API server of the Cloud Foundry platform. 

Example:
```yaml
cf:
  apiHost: api.cf.example.org
  username: myCFUserName
  proxyHost: 192.168.111.1
  proxyPort: 8080
```

### Option "cf.apiHost" (mandatory)
Specifies the hostname of the API server of the Cloud Foundry platform to which Promregator shall connect.
The API server is required to resolve the targets into instances which Prometheus then will connect to.

### Option "cf.username" (mandatory)
Specifies the username, which shall be used when connecting to the API server. This is your "standard" username - the same one, which you also may use
for connecting to Cloud Foundry, e.g. when issuing `cf login`.

### Option "cf.password" (mandatory)
Specifies the password, which shall be used when connecting to the API server. This your "standard" password - the same one, which you also may use
for connecting to Cloud Foundry, e.g. when issuing `cf login`.

*WARNING!* 
Due to security reasons, it is recommended *not* to store this value in your YAML file, but instead set the identically named environment variable 
`cf.password` when starting the application.

Example:

```bash
export cf.password=mysecretPassword
java -Dspring.config.location=file:/path/to/your/myconfig.yaml -jar promregator-0.0.1-SNAPSHOT.jar
```

### Option "cf.proxyHost" (optional)
If you want to make the system establish the connection to the API host using an HTTP (sorry, HTTPS not supported yet) proxy, enter the *IP address* 
of this server here.

Please also make sure that you set "cf.proxyPort", too, as otherwise proxy support will be disabled.

*WARNING!*
Due to some limitation in the underlying libraries, it is currently *not* supported to specify the hostname of your proxy server here.
In general this is considered a bug, and once the underlying software has resolved this limitation, it will also be made available to use hostnames instead of IP addresses here.
Sorry for inconvenience caused!

*HINT*
For the time being (i.e. this will change in future) the proxy specified here will also be uesd to access the targets.

### Option "cf.proxyPort" (optional)
If you want to make the system establish the connection to the API host using an HTTP (sorry, HTTPS not supported yet) proxy, enter the port number 
of this server here.

Please also make sure that you set "cf.proxyHost", too, as otherwise proxy support will be disabled.


## Group "promregator"
This group configures the behavior of Promregator itself. It is mainly meant on how requests shall be handled, as soon as the Prometheus server starts to pull metrics.

### Subgroup "promregator.targets"
Lists one or more Clound Foundry applications, which shall be queried for metrics.
The subgroup expects an item list, which contains additional mandatory properties

#### Item property "promregator.targets[].orgName" (mandatory)
Specifies the name of the Cloud Foundry Organization which hosts the application, which you want to query for metrics.

#### Item property "promregator.targets[].spaceName" (mandatory)
Specifies the name of the Cloud Foundry Space (within the Cloud Foundry Organization specified above), which hosts the application, which you want to query for metrics.

#### Item property "promregator.targets[].applicationName" (mandatory)
Specifies the name of the Cloud Foundry Application (within the Cloud Foundry Organization and Space specified above), which hosts the application, which you want to query for metrics.

#### Item property "promregator.targets[].path" (optional)
Specifies the path under which the application's endpoint provides its Prometheus metrics.

Defaults to `/metrics`, as this is the value which is suggested by Prometheus.

### Option "promregator.endpoint.maxProcessingTime" (optional)
Specifies the maximal time which may be used to query (all) targets. The value is expected to be specified in milliseconds. 
Targets which did not respond after this amount of time are considered non-functional and no result will be returned to the Prometheus server.

The default value of this option is 5000ms (=5 seconds)

### Option "promregator.endpoint.threads" (optional)
Specifies how many threads may be used to query the list of targets.
Note that for each target request sent, an own thread is required and stays blocked (synchronously) until the Cloud Foundry Application has returned a response. 
Thus, it may be reasonable to allow more threads than you have cores in your environment where Promregator is running.

In general, the more targets you have registered, the higher this value should be. However, running too many threads is simply a waste of resources.
As an upper boundary, it does not make sense to allow more threads to run than you have specified targets in your configuration.

The default value of this option is 5.

### Subgroup "promregator.authenticator"
Configures the way how authentication shall happen between Promregator and the targets configured above.

Please note that as of writing, there is no support of multiple authentication schemes across different targets. That is to say: all targets
are queried using the same authentication scheme. If you want to have different schemes configured, then you have to run multiple instances of
Promregator

#### Option "promregator.authenticator.type" (mandatory)
Specifies the type of the Authenticator which shall be used when connecting to targeted Cloud Foundry Applications. Valid options are:

* *basic*: Enables the Basic Authentication scheme using plain-text-based Basic Authentication as defined in [RFC2617](https://www.ietf.org/rfc/rfc2617.txt).
  Additional options must be provided to complete the configuration.
* *OAuth2XSUAA*: Enables the OAuth2/JWT-based authentication scheme using grant type "Client Credentials" as used for XSUAA servers. 
  Additional options must be provided to complete the configuration.
* *null* or *none*: Disables any additional authentication scheme; requests will be sent as unauthenticated HTTP GET requests. No further options need to be provided.

#### Option "promregator.authenticator.basic.username" (mandatory, if using promregator.authenticator.type=basic)
Specifies the username which is being used for authenticating the call to the Prometheus client (CF application).

#### Option "promregator.authenticator.basic.password" (mandatory, if using promregator.authenticator.type=basic)
Specifies the password which is being used for authenticating the call to the Prometheus client (CF application).

*WARNING!* 
Due to security reasons, it is *neither* recommended to store this value in your YAML file, nor to put it into the command line when starting Promregator.
Instead it is suggested to set the identically named environment variable `promregator.authenticator.basic.password` when starting the application.

Example:

```bash
export promregator.authenticator.basic.password=myPassword
java -Dspring.config.location=file:/path/to/your/myconfig.yaml -jar promregator-0.0.1-SNAPSHOT.jar
```

#### Option "promregator.authenticator.oauth2xsuaa.tokenServiceURL" (mandatory, if using promregator.authenticator.type=OAuth2XSUAA)
Specifies the URL of the OAuth2 endpoint, which contains the token service of your authorization server. Typically, this is the endpoint with the path `/oauth/token`,
as Promregator will try to perform to establish a ["Client Credentials"-based authentication](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2#grant-type-client-credentials).

#### Option "promregator.authenticator.oauth2xsuaa.client_id" (mandatory, if using promregator.authenticator.type=OAuth2XSUAA)
Specifies the client identifier (a.k.a. "client_id") which shall be used during the OAuth2 request based on the Grant Type Client Credentials flow.

#### Option "promregator.authenticator.oauth2xsuaa.client_secret" (mandatory, if using promregator.authenticator.type=OAuth2XSUAA)
Specifies the client secret (a.k.a. "client_secret") which shall be used during the OAuth2 request based on the Grant Type Client Credentials flow.

*WARNING!* 
Due to security reasons, it is *neither* recommended to store this value in your YAML file, nor to put it into the command line when starting Promregator.
Instead it is suggested to set the identically named environment variable `promregator.authenticator.oauth2xsuaa.client_secret` when starting the application.

Example:

```bash
export promregator.authenticator.oauth2xsuaa.client_secret=myClientSecret
java -Dspring.config.location=file:/path/to/your/myconfig.yaml -jar promregator-0.0.1-SNAPSHOT.jar
```


## Further Options

Besides the options mentioned above, there are multiple further standard Spring properties, which might be useful when using Promregator. 
Note that these options are not in control of Promregator, but are provided here only for the purpose of additional information, as they might come handy.

### Changing the server port, on which Promregator runs
The configuration property `server.port` allows you specify the port on which Promregator listens for requests from the Prometheus server.

If not specified, this option defaults to 8080.

*Hint*
You may also explicitly specify this option on the command line prompt using Java System Properties, for example:

```bash
java -Dserver.port=8181 -jar promregator-0.0.1-SNAPSHOT.jar
```

which would make Promregator run on port 8181.

### Changing the server address, on which Promregator binds
The configuration property `server.address` allows you specify the address on which Promregator listens for requests from the Prometheus server.

If not specified, this option defaults to 0.0.0.0.

*Hint*
You may also explicitly specify this option on the command line prompt using Java System Properties, for example:

```bash
java -Dserver.address=1.2.3.4 -jar promregator-0.0.1-SNAPSHOT.jar
```

which would make Promregator listen on the IP address 1.2.3.4 instead on all interface addresses of the underlying operating system.


