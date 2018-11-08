This document details the design to automate the deployment of Quantiply
Applications to customer on-premises Standalone systems.

The overall deployment is divided into 3 steps:

1.  Installation preparation: In this step, we create a Deployment user
    which will be used for all deployment , generate certificates and
    install basic utilities.

2.  Downloading/Generating Projects Artifacts which need to be deployed.

3.  Deploying the Artifacts to on-premises standalone system

** 1.1 Installation Preparation:**
-----------------------------

To start with, the user has to clone a git repo (amlops) which contains
all the scripts to automate the deployment . Besides that it also
contains pkgs folder which contains supporting files (e.g. for gradle,
it contains gradle.properties which contains details to connect to
Nexus. Also, for factbase, it contains supervisord.conf file). Next we
need to run a script which does following steps :

a)  User needs to create an input file or can just edit the existing
    file which details all the values that will be part of user input
    during deployment. The sample file will be like :

DEP\_user=

Domain\_name=

DEFAULT\_ORACLE\_SYS\_PASS=

FB\_DB\_USERNAME=

FB\_DB\_PASSWORD=

PREDICTS\_DB\_USERNAME=

PREDICTS\_DB\_PASSWORD=

KEYSTORE\_FILE\_PATH=

KEYSTORE\_PASSWORD=

KEY\_PASSWORD=

KEYSTORE\_FILE\_PATH=

TRUSTSTORE\_FILE\_PATH=

TRUSTSTORE\_PASSWORD=

SSL\_CRT\_FILE=

SSL\_KEY\_FILE=

ORIENTDB\_ROOT\_PASSWORD=

ORIENTDB\_JAVA\_OPTS=

POSTGRESQL\_VERSION=

PGUSER=

QUID\_DB\_USER=

QUID\_DB\_PASSWORD=

FACTBASE\_VERSION=

SPRING\_ACTIVE\_PROFILE=

Oracle\_server\_hostname=

Oracle\_server\_HostIP=

MDB\_server\_hostname=

MDB\_server\_HostIP=

Quantid\_server\_hostname=

Quantid\_server\_HostIP=

ES\_server\_HostIP=

ES\_server\_hostname=

Factbase\_hostname=

Factbase\_HostIP=

iPulse\_hostname=

iPulse\_HostIP

OrientDB\_hostname=

OrientDB\_HostIP=

FACTBASE\_VERSION=

SPRING\_ACTIVE\_PROFILE=

FB\_JAVA\_OPTS=

FACTBASE\_PROTO=

QUANT\_ID\_PEM=

QUANT\_TRUSTSTORE\_FILE=

QUANT\_ID\_CLIENT\_SECRET=

QUANT\_ID\_PULSE\_USER=

QUANT\_ID\_PULSE\_USER\_PWD=

\#\# For certificates

CA\_COUNTRY=US

CA\_STATE=SanJose

CA\_LOCALITY=SanJose

CA\_ORG=Quantiply

CA\_COMMON\_NAME=ca.quantiply.com

SUBJ\_COUNTRY=US

SUBJ\_STATE=SanJose

SUBJ\_LOCALITY=SanJose

SUBJ\_ORG=Quantiply

SUBJ\_COMMON\_NAME=\*.quantiply.com

b)  Create deployment user and provide sudo access to it,

c)  Copy dir structure of pkgs along with the supporting files to the
    home directory of deployment user. Just to make sure we donot use
    root or any of its folder.

d)  Copy env file for all the appls so that the env variables are set
    permanently even after reboot.

e)  Install utilities like wget, unzip etc which are needed to download
    artifacts from Nexus.

f)  Generate certificates based on the values mentioned in input file.

**Downloading Project Artifacts**
---------------------------------

Following are the options we can choose from:

### **Create tarball **

The tarball contains everything needed for the deployment(e.g scripts,
pkgs , jars, misc utilities and supporting files). Upload the tarball to
either nexus or to AWS S3.

*Note: tarball need not be a single .tar.gz file. Instead it can be a
collection of multiple .tar.gz file of each Application which are kept
in a folder/bucket.*

**[Procedure:]{.underline}**

