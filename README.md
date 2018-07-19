# DevNations Live: Tips for CI/CD Jenkins Demo



Install minishift to get a local OpenShift cluster on your workstation:
https://docs.openshift.org/latest/minishift/getting-started/installing.html 

Start minishift:

```
minishift start --vm-driver virtualbox --memory 4096 --openshift-version=v3.9.0
```


Install Java imagestream

```
oc login -u system:admin
oc create -f https://raw.githubusercontent.com/jboss-openshift/application-templates/ose-v1.4.14/openjdk/openjdk18-image-stream.json -n openshift
```

Create projects

```
oc login -u developer

oc new-project dev
oc new-project prod
oc new-project cicd
```

Deploy Jenkins

```
oc new-app jenkins-persistent --param=MEMORY_LIMIT=2Gi -n cicd
```

Give Jenkins access to dev and prod projects

```
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n dev
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n prod
```

Create Mapit Spring app 

```
oc new-app java:8~https://github.com/siamaksade/mapit-spring.git -n dev
oc expose svc/mapit-spring -n dev
```

Set health probes

```
oc set probe dc/mapit-spring --readiness --get-url=http://:8080/ --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10 -n dev
oc set probe dc/mapit-spring --liveness  --get-url=http://:8080/ --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10 -n dev
```


Create simple pipeline

```
oc new-build https://github.com/siamaksade/devnation-cicd-demo --name=mapit-build -n cicd
```


Customize Jenkins slave configurations:

```
oc create -f https://raw.githubusercontent.com/siamaksade/devnation-cicd-demo/master/support/slave-config.yml -n cicd
```

Disable deployment triggers on OpenShift

```
oc set triggers dc/mapit-spring --manual -n dev
```


Change Jenkinsfile in the pipeline object to use OpenShift client plugin

```
oc patch bc mapit-build --patch '{"spec": {"strategy": {"jenkinsPipelineStrategy": {"jenkinsfilePath": "Jenkinsfile-deploy"}}}}' -n cicd
```


Register at https://quay.io, create a repository called _mapit-spring_ and under account settings click on _Generate Encrypted Password_ to get an encrypted password. Replace your username and encrypted password in the following:

```
oc create secret generic quay-credentials --from-literal=username=USERNAME --from-literal=password=PASSWORD -n cicd
oc label secret quay-credentials credential.sync.jenkins.openshift.io=true -n cicd
```


Create the release pipeline

```
oc create -f https://raw.githubusercontent.com/siamaksade/devnation-cicd-demo/master/support/pipeline-release.yml -n cicd
```


Create some tags as if they were released.

```
oc tag dev/mapit-spring:latest dev/mapit-spring:1.0
oc tag dev/mapit-spring:latest dev/mapit-spring:1.1
oc tag dev/mapit-spring:latest dev/mapit-spring:1.2

oc tag dev/mapit-spring:latest prod/mapit-spring:1.0
```

Deploy Mapit Spring in prod

```
oc new-app mapit-spring:1.0 -n prod
oc expose svc/mapit-spring -n prod
```

Set health probes

```
oc set probe dc/mapit-spring --readiness --get-url=http://:8080/ --initial-delay-seconds=30 --failure-threshold=10 --period-seconds=10 -n prod
oc set probe dc/mapit-spring --liveness  --get-url=http://:8080/ --initial-delay-seconds=180 --failure-threshold=10 --period-seconds=10 -n prod
```

Disable deployment triggers on OpenShift

```
oc set triggers dc/mapit-spring --manual -n prod
```

Create prod pipeline

```
oc create -f https://raw.githubusercontent.com/siamaksade/devnation-cicd-demo/master/support/pipeline-release.yml -n cicd
```