# XCopy install ArangoDB on Windows

## Problem
While there is a nice guided installer for windows users, not all users prefer this means of installation. In order to have a [Portable application](http://en.wikipedia.org/wiki/Portable_application) [XCOPY deployment](http://en.wikipedia.org/wiki/XCOPY_deployment) is neccessary.

## Solution
With ArangoDB Version 2.5.1 it doesn't rely on registry entries anymore, so we can deploy using a ZIP-file.


## Steps

### Unzip archive
Open explorer, choose a place where you want arangodb to live in, unzip files there. It will create its own toplevel directory with the Version-number in the string.

### Alter configuration
*(optional)* Edit etc\arangodb\arangodb.conf if the default values don't suit your needs (i.e. the location of the database files, ports to bind, etc.)

### Create Runtime directories
Arangod leans on the existence of some directories in the **var** subdirectory, create them:

    C:\Program Files\ArangoDB-2.5.1>mkdir var\lib\arangodb
    C:\Program Files\ArangoDB-2.5.1>mkdir var\lib\arangodb-apps

### Run arangod
To start the database, simply run it like that:

    C:\Program Files\ArangoDB-2.5.1>bin\arangod

Now it takes a while to open all its databases, load system facilities, bootstrap the javascript environments, etc.; once its ready it prints:

    INFO ArangoDB (version 2.5.1 [windows]) is ready for business. Have fun!

You can now open the administrative webinterface in your browser using [this URL](http://127.0.0.1:8529/).

### Installing as service
If you don't want to run arangod from a cmd - window each time, installing it as a system service is the right thing to do.
This requires administrative privileges; you need to *Run as Administrator* the *cmd*-shell.
First we need to grant the SYSTEM-user access to our database directory, since arangod is going to be running as that user:

    C:\Program Files\ArangoDB-2.5.1>icacls var /grant SYSTEM:F /t

Next we can install the service itself:

    C:\Program Files\ArangoDB-2.5.1>bin\arangod --install-service

You now will have a new entry in the **Services** dialog labeled *ArangoDB - the multi-purpose database*; you can start it there, or just do it on the commandline using:

    C:\Program Files\ArangoDB-2.5.1>NET START ArangoDB

It will take a similar amount of time as your start from the comandline above till the service is up and running.
Since now you don't have any console to inspect the startup, messages of the serverety FATAL & ERROR are also output into the windows eventlog, so in case of failure you can have a look at the **Eventlog** in the **Managementconsole**

