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

### Logging

The default location for logs is `$PWD/../logs/`, which is relative to where Tomcat was started from. When running Tomcat as a service, that won't do.

NSESA uses [log4j](https://logging.apache.org/log4j/) through [slf4j](http://www.slf4j.org/). We can configure more advanced logging by providing our own `production-log4j.xml`.

### `<at4am/e-parliament>/production-log4j.xml`

Example configuration. You might want to use [`SyslogAppender`](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/net/SyslogAppender.html) ([example](https://wiki.apache.org/logging-log4j/syslog)) or even [`SMTPAppender`](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/net/SMTPAppender.html) ([example](https://wiki.apache.org/logging-log4j/SMTPAppender)).

**NOTE:** [`org.apache.log4j.rolling.RollingFileAppender`](https://logging.apache.org/log4j/extras/apidocs/org/apache/log4j/rolling/RollingFileAppender.html) from [`log4j-extras`](https://logging.apache.org/log4j/extras/) requires NSESA to use `log4j-1.2.17.jar`. Remove this comment when that has been implemented.


```bash
# Edit location!
sudo mkdir /var/log/nsesa/

# Edit username/group to match your service user!
# Edit location!
sudo chown <tomcat-service-user> /var/log/nsesa/
sudo chgrp <tomcat-service-group> /var/log/nsesa/
```


```bash
# Download log4j extras to get the rolling file appender.
LOG4JVERSION="1.2.17"
wget "https://www.apache.org/dist/logging/log4j/extras/${LOG4JVERSION}/apache-log4j-extras-${LOG4JVERSION}-bin.tar.gz"

# Be smart and verify the package.
wget "https://www.apache.org/dist/logging/log4j/extras/${LOG4JVERSION}/apache-log4j-extras-${LOG4JVERSION}-bin.tar.gz.asc"
wget  --output-document "KEYS.asc" "http://www.apache.org/dist/logging/KEYS"
# You may or may not want to import all the Apache developers in KEYS.asc.
gpg --import "KEYS.asc"
gpg --verify "apache-log4j-extras-${LOG4JVERSION}-bin.tar.gz.asc"

# Unpack to "<at4am/e-parliament>/lib" then put it in the Tomcat CLASSPATH in setenv.sh: `CLASSPATH="<at4am/e-parliament>/lib"`.
# Could also extract to "$AT4AM_TOMCAT"/libexec/lib, which doesn't require setting CLASSPATH.
mkdir -p "<at4am/e-parliament>/lib"
tar -zxvf "apache-log4j-extras-${LOG4JVERSION}-bin.tar.gz" --strip-components=1 -C "lib/" "apache-log4j-extras-${LOG4JVERSION}/apache-log4j-extras-${LOG4JVERSION}.jar"
```


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<log4j:configuration>
    <appender name="nsesa-rollingfile" class="org.apache.log4j.rolling.RollingFileAppender">
        <!-- Edit location! -->
        <param name="file" value="/var/log/nsesa/nsesa.log" />
        <param name="append" value="true" />
        <param name="maxBackupIndex" value="90"/>
        <param name="encoding" value="UTF-8" />

        <rollingPolicy class="org.apache.log4j.rolling.TimeBasedRollingPolicy">
            <!-- Edit location! -->
            <param name="fileNamePattern" value="/var/log/nsesa/nsesa.%d.log.gz" />
        </rollingPolicy>

        <layout class="org.apache.log4j.PatternLayout">
            <param name="conversionPattern" value="%d{ISO8601} %p [%t] %c{2}: %m%n"/>
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
