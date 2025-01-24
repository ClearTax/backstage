              K8s New Service Onboarding Document

This documents helps you to onboard your application and deploy it to the kubernetes cluster.

If it's a new application then do reach out to \#help-infra-devops-zurg over slack. We will create,  
Team,policies for accessing passwords from vault.  
Namespace and serviceaccounts in K8s.  
IAM role creation for the new namespace

Steps that should be followed to create a new deployment in kubernetes.

﻿Making your application docker compatible.   
Visit links to get an idea on docker file \- https://outline.internal.cleartax.co/doc/dockerfile-standardisation-vsd6EOTdqp ,  
https://outline.internal.cleartax.co/doc/base-java-docker-image-f2R37wNp8K

Writing application configs, jmx exporter files etc as part of build process.  
Create a infra folder in the root directory of your Github repository. Refer this repo link https://github.com/ClearTax/e-invoicing-be/tree/main/infra .

Monitoring  
JVM metrics for Java applications https://outline.internal.cleartax.co/doc/jvm-metrics-for-java-applications-IP6WGP0SCq  
APM Agent standardisation .

We are doing the build process from Github itself through Github actions.  For your java applications to setup GH actions follow document \=\>\> https://outline.internal.cleartax.co/doc/git-branching-strategy-for-backend-applications-E60MUXrTXa  and also for your python-nodejs projects follow link  https://outline.internal.cleartax.co/doc/ci-with-github-actions-for-python-and-nodejs-projects-X6xwTNZ7Os .  
While writing your pom.xml file for your java application dont forget to add dependencies for sts and sqs. Or else after deployment IAM, SQS etc wont work.  
\<\!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-sts \--\>  
\<dependency\>  
    \<groupId\>com.amazonaws\</groupId\>  
    \<artifactId\>aws-java-sdk-sts\</artifactId\>  
    \<version\>1.12.620\</version\>  
\</dependency\>

\<\!-- https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-sqs \--\>  
\<dependency\>  
    \<groupId\>com.amazonaws\</groupId\>  
    \<artifactId\>aws-java-sdk-sqs\</artifactId\>  
    \<version\>1.12.621\</version\>  
\</dependency\>