1.  User 1 has to download the platform kits(e.g. jdk, maven, yum utils
    etc) either from nexus or from internet.

2.  For all the Appl kits(e.g. factbase or ipulse), clone the repo,
    build it using maven or gradle.

3.  All the kits from 1 and 2 has to be compressed using zip or tar.

4.  Create a folder in nexus or bucket in AWS . Enable versioning and
    put all the zipped files there.

5.  User 2 (deployment engineer) downloads the zipped files offline in
    any storage media and then starts the installation in
    customer-premises.

**[Pros: ]{.underline}**

1.  Standard practice which most tech companies follows for the
    deployment .

2.  Its is QA certified tarball which we are sure will work without any
    workarounds.

3.  No need for any internet connectivity while deployment and thus more
    secure.

4.  Its possible to enable versioning of the tarball and thus can decide
    which version to use for the deployment.

5.  Excellent choice for the first time deployment . For the subsequent
    installation, can just change the jars of the Appls (if possible)

**[Cons:]{.underline}**

1.  Size of tar ball is huge , hence storage of this tarball is a bit
    concern.

2.  If tarball is in Nexus server, then downloading it would may slow
    nexus server.

3.  Creating a tarball is complex and time-consuming. Thus providing a
    hot-fix will be challenging when time is critical.

4.  Not a good option if need to deploy more often.

### **Use jenkins**

Currently Jenkins is used for CI/CD tool. It is responsible for
compiling Appl code in github, building docker image, push it to nexus
and then deploy Docker containers with that image.

We need to include an additional step to push the Application artifacts
(like jar) to Nexus.

**[Procedure:]{.underline}**

1.  User runs Jenkins pipeline job manually or automatic with each
    commit which generates Application artifacts and is pushed to Nexus

2.  User then runs a script which downloads all the artifacts(Platform
    and Appl) from Nexus.

3.  User starts the deployment.

    Note: Need to change existing Jenkins file to push the artifacts to
    nexus.

**[Pros: ]{.underline}**

1.  No extra steps in the script to clone git repo, build the artifacts.
    Instead the user has to just download the artifact from nexus.

2.  Since all the artifacts are in one place (Nexus), downloading is
    much simpler and all the artifacts can be downloaded in a single
    session created with Nexus.

3.  Since no interaction with Git, the script needs lesser manual
    intervention to enter username and password.

**[Cons:]{.underline}**

1.  While pushing the artifacts to Nexus, it will be tagged . The user
    need to remember which tag to use while downloading the artifacts
    from nexus and which tag contains what features/fix.

2.  Multi-actors involved in this i.e. Jenkins, Nexus. Failure of any of
    this is a show-stopper and much time may need to be invested
    debugging this component .

3.  Size of some Application artifacts is extremely huge (e.g. for
    ipulse , its like 11GB). Thus pushing these artifacts to Nexus can
    gradually take most of the storage in Nexus

### **Clone repo from Github and build locally**

In this approach, Git Hub repo will be cloned and then using maven or
gradle, it will be build. Then the artifacts will be put in correct dir
structure and deployment can start.

**[Procedure:]{.underline}**

1.  User runs a script which downloads all the Platform kits from Nexus
    and then installs those Platform kits.

2.  Next user runs another script which downloads the github repo,
    builds it and then installs those Appls.

**[Pros: ]{.underline}**

1.  Very straightforward. Clones the github repo containing latest code.
    Builds it and then installs it. No need to push the artifacts
    anywhere or remember which jar contains what features/fix.

**[Cons:]{.underline}**

1.  Bit complex since we need to download the Platform kits from Nexus
    and Application repo from github.

2.  The scr code is exposed to the customer. We can tweak it in the
    script to delete all the src files and keep just jar files. But
    these becomes extremely complex and erroneous for some Appls e.g.
    ipulse which has multiple dirs and each dir has src dir and jar
    files.

**Deploying the artifacts to on-premises servers**
--------------------------------------------------

Once the project artifacts are in the correct directory, we can start
deploying the artifacts. The installation proceeds in the following
order. The below pkgs will be installed as "deployment user" except
oracle which uses "root" user.

