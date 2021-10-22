
# Splunk Observability - Getting started

This repo started as a fork of the classic Spring Pet Clinic repo and it was used during a presentation about Splunk Observability.

The goal is to walk through the basic steps to configure the following components of the Splunk Observability platform:

1. Splunk Infrastructure Monitoring (IM)
2. Splunk Application Performance Monitoring (APM)
3. Splunk Real User Monitoring (RUM)
4. Splunk LogsObserver (LO)

We will also show the steps about how to clone (download) a sample java application (Spring PetClinic), as well as how to compile, package and run the application. 

Once the application is up and running, we will instrument the application using OpenTelemetry Java Instrumentation libraries that will generate traces and metrics used by the Splunk APM product.

After that, we will instrument the PetClinic's end user interface (html pages rendered by the application) with the Splunk Open Telemetry Javascript Libraries that will generate traces about all the individual clicks and page loads executed by the end user.

Lastly, we will configure the Spring PetClinic application to write application logs to the filesystem and also configure the Splunk Open Telemetry Collector to read (tail) the logs and report to the Splunk Observability Platform.

Here's a diagram of the final state of this exercise:

![This is the final state of the exercise, with multiple Splunk Observability Components configured](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/exercise.png?raw=true)

## Pre-Requisites (Or Pre-Work)

### 1. Environment (VM)

The exercise and instructions below were created using an Ubuntu VM (18.04), but any compatible Ubuntu (debian) distro should work.

