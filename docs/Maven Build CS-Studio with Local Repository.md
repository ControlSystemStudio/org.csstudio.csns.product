# Maven Build CS-studio with Local Repository #


## 1.	Introduction ##

This document is a hand-in-hand mannual of building CS-Studio by using maven local repository.Originated from [CS-Studio Online Docbook-Chapter 4,Compiling, Running, Debugging CSS](http://cs-studio.sourceforge.net/docbook/ch04.html#idp140240).

**Note:** All the following works are done on Windows. If you are working on other OS, change the commands based on the same steps.

## 2. Prerequisite ##
Make sure you have the following installed on your computer.

- **Jdk 1.8** , set up the following environment variables :


>    JAVA_HOME = C:\Program Files\Java\jdk1.8.0_25
> 
>    CLASSPATH = .%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar
> 
>    PATH = %JAVA_HOME%\bin;$PATH


- **Apache-maven-3.2.3** , assumed location is D:/apache-maven-3.2.3 , set up the following environment variables :

> M2_HOME = D:\apache-maven-3.2.3
> 
> M2 = %M2_HOME%\bin
> 
> PATH = %M2%\bin;$PATH

- **Eclipse Luna 4.4.1**, assumed location is D:/Eclipse

- **Eclipse Tycho/Maven plugin**

## 3. Obtaining CS-Studio Source Codes ##
Make a git repository and clone the source codes:

    mkdir F:\newCSStudio
    cd F:\newCSStudio
    git clone https://github.com/ControlSystemStudio/maven-osgi-bundles.git
    git clone https://github.com/ControlSystemStudio/cs-studio-thirdparty.git
    git clone https://github.com/ControlSystemStudio/cs-studio.git
    git clone https://github.com/ControlSystemStudio/org.csstudio.product.git

Then select one of the products from github:

    git clone https://github.com/ControlSystemStudio/org.csstudio.sns.git

Or

    git clone https://org.csstudio.nsls2.product.git

Or make your site-specific products(Not included in this documents yet).


## 4. Configuring maven ##

First , download the composite p2 repository **'p2repo'** here:
https://github.com/ksdj/org.csstudio.csns.product/tree/master/docs

Make a Local Repository:
    
    cd F:\newCSStudio
    mkdir repo

Change D:/apache-maven-3.2.3/conf/settings.xml contents to the following one:

    <!-- Maven settings.xml -->
    <settings>
      <profiles>
    	<profile>
      		<id>my-css-settings</id>
      		<properties>
    
    			<csstudio.composite.repo>F:\newCSStudio\p2repo</csstudio.composite.repo>
    			<csstudio.local.repo>F:\newCSStudio\repo</csstudio.local.repo>
    			<cs-studio>dummy_value</cs-studio>
    
    			<!-- Issue warnings instead of aborting on error -->
    			<baselineMode>warn</baselineMode>
    			<jgit.dirtyWorkingTree>warning</jgit.dirtyWorkingTree>
    
    			<!-- Skip unit tests -->
    			<maven.test.skip>true</maven.test.skip>
    			<skipTests>true</skipTests>
    
      		</properties>
    	</profile>
      </profiles>
    
      <!-- Enable the above settings -->
      <activeProfiles>
    		<activeProfile>my-css-settings</activeProfile>
      </activeProfiles>
    </settings>


## 5.  Configuring Eclipse##

Run Eclipse.

### 5.1 Configure target platform: ###

- **Windows - Preferences - Plugin-in Development Target Platform**

Ticking **'Running Platform'**,

- **Add - Current Target : Copy settings from the Current Platform**

- **Add - Directory** 

Browse and select 
**F:\newCSStudio\repo**

### 5.2 Configure Tycho/maven ###

- **Windows - Preferences - Maven - Installations - Add**

Then select D:/apache-maven-3.2.3 as your built-in maven.

- Select **Plugin execution not covered by lifecycle configuration** as
**'Ignore'**​

### 5.3 Configure Default JRE ###

- **Windows - Preferences - Java - Installed JREs - Add - Standard VM** Select C:\Program Files\Java\jdk1.8.0_25, then tick it to be the default platform JRE.


## 6.  Maven Build CS-Studio ##

All the following repository should be built successully.
    
    cd F:\newCSStudio\maven-osgi-bundles
    mvn clean verify
    
    cd F:\newCSStudio\cs-studio-thirdparty
    mvn clean verify
    
    cd F:\newCSStudio\cs-studio\core
    mvn clean verify
    
    cd F:\newCSStudio\cs-studio\applications
    mvn clean verify
    
    cd F:\newCSStudio\org.csstudio.product
    mvn clean verify

	cd F:\newCSStudio\org.csstudio.csns.product
	mvn clean verify

After all the mvn building finished, you can see the result in the local repository **F:\newCSStudio\repo**

## 7.  Import Into Eclipse ##
Run Eclipse then **"File - Import - Maven - Exisiting Maven Projects"** ,then **Browse** to import the sources in the following order:

F:\newCSStudio\cs-studio\core
F:\newCSStudio\cs-studio\applications
F:\newCSStudio\org.csstudio.product
F:\newCSStudio\org.csstudio.sns.product

## 8.  Fix Errors ###

With the local reopsitory target platform, all the required dependencies plugins should be built up in F:\newCSStudio\repo. All plugins can find their dependencies there. However, there may be some other errors.

One type of errors is JDK compatibility. For example:

plugin **org.csstudio.archive.writer.rdb**

Right Click the plugin, then **"Build Path - Configure Build Path... - Libraries"**. Select the **J2SE 1.6 - Remove - Add Library - JRE System Library - Next - Workspace default JRE(jdk1.8.0_25)** 

After trying eliminating those errors above. If there are still errors in the following plugins, ignore and close the project:

1.**org.csstudio.diag.probe**
> This one is no longer used, ignore it for org.csstudio.diag.pvmanager.probe

2.**org.csstudio.jre6.fragment** and **org.csstudio.rap.ui.util**
> The *rap* plugins won't work in the IDE configured for RCP, because they are, well, RAP plugins. The jre6 fragment is also for RAP.


## 9.  Running Products ###

Run Eclipse.Select **sns-repository -> sns-css.product** .Double click to open the product file. Click 'Launch an Eclipse application'.

Then cs-studio will have its first run. There may be some errors like missing required bundle, cannot revoled dependencies. If so , terminate the running csstudio. Right click the product file. **"Run As - Run Configuration"** Open **"Eclipse Application - sns-css.product"** Select tab **Plug-ins** .There is a list showing. It includes all the plugin you will use when you run the product.

Search for **'rap'** and untick all of the plugins listed out. Then push the button **'Validate Plug-ins'**, watch carefully each error, add the required plugin one by one. At last all of the errors will be eliminated.

If you want to use some other plugin, such as 'scan', tick all the scan relevant plugins.

At last, run the product again.