app\_config files for deployment config. Every environment and deployment will have different app\_config.Check repo for more references.  
Make sure that your app\_config folder name should match the app\_info section.  
  "app\_info": {  
    "name": "example",  
    "env": "dev",  
    "team": "infra",  
    "stack": "backend",  
    "details": "http"  
folder name should be in the pattern as name-env-details.

Creating a repo in ECR for storing and deploying your Docker images.   
Use this Jenkins job for creating your ECR.

We are using Vault for storing your configs and secrets.Authentication of vault is done via namespace and service accounts. So, please make sure that you are creating secrets in the right folder and creating pods in the right namespace.  
You need to add the vault path in your app \_ config.json file, for example it looks like below example   
"vault\_secrets": {  
    "source": "infra/userservice/dev/configs infra/userservice/dev/secrets",  
    "destination": "/etc/default/ct-user-service"

Created your K8s deployment jenkins job using this job. 

Map your Domain endpoints  
For mapping your domain endpoints with traefik endpoints use this jenkins job  
Below are the traefik endpoints for mapping your domains based on the use case whether its internal endpoints or public facing domains.  
traefik-staging-internal.internal.cleartax.co  
AWS Mumbai non-prod internal domain  
traefik-staging-internet.internal.cleartax.co  
AWS Mumbai non-prod internet domain  
traefik-prod-internal.internal.cleartax.co  
AWS Mumbai prod internal domain  
traefik-staging-internet.internal.cleartax.co  
AWS Mumbai prod internet domain  
traefik-prod-oci-http.internal.cleartax.co  
OCI prod internal domain  
traefik-staging-oci-internal.internal.cleartax.co  
OCI non-prod internal domain  
traefik-staging-oci-internet.internal.cleartax.co  
OCI non-prod internet domain  
traefik-prod-oci-http.internet.cleartax.co  
OCI prod internet domain

IAM policies to access resources  
For policies with s3,sqs etc to be attached to IAM role, you need to create PR in ct-infra repo and share in \#help-infra-devops-zurg , devopsoncall will review it.  
Deployment/ build failures basic troubleshooting steps  
Post deployment if your facing heap memory issues for example logs show like below then

Defaulted container "businesshierarchy-dev-consumer" out of: businesshierarchy-dev-consumer, node2pod (init)  
Starting application /home/cleartax/application.jar with arguments \-server \-Xmx \-Xms \-XX:MaxRAMPercentage=90.0 \-XX:MaxGCPauseMillis=100 \-XX:+CrashOnOutOfMemoryError \-Xlog:gc\* \-verbose:gc \-javaagent:/etc/jmx-exporter/jmx\_prometheus\_javaagent-0.13.0.jar=8085:/etc/jmx-exporter/jmx\_exporter\_config.yml \-Dcom.sun.management.jmxremote.port=5000 \-Dcom.sun.management.jmxremote.rmi.port=5000 \-Dcom.sun.management.jmxremote.authenticate=false \-Dcom.sun.management.jmxremote.ssl=false \-XX:+HeapDumpOnOutOfMemoryError \-XX:HeapDumpPath=/dump/businesshierarchy-dev-consumer-fbcf985c-hdssl.hprof    
Invalid maximum heap size: \-Xmx  
Error: Could not create the Java Virtual Machine.  
Error: A fatal exception has occurred. Program will exit.  
then check your application jvm.conf and also your vault values.For example   
"MAX\_JVM\_HEAP\_SIZE": "1024m",  
  "MIN\_JVM\_HEAP\_SIZE": "750m",  
Minimum memory you might need to increase. Ideally heap size should be atleast 75% min memory for this to work.  
If your deployment in jenkins is failing with below error, please make sure that you are providing the version tag correctly. If there is a space in between, this can also cause below error. Do a cross-check at the params provided.

java.lang.NullPointerException: Cannot invoke method getAt() on null object  
    at org.codehaus.groovy.runtime.NullObject.invokeMethod(NullObject.java:91)  
    at org.codehaus.groovy.runtime.callsite.PogoMetaClassSite.call(PogoMetaClassSite.java:48)  
    at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)

During github actions run, if you encountered with below error, please cross check your pom.xml file for any missing dependencies .  
Error:  Failed to execute goal on project notice-management: Could not resolve dependencies for project in.cleartax.noticemanagement:notice-management:pom:0.146.1-SNAPSHOT: Could not find artifact junit:junit:jar:5.8.2 in github (https://maven.pkg.github.com/ClearTax/\*) 

In your local run if you are facing dependencies issues  
Could not transfer artifact in.clear.invoicing:client-invoice:pom:1.101.1-SNAPSHOT from/to github (https://maven.pkg.github.com/ClearTax/\*): authorization failed for https://maven.pkg.github.com/ClearTax/\*/in/clear/invoicing/client-invoice/1.101.1-SNAPSHOT/client-invoice-1.101.1-SNAPSHOT.pom, status: 403 Forbidden  
Then follow below steps:  
Create a PAT on GitHub with all scope privileges  
Copy the token  
Configure SSO for that PAT  
Add a settings.xml file in .m2 folder  
It should have the same contents as that of the settings.xml file in .github folder in project structure  
Along with that, add the following:  
\<servers\>  
\<server\>  
      \<id\>github\</id\>  
      \<username\>{github-username}\</username\>  
  \<password\>{PAT}\</password\>  
\</server\>  
\</servers\>  
Save the file and then run: mvn clean install

After your deployment if pods are going to crashloopbackoff with node2pod errors, then please check your vault values for unsupported characters or wrong entries.  
Defaulted container "einvoicingbe-prod-consumerlarge" out of: einvoicingbe-prod-consumerlarge, node2pod (init)  
/shared/secrets: line 109: export: \`PUBLIC\_KEY\_FILE2:=einv\_production.pem': not a valid identifier

Job to delete a single pod in dev env https://jenkins.internal.cleartax.co/job/Delete-k8s-dev-env-pods/.

Dashboards for checking

https://kubernetes-dashboard-staging.internal.cleartax.co/  
https://kubernetes-dashboard-prod.internal.cleartax.co/  
https://kubernetes-dashboard-ap-prod.internal.cleartax.co/  
https://kubernetes-dashboard-ap-sandbox.internal.cleartax.co/  
https://kubernetes-dashboard-infra.internal.cleartax.co/  
https://kubernetes-dashboard-oci-prod.internal.cleartax.co/  
https://kubernetes-dashboard-oci-staging.internal.cleartax.co/

Resource Costs checker

https://kube-resource-report-ap-prod.internal.cleartax.co/  
https://kube-resource-report-ap-sandbox.internal.cleartax.co/  
https://kube-resource-report-staging.internal.cleartax.co/  
https://kube-resource-report-prod.internal.cleartax.co/

Grafana dashboard link : https://grafana-prod.internal.cleartax.co/  