If you are planning to run this exercise locally, in a lab format, we recommend Multipass ([https://multipass.run/](https://multipass.run/)) or VirtualBox ([https://www.virtualbox.org/](https://www.virtualbox.org/)). While we do not cover VM creation here, there are plenty of resources available with detailed instructions on a number of websites. A VM with 2GB, 1vCPU and 15gb HD should be able to handle it. 

The VM needs to have access to the the internet for both downloading installers as well as sending the telemetry to the Splunk endpoints

### 2. Splunk Observability Cloud Account

Another Pre-requisitite is access to the Splunk Observability Cloud. You should check with your team members and account admins in order to get credentials. In case you don't have an account and want to run a trial, you can create your account here: [https://www.splunk.com/en_us/observability/o11y-cloud-free-trial.html](https://www.splunk.com/en_us/observability/o11y-cloud-free-trial.html)

### 3. Basic Console (Shell) knowledge
Lastly, this exercise requires basic knowledge and familiarity with using a shell console, running commands and editing configuration files (we are use vi here, you can use whatever editor you prefer). If you never tried that out, we recommend reading a bit around linux shell and commands/editors.


## Getting Started

### VM Login
First step is to log in to your VM. In the steps here, we will use the commands for multipass.

    multipass start my-o11y-vm
    multipass shell my-o11y-vm

Or you can ssh to the VM using your terminal app 

    ssh ubuntu@VM-IP

Or ff using VirtualBox with Desktop environment, just start the VM and open the Terminal app


### VM Prepare

We will now run a few commands to download required components:

    sudo apt update
    sudo apt install curl git maven openjdk-11-jdk


It might take a few minutes depending on your VM specs and network speed. The commands above will install components necessary for the exercise

### First Login to Splunk Observability Cloud
Meanwhile, you can go ahead and login to your Splunk Observability Account.

 - https://app.signalfx.com (us0 realm) 
 - https://app.us1.signalfx.com (us1 realm) 
 - https://app.us2.signalfx.com (us2 realm)
 - https://app.eu0.signalfx.com (eu0 realm)

![enter image description here](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/loginpage.png?raw=true)

If you are not sure where your account is/was set, please contact your administrator and/or check your email for a login link.

After login, you should land in a page like this:

![enter image description here](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/o11y-landingpage.png?raw=true)


## Exercise
### Splunk Infrastructure Monitoring (IM)
Let's get started with step #1: **Install the OpenTelemetry Collector**. 
The OpenTelemetry Collector is a key component responsible for
- Collecting and Reporting IM metrics (disk, cpu, memory, etc)
- Receiving and Reporting APM Traces
- Collecting and Reporting host and application logs

Splunk Observability offers wizards to walk you through the setup of the agents and instrumentation. To get to the wizard, click in the top left corner icon (the hamburger menu), then click on Data Setup

![enter image description here](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/o11y-landingpage-hamburguer.png?raw=true)


![enter image description here](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/side-menu-data-setup.png?raw=true)

Select Data Setup > Linux > Add Connection

You'll be taken to a short wizard where you will select some options. The default settings should work, no need to make changes. The wizard will output a few commands that need to be executed in the shell. 

Here's an example: 

    curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
    sudo sh /tmp/splunk-otel-collector.sh --realm us1 -- <API TOKEN REDACTED> --mode agent

*(Please do not copy and paste this command during your exercise as it will not work. You should copy the command from your Splunk Observability Wizard page. The command above has the API TOKEN REDACTED and we need the real API TOKEN associated with your account)*

This command will download and setup the OpenTelemetry Collector. Once the install is completed, you can navigate to the Infrastructure page to see the data from your Host

![Hamburger Menu](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/hamburguer.png?raw=true) (Hamburguer Menyu) >> Infrastructure >> My Data Center >> Hosts

Add Filter >> host.name >> (type or select your hostname)

Once you see data flowing for your host, we are then ready to get started with the APM component

--------------------------------------------------
### Splunk Application Performance Monitoring (APM)

#### Download and Build the Spring PetClinic App
First thing we need to setup APM is... well, an application. For this exercise, we will use the Spring Pet Clinic application. This is a very popular sample java application built with Spring framework (Springboot).

We will now clone the application repository, then we will compile, build, package and test the application.

    git clone https://github.com/spring-projects/spring-petclinic

(eventually -> https://github.com/asomensari-splunk/spring-petclinic)

then we open the directory

    cd spring-petclinic

and run the maven command to compile/build/package

    ./mvnw package -Dmaven.test.skip=true

(this might take a few minutes the first time you run, maven will download a lot of dependencies before it actually compiles the app. Future executions will be a lot shorter)

Once the compilation is complete, you can run the application with the following command:

    java -jar target/spring-petclinic-*.jar

You can validate if the application is running by visiting 

    http://<VM_IP_ADDRESS>:8080 

(feel free to navigate and click around )

#### Instrument the Application With Splunk OpenTelemetry Java Libraries
Now that the application is running, it is time to setup the APM instrumentation. The Splunk APM product uses Open Telemetry libraries to instrument the applications ([https://github.com/signalfx/splunk-otel-java](https://github.com/signalfx/splunk-otel-java)). 
The Otel-Java library will instrument code to generate metrics and spans/traces that are reported to the OpenTelemetry Collector. 

Let's continue the process by visiting the Splunk Observability Cloud UI again. 

![Hamburguer Menu](https://github.com/asomensari-splunk/spring-petclinic/blob/main/src/main/resources/static/resources/images/hamburguer.png?raw=true) (Hamburguer Menu) >> Data Setup

Then

APM Instrumentation >> Java >> Add Connection

The APM Instrumentation Wizard will show a few options for you to select, things like application name, environment, etc. In this scenario we are using:
- Application Name: petclinic
- Environment: conf21

At the end of the wizard, you'll be given a set of commands to run (similar to the Splunk IM instructions)

*(make sure you are in the spring-petclinic directory)*

    curl -L https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent-all.jar -o splunk-otel-javaagent.jar

*(this command downloads the Splunk Open Telemetry Java Instrumentation library)*

    export OTEL_SERVICE_NAME='petclinic'
    export OTEL_RESOURCE_ATTRIBUTES='deployment.environment=conf21,version=0.314'
    export OTEL_EXPORTER_OTLP_ENDPOINT='http://localhost:4317'

*(these commands define settings required by the instrumentation library)*

Lastly, we will run our application adding the -javaagent tag in front of the command

    java  -javaagent:./splunk-otel-javaagent.jar -jar target/spring-petclinic-*-SNAPSHOT.jar

Let's go visit our application again to generate some traffic. 

    http://<VM_IP_ADDRESS>:8080 

*(click around, generate errors, add visits, etc )*

Then you can visit the APM UI and examine the application components, traces, etc

Hamburguer Menu >> APM >> Explore

--------------------------------------------------
### Splunk Real User Monitoring (RUM)

For the Real User instrumentation, we will add the Open Telemetry Javascript ([https://github.com/signalfx/splunk-otel-js-web](https://github.com/signalfx/splunk-otel-js-web)) snippet in the pages. We will use the wizard again.

Data Setup >> RUM Instrumentation >> Browser Instrumentation >> Add Connection

Then you'll need to select the RUM token and define the application and environment names. The wizard will then show a snipped of HTML code that needs to be place at the top at the pages (preferably in the < HEAD > section). In this example we are using:
- Application Name: petclinic
- Environment: conf21

``

    <script src="https://cdn.signalfx.com/o11y-gdi-rum/latest/splunk-otel-web.js" crossorigin="anonymous"></script>
    <script>
    SplunkRum.init({
        beaconUrl: "https://rum-ingest.us1.signalfx.com/v1/rum",
        rumAuth: "XXXXXXXXXXXXXXXXXXXX",
        app: "petclinic",
        environment: "conf21"
        }); </script>

The Spring PetClinic application uses a single html page as the "layout" page that is reused across all pages of the application. This is the perfect location to insert the Splunk RUM Instrumentation Library as it will be loaded in all pages automatically.

Let's then edit the layout page:

    vim src/main/resources/templates/fragments/layout.html

and let's insert the snipped we generated above in the < HEAD > section of the page.

Now we need to rebuild the application and run it again:

    ./mvnw package -Dmaven.test.skip=true
    java  -javaagent:./splunk-otel-javaagent.jar -jar target/spring-petclinic-*-SNAPSHOT.jar

Then let's visit the application again to generate more traffic, now we should see RUM traces being reported.

    http://<VM_IP_ADDRESS>:8080 

(feel free to navigate and click around )

Let's visit RUM and see some of the traces and metrics.

Hamburger Menu >> RUM

You should see some of the Spring PetClinic urls showing up in the UI

--------------------------------------------------
### Splunk Log Observer (LO)
For the Splunk Log Observer component, we will configure the Spring PetClinic application to write logs to a file in the filesystem and configure the Splunk OpenTelemetry Collect to read (tail) that log file and report the information to the Splunk Observability Platform.

#### Splunk Open Telemetry Collector (FluentD)  Configuration
We need to configure the Splunk OpenTelemetry Collector to tail the Spring Pet Clinic log file and report the data to the Splunk Observability Cloud endpoint.

The Splunk OpenTelemetry Collector uses FluentD to consume/report logs and to configure the proper setting to report Spring PetClinic logs, we just need to add a FluentD configuration file in the default directory (/etc/otel/collector/fluentd/conf.d/).

Here's the sample FluentD configuration file (petclinic.conf, reading the file /tmp/spring-petclinic.log)

    <source>
      @type tail
      @label @SPLUNK
      tag petclinic.app
      path /tmp/spring-petclinic.log
      pos_file /tmp/spring-petclinic.pos_file
      read_from_head false
     <parse>
       @type none
     </parse>
    </source>

So we need to create the file

    sudo vim /etc/otel/collector/fluentd/conf.d/petclinic.conf

And paste the contents from the snippet above. Once the file is created, we need to restart the FluentD process

    sudo systemctl restart td-agent

#### Spring Pet Clinic Logback Setting
The Spring PetClinic application can be configure to use a number of different java logging libraries. In this scenario, we are using logback. Here's a sample logback configuration file:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE xml>
    <configuration scan="true" scanPeriod="30 seconds">
      <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
         <resetJUL>true</resetJUL>
      </contextListener>
      <logger name="org.springframework.samples.petclinic" level="debug"/>
      <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/tmp/spring-petclinic.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
          <fileNamePattern>springLogFile.%d{yyyy-MM-dd}.log</fileNamePattern>
          <maxHistory>5</maxHistory>
          <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
          <pattern>
          %d{yyyy-MM-dd HH:mm:ss} - %logger{36} - %msg trace_id=%X{trace_id} span_id=%X{span_id} trace_flags=%X{trace_flags} service.name=%property{otel.resource.service.name}, deployment.environment=%property{otel.resource.deployment.environment} %n
          </pattern>
        </encoder>
      </appender>
      <root level="debug">
        <appender-ref ref="file" />
      </root>
    </configuration>

We just need to create a file named logback.xml in the configuration folder. 

    vim src/main/resources/logback.xml

and paste the XML content from the snippet above. After that, we need to rebuild the application and run it again:

    ./mvnw package -Dmaven.test.skip=true
    java  -javaagent:./splunk-otel-javaagent.jar -jar target/spring-petclinic-*-SNAPSHOT.jar


Then let's visit the application again to generate more traffic, now we should see log messages being reported.

    http://<VM_IP_ADDRESS>:8080 
(feel free to navigate and click around )

Then visit:
Hamburger Menu > Log Observer

And you can add a filter to select only log messages from your host and the Spring PetClinic Application:
Add Filter > Fields > host.name > your host name

Add Filter > Fields > service.name > your application name

## Summary 
This the end of the exercise and we certainly covered a lot of ground. At this point you should have metrics, traces and logs being reported into your Splunk Observability account. 


