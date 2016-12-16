# Elytron client demo

## About

This demo uses WildFly `ModelControllerClient` to show, how to work with an Elytron-enabled WildFly client.

The demo application ([SimpleClient.java](src/main/java/org/wildfly/security/elytron/SimpleClient.java)) connects to a WildFly server and calls `:whoami` operation twice:

1. with default AuthenticationContext (from `wildfly-config.xml`)
1. with programmatically created `AuthenticationContext`

### wildfly-config.xml

The default context is [loaded](https://github.com/wildfly/wildfly-client-config/blob/master/src/main/java/org/wildfly/client/config/ClientConfiguration.java#L131-L170) by a discovery mechanism ([wildfly-client-config](https://github.com/wildfly/wildfly-client-config) GitHub project) and can be customized by a `wildfly.config.url` system property.

The Elytron part of `wildfly-config.xml` client configuration is described in Elytron XSD (e.g. [version 1.1.0.Beta17](https://github.com/wildfly-security/wildfly-elytron/blob/1.1.0.Beta17/src/main/resources/schema/elytron-1_0.xsd)).

### Elytron API

Entrypoint for programmatic configuration is the class [AuthenticationContext](https://github.com/wildfly-security/wildfly-elytron/blob/1.1.0.Beta17/src/main/java/org/wildfly/security/auth/client/AuthenticationContext.java).

The `AuthenticationContext` instance created in this demo contains following rules:

1. client connecting to `localhost` hostname is handled as `administrator`
1. any client is handled as `monitor`

## Prerequisities

### Build WildFly with Elytron integration

As for now (2016-12-15) you need to build WildFly from Elytron integration fork to get the demo working:

```bash
git clone -b ladybird git@github.com:wildfly-security-incubator/wildfly-core.git
git clone -b ladybird git@github.com:wildfly-security-incubator/wildfly.git
cd wildfly-core
mvn clean install -DskipTests -Dcheckstyle.skip -Denforcer.skip
WFLY_CORE_VERSION=$(mvn org.apache.maven.plugins:maven-help-plugin:2.1.1:evaluate -Dexpression=project.version | grep -v '\[')
cd ../wildfly
sed -i "s#<version.org.wildfly.core>.*#<version.org.wildfly.core>$WFLY_CORE_VERSION</version.org.wildfly.core>#" pom.xml
mvn clean install -DskipTests -Dcheckstyle.skip -Denforcer.skip

cp -r dist/target/wildfly-*-SNAPSHOT/ /tmp/wildfly-ladybird

cd /tmp/wildfly-ladybird
```
### Add management users

```bash
bin/add-user.sh -u monitor -p password1! -s
bin/add-user.sh -u administrator -p password1! -s
```

### Run WildFly with Elytron profile

```bash
bin/standalone.sh -c standalone-elytron.xml
```

### Update version.org.wildfly.core in the project
If necessary, update the WildFly Core version in the demo project too.

```bash
sed -i "s#<version.org.wildfly.core>.*#<version.org.wildfly.core>$WFLY_CORE_VERSION</version.org.wildfly.core>#" pom.xml
```


## Run demo

### Default run
The default WildFly host to which this client connects is `127.0.0.1` 

```bash
mvn package exec:java
```

The first demo should print `$local` username:
```
"identity" => {"username" => "$local"},
```
Default configuration doesn't contain any user/password specification.

The second demo should print `monitor` username:
```
"identity" => {"username" => "monitor"},
```
As the default host is `127.0.0.1` and not the `localhost`, we see here the `monitor` identity.

### Run with hostname specified
By setting system property `hostname` you can set to which host the controller client will connect: 

```bash
mvn package exec:java -Dhostname=localhost
```
The first part of the demo should still report `$local`user, but the second part should print `administrator` user:
```
"identity" => {"username" => "administrator"},
```

### Run with custom wildfly-config XML
By setting system property `wildfly.config.url` you can control from which location is the default `AuthenticationContext` configuration loaded.

```bash
mvn package exec:java -Dwildfly.config.url=https://rawgit.com/jboss-security-qe/elytron-client-demo/master/custom-config.xml
```
The first part of the demo should now report the same user as the second one:
```
"identity" => {"username" => "monitor"},
```

## Play with code

For instance, you can try to use `AuthenticationContext.captureCurrent()` instead of `AuthenticationContext.EMPTY`, which should take current context as a base instead of
building one from scratch.