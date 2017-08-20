---
layout: post
title: Patch User Defined WildFly Modules in Domain Mode
tags: [wildfly,java,patch]
---

WildFly 8 added support for patching application server. This is an interesting feature and I wanted to play around a bit.
But unfortunately I could not find any official documentation explaining how to generate a patch.
With some effort, I could finally patch user defined static module. In the following blog post I would like to share the details.

## Custom Module

I created a simple static module that makes configurations (via property files) available to applications (WAR, EAR).
I had some confusion about the base directory for modules; should it be _modules/system/layers/base_ or _modules/system/add-ons_ or just _modules_.
But based on this https://developer.jboss.org/thread/222551?tstart=0[post]; static user defined modules are not layered distributions.
and shall be placed directly under modules/foo/bar folder just like they used to in 7.1.1.Final - that's the recommended way.

```bash
└── modules
    └── net
        └── arunoday
            └── configuration
                └── main
                    ├── arunoday.properties
                    └── module.xml
```


## How to generate a patch?

The _patch_ command needs the patch content to be available in a zip archive.
The patch content shall be organized in the following directory layout.
With this layout, module is directly installed in modules directory.
The custom modules shall be organized as misc files. Create a zip archive once the
patch content is ready.

[source,bash]
----
server:1.0 Aparna$ tree
.
├── META-INF
├── misc
├── patch-udm-configuration-1.0
│   └── misc
│       └── modules
│           └── net
│               └── arunoday
│                   └── configuration
│                       └── main
│                           ├── arunoday.properties
│                           └── module.xml
└── patch.xml