a)  Install Base utils which includes installing supervisor , copy
    certificates, change system properties (required for postgres) etc.

b)  Install Platform pkgs like JDK, Maven, Gradle, Tomcat

c)  Install DB kits e.g. Oracle, OrientDB, Elastic search, Postgres

d)  Install Quantiply Application kits e.g. Factbase, iPulse, Quant-id.

### **Installing Platform pkgs:**

a)  To install Platform pkgs, We are using the pkgs that are already
    copied from nexus as mentioned in section 1.2 .

b)  After installing/configuring the pkgs, we are coping the supporting
    config files (which are already there in pkgs dir) to the respective
    location e.g. after installing Maven, need to copy settings.xml to
    .m2 dir.

### **Installing Oracle:**

a)  While installing oracle , we are installing oracle rpm file , then
    setting below env variables:

export ORACLE\_HOME=/u01/app/oracle/product/11.2.0/xe \\

export ORACLE\_SID=XE \\

export DEFAULT\_SYS\_PASS=oracle \\

export processes=500 \\

export sessions=555 \\

export transactions=610 \\

export FB\_DB\_USERNAME=factbasejava \\

export FB\_DB\_PASSWORD=password \\

export PREDICTS\_DB\_USERNAME=predicts \\

export PREDICTS\_DB\_PASSWORD=predicts \\

export PATH=\$ORACLE\_HOME/bin:\$PATH

b)  Then configuring the DB as root user(since any other use is not
    allowed to configure the DB)

c)  changing the HTTP port from 8080 to 8040 since port 8080 is used by
    factbase.

### **Installing Elastic Search:**

a)  Check if JDK is installed.

b)  Set below env variables:

> export ELASTIC\_CONTAINER true
>
> export PATH /usr/share/elasticsearch/bin:\$PATH

c)  Create a elasticsearch user

d)  Install Elastic Search

e)  Copy supporting files to respective dir.

f)  Login as "elasticsearch" user and start ES process

### **Installing OrientDB:**

a)  Set below env variables:

> export ORIENTDB\_HOME=/home/\<dep user\>/orientdb
>
> export
> KEYSTORE\_FILE\_PATH=/etc/secrets/tls-certs/orientdb/server/keystore.jks
>
> export KEYSTORE\_PASSWORD=quantiply
>
> export KEY\_PASSWORD=quantiply
>
> export TRUSTSTORE\_FILE\_PATH=/etc/secrets/tls-ca/truststore.jks
>
> export TRUSTSTORE\_PASSWORD=quantiply
>
> export DB\_NAMES=factbase,
>
> export ORIENTDB\_ROOT\_PASSWORD=quantiply
>
> export JAVA\_OPTS=\"-Xms1g -Xmx2g\"
>
> export LOG\_DIR=\$ORIENTDB\_HOME/log

b)  Install OrientDB

c)  Copy supporting files to respective dir.

d)  Create Database for factbase

e)  Start OrientDB as dep user.

### **Installing Postgres:**

a)  Copy supporting files to respective dir.

b)  Install Postgres

c)  Login as postgres user and set below env variables:

> export POSTGRESQL\_VERSION=9.5
>
> export PGUSER=postgres
>
> export QUSER=quantiply
>
> export
> CONTAINER\_SCRIPTS\_PATH=/usr/share/container-scripts/postgresql
>
> export ENABLED\_COLLECTIONS=rh-postgresql95
>
> export QUID\_DB\_USER=quid
>
> export FB\_DB\_PASSWORD=factbase
>
> export QUID\_DB\_PASSWORD=quid
>
> export QUID\_DB\_NAME=quid
>
> export FB\_DB\_NAME=factbase
>
> export FB\_DB\_USER=factbase
>
> export HOSTNAME=mdb.quantiply.com
>
> export PGDATA=/var/lib/pgsql/data/userdata/
>
> export
> SSL\_CRT\_FILE=/usr/share/container-scripts/postgresql/server.crt
>
> export
> SSL\_KEY\_FILE=/usr/share/container-scripts/postgresql/server.key

d)  Start Postgres

### **Installing Factbase:**

a)  Login as dep\_user and export the following variables:

