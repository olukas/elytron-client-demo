## Prerequisities

### Add management users

```bash
bin/add-user.sh -u monitor -p password1! -s
```

### Run WildFly with Elytron profile

```bash
bin/standalone.sh
```

## Run

The default WildFly host to which this client connects is `127.0.0.1` and set property `wildfly.config.url` to custom-config.xml in root of this project: 

```bash
mvn package exec:java -Dwildfly.config.url=PATH_TO/custom-config.xml
```

It prints
```
"identity" => {"username" => "monitor"},
```
even if GSSAPI SASL mechanism is set on client side.

