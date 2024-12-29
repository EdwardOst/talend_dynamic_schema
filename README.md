# talend_dynamic_schema
Utilities for using Talend Dynamic Schema

## Installing Talend Dynamic Schema jar and source

The Talend dynamic schema libraries can be found in `<studio>\configuration\.m2\repository\org\talend\system-routines\8.0.1`
in `system-routines-8.0.1.jar`.  The source code can be found in the same directory in the `system-routines-8.0.1-sources.jar`

There are multiple classes included in the jar other than just dynamic schema classes.

To use these libraries in this code project you can add both the binary and source jars to the local .m2 repository.  Note the version as well as the packaging option is different for each file.

````
mvn install:install-file -DgroupId=org.talend -DartifactId=system-routines -Dversion=8.0.1 -Dpackaging=jar -Dfile=<studio>\configuration\.m2\repository\org\talend\system-routines\8.0.1\system-routines-8.0.1.jar

mvn install:install-file -DgroupId=org.talend -DartifactId=system-routines -Dversion=8.0.1 -Dpackaging=src -Dfile=<studio>\configuration\.m2\repository\org\talend\system-routines\8.0.1\system-routines-8.0.1-sources.jar 
````

Having installed the jar to the local .m2 repo you could now try to reference it is a regular dependency in this pom, but there are other dependencies.

Assuming (possibly incorrectly) that we are only interested in the dynamic schema classes and that dynamic schema behavior is contained entirely within this one jar we ignore those transitive dependencies.

We could exclude those transitive dependencies as shown below:

````
		<dependency>
			<groupId>org.talend</groupId>
			<artifactId>system-routines</artifactId>
			<version>8.0.1</version>
			<exclusions>
				<exclusion>
					<groupId>*</groupId>
					<artifactId>*</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
````

But a simpler way that expresses the intent is to set the scope as _provided_.  This ensures that the classpath is only added for the compilation and test phases.  The provided scope is not transitive, so no problems arise due to other Studio specific jars.

````
		<dependency>
			<groupId>org.talend</groupId>
			<artifactId>system-routines</artifactId>
			<version>8.0.1</version>
			<scope>provided</scope>
		</dependency>
````

However, neither of these worked.  When trying to build it returned the error

````
Failed to execute goal on project Could not collect dependencies for project com.talend.se.dynamic_schema:util:jar:0.0.1
Failed to read artifact descriptor for org.talend:system-routines:jar:8.0.1
Caused by: The following artifacts could not be resolved: org.talend.studio:tcommon-studio-se:pom:8.0.2-SNAPSHOT (absent): Could not transfer artifact org.talend.studio:tcommon-studio-se:pom:8.0.2-SNAPSHOT from/to talend_open_snapshots (https://artifacts-oss.talend.com/nexus/content/repositories/TalendOpenSourceSnapshot/): status code: 401, reason phrase: Unauthorized (401)
````

Opening the jar of the the sysltem-routines and looking in the META-INF/maven/org.talend/system-routines folder we can see the pom used to build the system-routines.jar.

In there we see that parent pom was org.talend.studio : tcommon-studio-se

