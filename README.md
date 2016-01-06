<!--
* DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS HEADER.
*
* Copyright 2015-2017 ForgeRock AS.
*
* The contents of this file are subject to the terms
* of the Common Development and Distribution License
* (the License). You may not use this file except in
* compliance with the License.
*
* You can obtain a copy of the License at
* http://forgerock.org/license/CDDLv1.0.html
* See the License for the specific language governing
* permission and limitations under the License.
*
* When distributing Covered Code, include this CDDL
* Header Notice in each file and include the License file
* at http://forgerock.org/license/CDDLv1.0.html
* If applicable, add the following below the CDDL Header,
* with the fields enclosed by brackets [] replaced by
* your own identifying information:
* "Portions Copyrighted [year] [name of copyright owner]"
*
-->
# openam-auth-rsa-axm
**A Custom Authentication Module For [ForgeRock's AM and OpenAM][forgerock_am] in a RSA ClearTrust/Access Manager Environment**

[The ForgeRock Identity Platform][forgerock_platform] was developed to integrate with any of your digital services. More than legacy customer identity management, we designed the platform for the needs of IoT. With ForgeRock, you get the feeling it was all built to work together, because it was.
Securely connect people, devices, and things, so everyone and everything can interact in today’s IoT world. With billions of devices coming online every year, you need a flexible platform that scales.

This AM Authentication Module enables SSO from ClearTrust to OpenAM.




## What's in the Repository?

The repository contains branches corresponding to the version of OpenAM/AM that you're working with. At the time of writing this includes the following branches:

OpenAM/AM version  	| Repository Branch
-------------------	|------------------
5.5.1					| releases/5.5.1
5.1.1					| releases/5.1.1
5.0.0					| releases/5.1.1
OpenAM 13.5.1			| releases/13.5.1
OpenAM 13.0.0			| releases/13.0.0
OpenAM 12.0.4			| releases/12.0.4

Check out the branch for the version of AM/OpenAM that you're working with before building and deploying.

Each branch contains following components:

    1. src/main/resources - contains files for registeration of the custom auth module
    2. src - This directory contains source for custom auth module.
    3. common.cfg, compile.sh and test.sh - scripts for compiling and testing the auth
       module. Edit common.cfg for your enviorment before compiling and testing.

**Note:** The ClearTrust SDK jar files are not included in this repository due to licensing restrictions.

# Building The Code

The code in this repository has binary dependencies that live in the ForgeRock maven repository. Maven can be configured to authenticate to this repository by following the following [ForgeRock Knowledge Base Article](https://backstage.forgerock.com/knowledge/kb/article/a74096897)

This authentication module has a dependency on the RSA axm-runtime-api library. The best way to build this code is to install your axm-runtime-api jar file to your maven repsository. The following shows how to deploy the 6.2.2 version of the axm-runtime-api into your local maven repository for testing on your local machine;

```
$ mvn install::install-file -Dfile=$RSA_AXM_DIR/sdk/java/runtime/lib/axm-runtime-api-$RSA_AXM_VER.jar \
-DgroupId=rsa.axm -DartifactId=axm-runtime-api -Dversion=$RSA_AXM_VER -Dpackaging=jar
```

Where `$RSA_AXM_DIR` is an environment variable set to the directory in which your RSA Access Manager SDK may be found and `$VERSION` is an environment variable set to the version of the AXM runtime api jar file.

### Using Different Versions of the RSA AxM Libraries

The maven POM file for this project declares a dependency on version 6.2.2 of the `axm-runtime-api` jar file. If you are using a different but compatible version, install the jar file in the same was as described above. You will then need to change the `rsa.axm.version` property in this project's pom file to specify the version of the RSA AxM library you have installed, e.g.

```
<properties>
  <rsa.axm.version>6.2.1</rsa.axm.version>
</properties>
```
Will make this auth module use the 6.2.1 version of the axm-runtime-api jar.

Once set up you can build the project using maven;

```
$ mvn clean install
```


## Pre-requisites :


   1. RSA ClearTrust server X.X or higer version installed and configured.
   2. RSA ClearTrust Runtime SDK X.X or higher version installed and configured.
   3. The openam.war from AM/OpenAM Distribution Kit

## Required SSO integration components:


   1. An installed version of AM or OpenAM
   2. A web container preferably Tomcat 8.x.
   3. Custom authentication module which is compiled from the branch of this repo compatible with your AM/OpenAM version.


## OpenAM Installation and Configuration:

  1. Create a temporary directory ($WAR_DIR) /var/tmp/amwar and explode 
     the openam.war into it as follows
     $ cd /var/tmp/amwar
     $ jar xvf openam.war

  2. Copy target/openam-auth-rsa-axm-X.X.X.jar to $WAR_DIR/WEB-INF/lib

  3. Copy resources/CTAuthService.properties to $WAR_DIR/WEB-INF/classes

  4. Copy resources/CTAuthService.xml to $WAR_DIR/config/auth/default and
     also to the directory $WAR_DIR/config/auth/default_en

  5. Re-war openam.war 
     $ jar cvf ../openam.war .

  6. Deploy openam.war onto your container and restart it

  7. Access OpenAM by pointing your browser to
      http://host:port/openam

  8. You will now see the OpenAM Configuration screen.  Choose "Custom 
     Configuration" and complete the wizard.  Refer to the OpenAM 
     Installation Guide for futher details.

  9. After successful configuration you will be redirected to the 
     OpenAM Admin Console's Login page. 


## Auth module configuration:

Now we have to load the RSA ClearTrust authentication module service into
OpenAM and configure it. The auth module service is loaded from a OpenAM 
command line utility called as "ssoadm". In OpenAM the ssoadm utility is 
also exposed as ssoadm.jsp.

If you want to use the commandline then refer to the following URL and skip
step 1-5
   https://backstage.forgerock.com/#!/docs/openam/12.0.0/dev-guide#chap-auth-spi

Here we will use use browser based ssoadm.jsp for OpenAM configuration
changes.

  1. Login into OpenAM using amadmin and enable ssoadm.jsp as follows
     https://wikis.forgerock.org/confluence/display/openam/Activate+ssoadm.jsp

  2. Now access the following URL provided that you have
     http://host:port/openam/ssoadm.jsp

  3. Choose "create-svc" option.

  4. Copy and paste the xml file from resources/CTAuthService.xml and Submit
     This will define the auth module service into OpenAM configuration.

  5. Now register the auth module into the authentication core framework.

     http://host:port/openam/ssoadm.jsp
     Choose "register-auth-module option".
     Enter "org.forgerock.openam.authentication.modules.cleartrust.CTAuth" as the
     auth module class name.


  6. Now verify that the auth module is registered to the default realm.
     http://host:port/openam, click on default realm, and click on
     "Authentication", click "New", you should see "RSA ClearTrust" in the
     list of modules.

  7. Click on "RSA ClearTrust" and create a new module instance called "CTAuth"

  8. Once the instance is created go back to Authentication and click on "CTAuth"
     to configure it. Fill in the form and save.

  9.  Restart openam by restarting the container it is running in


## Testing:


The testing of the module assumes that ClearTrust SDK is already
installed and configured. Please check the ClearTrust documentation
for ClearTrust SDK installation.

CAUTION: All the runtime associated jar files should be copied to
OpenAM WEB-INF/lib directory or in the classpath of OpenAM.  The
path is usually sdk/java/runtime/lib.


1. Now access the ClearTrust protected application and login with
   ClearTrust configured user to establish CTSESSION. The configuration
   of ClearTrust policy and authentication schemes are outside scope of this
   documentation and please check ClearTrust documentation for more
   information.

2. After successful authentication at ClearTrust server, access the OpenAM
   auth module url as follows:

   http://host:port/openam/XUI/#login/&module=CTAuth or if using Legacy UI
   http://host:port/openam/UI/Login?module=CTAuth

   This should Single Sign you into OpenAM AND provide a valid OpenAM session.

   Note: Assumption here is that ClearTrust and OpenAM are in the same
         cookie domain.

   By default OpenAM authentication framework looks for user profile existance
   in it's known data repositories. However, you could use ignoreProfile
   option if your integration does not require a user to be searched from
   ClearTrust's user repository. Check the OpenAM documentation for more info
   about ignoreProfile option.

3. You can also use the test.sh curl wrapper shell script to test the module.


## Disclaimer

This software is provided 'as is' without warranty of any kind, either express or implied, including, but not limited to, the implied warranties of fitness for a purpose, or the warranty of non-infringement. Without limiting the foregoing, ForgeRock makes no warranty that:

i. the software will meet your requirements  
ii. the software will be uninterrupted, timely, secure or error-free iii. the results that may be obtained from the use of the software will be effective, accurate or reliable  
iv. the quality of the software will meet your expectations  
v. any errors in the software obtained from ForgeRock servers web site will be corrected.  

This software and its documentation:  
vi. could include technical or other mistakes, inaccuracies or typographical errors. The BGS may make changes to the software or documentation made available on its web site.  
vii. may be out of date, and ForgeRock makes no commitment to update such materials. ForgeRock assumes no responsibility for errors or ommissions in the software or documentation available from its web site.  

In no event shall ForgeRock be liable to you or any third parties for any special, punitive, incidental, indirect or consequential damages of any kind, or any damages whatsoever, including, without limitation, those resulting from loss of use, data or profits, whether or not the BGS has been advised of the possibility of such damages, and on any theory of liability, arising out of or in connection with the use of this software.  

The use of this software is done at your own discretion and risk and with agreement that you will be solely responsible for any damage to your computer system or loss of data that results from such activities. No advice or information, whether oral or written, obtained by you from ForgeRock or from the ForgeRock web site shall create any warranty for the software.

[forgerock_platform]: https://www.forgerock.com/platform/
[forgerock_am]: https://www.forgerock.com/platform/access-management/