> export FACTBASE\_HOME=/home/\<dep-user\>/factbase
>
> export FACTBASE\_VERSION=3.5.4
>
> export SPRING\_ACTIVE\_PROFILE=mas
>
> export JAVA\_OPTS=\'-Xms1g -Xmx2g\'

b)  Copy factbase jar file to factbase\_home dir

c)  Edit supervisord.conf file to replace env variables with correct
    values.

d)  Start supervisord process which will start factbase Application.

### **Installing IPULSE:**

a)  Login as dep\_user and export the following variables:

export IPULSE\_HOME==/home/\<dep-user\>/IPULSE

export FACTBASE\_PORT=8080

export FACTBASE\_PROTO=http

export FACTBASE\_HOST=fb.quantiply.com

export QUANT\_ID\_PEM=/tmp/uaa.pub

export QUANT\_TRUSTSTORE\_FILE=/tmp/truststore.jks

export QUANT\_ID\_CLIENT=qtz-cli

export QUANT\_ID\_CLIENT\_SECRET=quantiply123

export QUANT\_ID\_PULSE\_USER=superuser\@quantiply.com

export QUANT\_ID\_PULSE\_USER\_PWD=quantiply

export QUANT\_ID\_HOST=qid.quantiply.com

export QUANT\_ID\_PORT=8443

b)  Copy all the iPulse jar to IPULSE\_HOME

c)  Start iPulse Appl

### **Installing QuantID:**

a)  Login as dep\_user and create a tomcat home dir

b)  Copy all the files from pkgs dir to tomcat home dir

c)  Export the following variables:

> export QUSER=quantiply
>
> export QUID\_HOME=/home/quantiply/quid
>
> export SUPER\_USER\_NAME=superuser
>
> export TOMCAT\_HOME=/home/quantiply/tomcat
>
> export DB\_HOST=mdb.quantiply.com
>
> export HOSTNAME=quid.quantiply.com
>
> export DB\_NAME=quid
>
> export DB\_PORT=5432
>
> export LOGOUT\_REDIRECT=https://:
>
> export QUANT\_ID\_CLIENT=qtz-cli
>
> export
> SERVER\_KEYSTORE=/etc/secrets/tls-certs/quant-id/server/keystore.jks
>
> export QUANT\_ID\_PULSE\_USER=pulse
>
> export JAVA\_OPTS=-Xmx2g
>
> export
> LOGIN\_CONFIG\_URL=file:\$QUID\_HOME/required\_configuration.yml
>
> export QUANT\_ID\_SIGNING\_KEY=/etc/secrets/rsa-keys/uaa.key
>
> export VERSION\_FILE=\$QUID\_HOME/version.txt
>
> export REDIR\_SCHEME=https
>
> export QUANT\_ID\_CLIENT\_SECRET=quantiply123
>
> export SECRETS=/etc/secrets
>
> export JAVA\_HOME=/opt/java/
>
> export DB\_PASSWORD=quid
>
> export DOMAIN=quantiply.com
>
> export QUANT\_ID\_PREDICTS\_USER=predicts
>
> export QUANT\_ID\_SIGNING\_CRT=/etc/secrets/rsa-keys/uaa.pub
>
> export DB\_USERNAME=quid
>
> export QTZ\_HOST=:
>
> export SUPER\_USER\_PWD=quantiply
>
> export QUANT\_ID\_PREDICTS\_USER\_PWD=quantiply
>
> export QUANT\_ID\_CLIENT\_REDICT\_URL=n/a
>
> export SERVER\_KEYSTORE\_PASSWORD=quantiply
>
> export QUANT\_ID\_PULSE\_USER\_PWD=quantiply
>
> export SERVER\_TRUSTSTORE=/etc/secrets/tls-ca/truststore.jks
>
> export UAA\_CONFIG\_PATH=/home/quantiply/quid/config
>
> export HOST=0.0.0.0
>
> export PORT=8443
>
> export PROTO=https
>
> export SECURE\_MODE=true

d)  Create QUID\_HOME

e)  Copy QUID jar files to QUID\_HOME

f)  Run QUID Appl
