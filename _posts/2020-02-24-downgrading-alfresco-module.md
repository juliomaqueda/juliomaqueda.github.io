---
layout: post
title: Downgrading an Alfresco 5 module
description: Guide on how to downgrade an Alfresco module without dying in the process
date:  2020-02-24
time: 10 mins
tags: alfresco tomcat java
language: english
---

Years ago, Alfresco used to store modules' versions directly in the database as plain text. As you can imagine, hacking versions and performing downgrades with such weak configuration was really easy stuff.

```sql
select node_id, qname_id, string_value from alf_node_properties where node_id = 3976;

+---------+----------+----------------------------------+
| node_id | qname_id | string_value                     |
+---------+----------+----------------------------------+
|    3976 |       85 | com.example.custom-module-name   |
|    3976 |      718 | 2.1.0                            |
+---------+----------+----------------------------------+
```

However, a series of security improvements in Alfresco 5 conflated, among others changes, in the way modules' information was stored. Instead of using the `string_value` column (table `alf_node_properties`), they started serializing a specific java object `org.alfresco.repo.module.ModuleVersionNumber`, inserting it in the column `serializable_value` of the same table.

<!-- more -->

You can explore this behaviour directly in the [Alfresco source code](https://github.com/Alfresco/alfresco-repository/blob/ac38ac94ff4f9cbdf2671a9517781bda389a13c4/src/main/java/org/alfresco/repo/module/ModuleVersionNumber.java) or by reviewing Alfresco's database:

```sql
select id from alf_qname where local_name = 'currentVersion';

+-----+
| id  |
+-----+
| 269 |
+-----+

select string_value, serializable_value from alf_node_properties where qname_id=269 limit 1;

+--------------+-----------------------------------------------------------+
| string_value | serializable_value                                        |
+--------------+-----------------------------------------------------------+
| NULL         | �� sr ,org.alfresco.repo.module.ModuleVersionNumberwG=��� |
+--------------+-----------------------------------------------------------+
```

## Getting to a solution

There is no official solution to downgrade an Alfresco module's version under this new paradigm, having all workarounds pros and cons. If you try to reach the Alfresco Support Team, they'll just say the operation is not supported.

The solution we'll be appling in this article is based on altering the application startup process, forcing Alfresco to override the module's version, regardless of the existing value. Doing it this way, we won't need to tune the module's configuration, relying on Alfresco mechanisms to store and load the module properly.

As a pre-step, this solution will require the Alfresco Tomcat server to [run in debug mode](/2020/01/17/configure-tomcat-debug-mode.html).

### Step 1: Double check the affected module

Before starting breaking things, we need to corroborate the affected module name and version. The server's log files are a good input to retrieve that information:

```
Caused by: org.alfresco.error.AlfrescoRuntimeException: 01240016
Downgrading of modules is not supported.
Module 'custom-module-name' version 2.1.0 is currently installed and must be uninstalled before version 2.0.0 can be installed.
```

As we can see, the affected module is `custom-module-name`, having 2.1.0 as the currently installed version, and 2.0.0 as the version we're trying to install.

### Step 2: Setting up breakpoints

Open your Alfresco CMS project on your preferred IDE and set a conditional breakpoint in the following line:

```java
// Class: org.alfresco.repo.module.ModuleComponentHelper
// Method: startModule

ModuleVersionNumber currentModuleVersion = this.getModuleVersionNumber(moduleCurrentVersion);
* if (moduleInstallVersion == null) {
	logger.warn(I18NUtil.getMessage("module.warn.no_install_version", new Object[]{moduleId, moduleCurrentVersion}));
	this.registryService.addProperty(moduleKeyInstalledVersion, currentModuleVersion);
}
```

Making the java process pause under the following condition:

```java
moduleId.equals("custom-module-name")
```

If you don't have the whole Alfresco source code don't worry, you just need a local java project with the `org.alfresco.alfresco-repository-X.Y.Z.jar` dependency (you can extract it from the Alfresco war file or its deployment folder).

### Step 3: Startup the Alfresco server in debug mode

Startup the Alfresco server in [debug mode](/2020/01/17/configure-tomcat-debug-mode.html) and connect your IDE to the debug port.

### Step 4: Set the new module's version

When the startup process reaches the modules' activation phase, you should see it stopping at your breakpoint, allowing you to hack the process by evaluating the following expression:

```java
currentModuleVersion = new ModuleVersionNumber("2.0.0")
```

Once the code evaluation finishes (it may take some time) you can move forward with the startup process.

That's it! your module's version got downgraded.

## Other solutions

There are other alternatives you may want to consider if this solution turns out being too intrusive for your infrastructure (we all know starting a production server in debug mode is not always possible).

### Alfresco patch

You could create an Alfresco patch to set the specific module version, making use of Alfresco's module helpers. On the plus side, you ensure the change is only applied once. However, you'll need to take care of that residual patch at some point. You'd also need to create a specific product release to execute the module downgrade.

### Administration webscript

Same concept but creating an Alfresco administration webscript instead of a patch. You'd need to evaluate the risks of having this kind of webscripts available in a production deployment. A new product release would be needed as well.

### Local integration test

Developers could connect a local integration test (or utility class) to the affected database and deserialize/update the stored ModuleVersionNumber object. Not for the faint of heart.

## Final considerations

Hope your Alfresco installation got to this point well and safe. There are a couple of important considerations to take into account.

### Alfresco cluster

Note the module configuration is directly persisted in the Alfresco database, meaning you'll only need to execute it once to have the changes available in all members of the cluser.

### Bootstrap processes

This workaround only changes the version of a concrete module in the Alfresco database to allow you install an older version. Any piece of code executed as part of the previous module's bootstrap **is not undone**.
