# [![AT4AM.eu](https://at4am.eu/resource/image/logo/at4ameu-32x32.jpg)](https://at4am.eu/) Running the [e-parliament AT4AM (NSESA) project](https://github.com/e-parliament) on production systems

Steps for fetching/compiling/installing/configuring/running/testing the [e-parliament project of AT4AM](https://github.com/e-parliament) on production systems. *You don't need to do this to develop or run AT4AM on your own computer!*


## Preparations

**Assumptions**

- You are familiar with [running the e-parliament AT4AM (NSESA) project](https://github.com/at4ameu/at4am-documentation/blob/master/docs/running-e-parliament-at4am.md), as this is a follow-up.
- You are experienced enough to see where this tutorial fails horribly, and are willing to contribute your improvements.
- The path `<at4am/e-parliament>` is where you previously put the source code.



## Steps

### Environment

```bash
# This path to Tomcat might differ on your system.
# For me it expands to `/usr/local/opt/tomcat7`.
AT4AM_TOMCAT=$(brew --prefix tomcat7)

# Change directory to where you previously put the source code.
cd "<at4am/e-parliament>"
```



### Configuration files

Instead of overriding NSESA configuration in properties files in `$HOME`, a custom path can be used by setting an environment variable or java parameter. We already have `setenv.sh`, so it's convenient to use it for these overrides as well.

Instead of having the database setup in `~/nsesa-server.properties`, let's use `<at4am/e-parliament>/nsesa-server.properties`. Edit `setenv.sh` and add `-DNSESA_EDITOR_AN_PROPERTIES=file://<at4am/e-parliament>/nsesa-server.properties`. You can also set `NSESA_EDITOR_AN_PROPERTIES` to override `nsesa-editor-an.properties`.


```bash
# Move database setup properties.
mv ~/nsesa-server.properties "<at4am/e-parliament>/nsesa-server.properties"

# Set path to nsesa-server.properties.
"$EDITOR" "$AT4AM_TOMCAT"/libexec/bin/setenv.sh
```



### Logging

The default location for logs is `$PWD/../logs/`, which is relative to where Tomcat was started from. When running Tomcat as a service, that won't do.

NSESA uses [log4j](https://logging.apache.org/log4j/) through [slf4j](http://www.slf4j.org/). We can configure more advanced logging by providing our own `production-log4j.xml`.

### `<at4am/e-parliament>/production-log4j.xml`

Example configuration. You might want to use [`SyslogAppender`](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/net/SyslogAppender.html) ([example](https://wiki.apache.org/logging-log4j/syslog)) or even [`SMTPAppender`](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/net/SMTPAppender.html) ([example](https://wiki.apache.org/logging-log4j/SMTPAppender)).

**TODO:** Use [`org.apache.log4j.rolling.RollingFileAppender`](https://logging.apache.org/log4j/extras/apidocs/org/apache/log4j/rolling/RollingFileAppender.html) from [`log4j-extras`](https://logging.apache.org/log4j/extras/) when NSESA has upgraded to `log4j-1.2.17.jar`. Check for another (git) branch of this file for details.


```bash
# Edit location!
sudo mkdir /var/log/nsesa/

# Edit username/group to match your service user!
# Edit location!
sudo chown <tomcat-service-user> /var/log/nsesa/
sudo chgrp <tomcat-service-group> /var/log/nsesa/
```


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration>
    <appender name="nsesa-rollingfile" class="org.apache.log4j.RollingFileAppender">
        <!-- Edit location! -->
        <param name="file" value="/var/log/nsesa/nsesa.log" />
        <param name="MaxFileSize" value="10MB"/>
		<param name="MaxBackupIndex" value="10" />
        <param name="encoding" value="UTF-8" />

        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{ISO8601} %p [%t] %c{2}: %m%n"/>
        </layout>
    </appender>

    <root>
        <priority value="INFO" />
        <appender-ref ref="nsesa-rollingfile" />
    </root>
</log4j:configuration>
```

Edit your `setenv.sh` and add `-Dlog4j.configuration=file://<at4am/e-parliament>/production-log4j.xml` to `CATALINA_OPTS`. Optionally add `-Dlog4j.debug` to verify that the right log4j settings have been loaded.

```bash
"$EDITOR" "$AT4AM_TOMCAT"/libexec/bin/setenv.sh
```

Remember to restart Tomcat.



---

[![AT4AM.eu](https://at4am.eu/resource/image/logo/at4ameu-16x16.jpg)](https://at4am.eu/) [AT4AM.eu](https://at4am.eu/) &copy; 2013, 2014, 2015 [Föreningen för digitala fri- och rättigheter (DFRI)](https://dfri.se/). The documentation is released under the [Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/) license. Related resources and projects may have other licences.
