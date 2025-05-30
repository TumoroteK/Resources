# Missing Dependencies

When cloning or forking the open-source project Tumorotek, building the project will fail due to missing dependencies. Even running `mvn compile` will result in failure for the same reason.

Let's examine some of those dependencies and their reasons:

## OJDBC
The OJDBC is marked as provided, indicating that the dependency is expected to be provided by the runtime environment or another module, and it will not be packaged with the application. When you declare a dependency with `<scope>provided</scope>`, it means that you expect the necessary JAR files for that dependency to be provided by the environment where your application will be running. The version in use is ojdbc6; currently (as of 2024), ojdbc10 is available.

## ZK
The ZK dependencies in this project are of version 6.5.7, a version that cannot be found in Maven Repo, probably because it was removed. A better solution is to update the ZK version to one that can be found in the Maven repository.

# Downloading Dependencies
You can download the JARs from the following links or use the JARs in this repository. They are equivalent:

Download:
- [OJDBC6](https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc6/11.2.0.4/ojdbc6-11.2.0.4.jar)
- [xws-security-1.3.1](https://repository.jboss.org/com/sun/xml/wsit/xws-security/1.3.1/xws-security-1.3.1.jar)
- [wsit-rt-1.1](https://repository.jboss.org/com/sun/xml/wsit/wsit-rt/1.1/wsit-rt-1.1.jar)

## Installing Dependencies
Move the JARs to a folder in your OS (pay special attention that none of the folders in the path has spaces in the folder name). In the project's folder, run the following commands:

```shell
mvn install:install-file -Dfile=path/to/your/ojdbc6-11.2.0.4.jar -DgroupId=oracle -DartifactId=ojdbc6 -Dversion=11.2.0.4 -Dpackaging=jar
mvn install:install-file -Dfile=path/to/your/xws-security-1.3.1.jar -DgroupId=com.sun.xml.wsit -DartifactId=xws-security -Dversion=1.3.1 -Dpackaging=jar   
mvn install:install-file -Dfile=path/to/your/wsit-rt-1.1.jar -DgroupId=com.sun.xml.wsit -DartifactId=wsit-rt -Dversion=1.1 -Dpackaging=jar 
```

## Verifing 
Navigate to the follwing folders and verify that the jar was correctly installed. Expect a *.jar file around 2MB in size
...\.m2\repository\com\sun\xml\wsit\xws-security\1.3.1
...\.m2\repository\com\sun\xml\wsit\wsit-rt\1.1
...\.m2\repository\oracle\ojdbc6\11.2.0.4
# ZK Dependencies
Here, the process is different. 
1. You should copy the folders inside the `zk.zip` to the `.../.m2/repository/org/zkoss` directory.
Do it first. If it doesn't work and you still missing dependencies...
1. Unpack zkoss.zip into `.../.m2/repository/org`
# Force Update Maven Dependencies and Clean Install Project
run
`mvn clean install -U
`

# Git Action
When using the following YAML configuration:

```yaml
name: Compile TK On Linux

on:
  push:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Set up environment variable
        run: echo "ZK_FOLDER=/home/runner/.m2/repository/org/zkoss" >> $GITHUB_ENV

      - name: Checkout code
        uses: actions/checkout@v2

      # Set up Java 8
      - name: Set up JDK 8
        uses: actions/setup-java@v2
        with:
          java-version: '8'
          distribution: 'temurin'

     # Create Maven Wrapper
      - name: Create Maven Wrapper
        run: mvn wrapper:wrapper

      # Restore Maven cache
      - name: Restore Maven cache
        uses: skjolber/maven-cache-github-action@v1
        with:
           step: restore
 
     # Install Missing JAR: Oracle JDBC
      - name: Install Oracle JDBC driver to local repository
        run: |
         wget -O ojdbc6-11.2.0.4.jar https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc6/11.2.0.4/ojdbc6-11.2.0.4.jar
         ./mvnw install:install-file -Dfile=ojdbc6-11.2.0.4.jar -DgroupId=oracle -DartifactId=ojdbc6 -Dversion=11.2.0.4 -Dpackaging=jar
         
     # Install Missing JAR: A dependency of Spring WS Security » 2.1.3.RELEASE
      - name: Download and install xws-security-1.3.1.jar
        run: |
          wget  -O com.sun.xml.wsit.xws-security-1.3.1.jar    https://repository.jboss.org/com/sun/xml/wsit/xws-security/1.3.1/xws-security-1.3.1.jar
          ./mvnw install:install-file -Dfile=com.sun.xml.wsit.xws-security-1.3.1.jar -DgroupId=com.sun.xml.wsit -DartifactId=xws-security -Dversion=1.3.1 -Dpackaging=jar   

      # Install Missing JAR: wsit-rt
      - name: Install wsit-rt to local repository
        run: |
          wget -O wsit-rt-1.1.jar https://repository.jboss.org/com/sun/xml/wsit/wsit-rt/1.1/wsit-rt-1.1.jar
          ./mvnw install:install-file -Dfile=wsit-rt-1.1.jar -DgroupId=com.sun.xml.wsit -DartifactId=wsit-rt -Dversion=1.1 -Dpackaging=jar 
          

      #The ZK dependencies in this project are of version 6.5.7: a version that can't be found in Maven Repo
      - name: Extract zk files to destination folder
        run: |
            if [ ! -d $ZK_FOLDER ]; then
            echo "ZK folder does not already exist. Extracting files..."        
            mkdir -p $ZK_FOLDER
              unzip -q $GITHUB_WORKSPACE/.github/workflows/zk.zip -d $ZK_FOLDER
            else
              echo "ZK folder already exists. Skipping extraction."
            fi


      # Build the project using Maven Wrapper
      - name: Compile with Maven Wrapper
        run: ./mvnw clean compile -DskipTests


      # Save Maven Cache
      - name: Save Maven cache
        uses: skjolber/maven-cache-github-action@v1
        with:
         step: save
```

Make sure to adjust paths and dependencies as necessary.