```


### Generate hash:

If you take a look at https://github.com/wildfly/wildfly-core/blob/master/patching/src/main/java/org/jboss/as/patching/HashUtils.java#L46-53[HashUtils.java],
you can see that WildFly patching module uses SHA1 hash.

```java
    private static final MessageDigest DIGEST;
    static {
        try {
            DIGEST = MessageDigest.getInstance("SHA-1");
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }
```

Let's generate SHA1 hash for the files that belong to the static module.

```bash
server:main Aparna$ ls
arunoday.properties	module.xml

# generate hash for configuration file
server:main Aparna$ cat arunoday.properties | shasum
409ca4766fed3dc2370b9c5fa591cfc2a7fcf9de  -


# generate hash for module.xml
server:main Aparna$ cat module.xml | shasum
97adffffd2bdd28b20358070a509e54b6b9c9fbf  -

```


### Prepare patch.xml:

Now that we have generated the hash, next step is define _patch.xml_ file.

```xml

<?xml version="1.0" ?>

<!-- Note the patch identifier, same as directory name -->
<patch xmlns="urn:jboss:patch:1.0" id="patch-udm-configuration-1.0">
    <description>
        This patch adds user defined configuration module to WildFly installation
    </description>
    <no-upgrade name="WildFly" version="9.0.0.Alpha1"/>

    <!-- Custom defined modules shall be installed as misc files -->
    <misc-files>
      <!-- since we are installing the module for the first time, use added element. -->
      <added path="modules/net/arunoday/configuration/main/arunoday.properties" hash="409ca4766fed3dc2370b9c5fa591cfc2a7fcf9de"/>
      <added path="modules/net/arunoday/configuration/main/module.xml" hash="97adffffd2bdd28b20358070a509e54b6b9c9fbf"/>
    </misc-files>

</patch>

```

## Patch Inspect:

Before installing a patch, we can see what's inside a patch using the inspect command.
Since WildFly is running in domain mode, make sure you provide the _host_ name.

```bash
patch --host=server.local inspect /Users/Aparna/Development/Projects/prototypes/wildfly-patch-custom-modules/1.0/patch-udm-1.0.zip
```

Example output:
```bash
[domain@172.16.123.1:9999 /] patch --host=server.local inspect /Users/Aparna/Development/Projects/prototypes/wildfly-patch-custom-modules/1.0/patch-udm-1.0.zip
Patch ID:         patch-udm-configuration-1.0
Type:             one-off
Identity name:    WildFly
Identity version: 9.0.0.Alpha1
Description:      This patch adds user defined configuration module to WildFly installation
[domain@172.16.123.1:9999 /]

```

## Install Patch:



```bash
patch --host=server.local apply /Users/Aparna/Development/Projects/prototypes/wildfly-patch-custom-modules/1.0/patch-udm-1.0.zip
```

Example output:

```bash
[domain@172.16.123.1:9999 /] patch --host=server.local apply /Users/Aparna/Development/Projects/prototypes/wildfly-patch-custom-modules/1.0/patch-udm-1.0.zip
{
    "outcome" : "success",
    "result" : null,
    "server-groups" : null,
    "response-headers" : {
        "operation-requires-restart" : true,
        "process-state" : "restart-required"
    }
}
[domain@172.16.123.1:9999 /]

```

## Restart Server:

```bash

/host=server.local:shutdown(restart=true)
```

Example output:
```bash

[domain@172.16.123.1:9999 /] /host=server.local:shutdown(restart=true)
{
    "outcome" => "success",
    "result" => undefined
}
```

## View Patch History:

```bash

[domain@172.16.123.1:9999 /] patch --host=server.local history
{
    "outcome" : "success",
    "result" : [{
        "patch-id" : "patch-udm-configuration-1.0",
        "type" : "one-off",
        "applied-at" : "11/16/14 10:58 AM"
    }],
    "server-groups" : null
}

```

## Verification

To verify if the patch is installed successfully; lets try to use the module in a sample application.
For this purpose, I created a sample application that reads the properties; converts them to JSON format and makes available through REST endpoint _/config-reader/rest/config_.
To deploy the application in domain mode following CLI command can be used.

```bash

deploy config-reader.war --server-groups=main-server-group
```

Now let's access the http://172.16.123.2:8080/config-reader/rest/config[web application] to see if the patch is installed and verify if new properties are visible.

### Installation directory

The patches installed are maintained in the $WILDFLY_HOME/.installation directory. One thing to notice is, in this directory layout, we cannot see our custom module configurations.
The reason for that is, when a patch is installed; miscellaneous files are staged directly under their target location; in our case under $WILDFLY_HOME/modules directory.

```bash

.
├── identity.conf
└── patches
    └── patch-udm-configuration-1.0
        ├── configuration
        │   ├── appclient
        │   │   └── appclient.xml
        │   ├── domain
        │   │   ├── domain.xml
        │   │   ├── host-master.xml
        │   │   ├── host-slave.xml
        │   │   └── host.xml
        │   └── standalone
        │       ├── standalone-full-ha.xml
        │       ├── standalone-full.xml
        │       ├── standalone-ha.xml
        │       └── standalone.xml
        ├── patch.xml
        ├── rollback.xml
        └── timestamp
```

A rollback file is generated based on the patch.xml.
Since our patch is quite straightforward (adds new files); rollback action is simply removing the installed files.

```xml

server:.installation Aparna$ more patches/patch-udm-configuration-1.0/rollback.xml
<?xml version='1.0' encoding='UTF-8'?>

<patch xmlns="urn:jboss:patch:rollback:1.0" id="patch-udm-configuration-1.0">
    <description>
        rollback patch
    </description>
    <no-upgrade name="WildFly Full" version="9.0.0.Alpha1"/>
    <misc-files>
        <removed path="modules/net/arunoday/configuration/main/arunoday.properties" hash="409ca4766fed3dc2370b9c5fa591cfc2a7fcf9de"/>
        <removed path="modules/net/arunoday/configuration/main/module.xml" hash="97adffffd2bdd28b20358070a509e54b6b9c9fbf"/>
    </misc-files>
    <installation>
        <identity name="WildFly Full" release-id="base"/>
        <layer name="base" release-id="base"/>
    </installation>
</patch>
```

## Update custom module

Now let's try to update our configuration module by changing some properties in _arunoday.properties_ file.
Follow similar steps for patch preparation as we did for the first patch.
Make sure that the patch identifier is unique and does not conflict with the existing patch-id.

```xml

<?xml version="1.0" ?>

<!-- Use unique patch identifier -->
<patch xmlns="urn:jboss:patch:1.0" id="patch-udm-configuration-1.1">
    <description>
        This patch updates user defined configuration module
    </description>
    <no-upgrade name="WildFly" version="9.0.0.Alpha1"/>

    <misc-files>
      <!-- Remove the existing properties file -->
      <removed path="modules/net/arunoday/configuration/main/arunoday.properties" hash="409ca4766fed3dc2370b9c5fa591cfc2a7fcf9de" />
      <!-- Add the updated one -->
      <added path="modules/net/arunoday/configuration/main/arunoday.properties" hash="075822e4d66e0ef4685de45f9258712ab1f23b0e" />
    </misc-files>

</patch>
```

After the patch is installed, let's look at the patch history.

```bash

[domain@172.16.123.1:9999 /] patch --host=server.local history
{
    "outcome" : "success",
    "result" : [
        {
            "patch-id" : "patch-udm-configuration-1.1",
            "type" : "one-off",
            "applied-at" : "11/16/14 11:47 AM"
        },
        {
            "patch-id" : "patch-udm-configuration-1.0",
            "type" : "one-off",
            "applied-at" : "11/16/14 10:58 AM"
        }
    ],
    "server-groups" : null
}
[domain@172.16.123.1:9999 /]

```

If we access the application now, we should see updated properties.


## Conclusion

In the above blog post I demonstrated how to install/update user defined module using new patching mechanism introduced in WildFly.
Since there is no official documentation about patch generation, it is unclear if this feature can be used for installation of user defined
configurations.

## Resources

* http://wildfly.org/news/2014/02/11/WildFly8-Final-Released/
* https://developer.jboss.org/wiki/SingleInstallationPatching/
* https://github.com/aparnachaudhary/prototypes/tree/master/wildfly-patch-custom-modules
