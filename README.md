# fis2 generic run template

A simple re-usable template to use in run namespaces for Openshift for spring boot binary builds.  The only items created are:

- A DeploymentConfig, and
- A Service

You may make modifications to the sections in `src/main/fabric8` as appropriate.  This project just uses the `raw` profile.

## How to use?

Firstly, grant permissions for your run namespace to access your build namespace imagestream:

`oc policy add-role-to-user system:image-puller system:serviceaccount:run-namespace:default -n build-namespace`

Then follow the following:

```
# build the template from the fragments
mvn clean package

# create a "run namespace"
oc new-project run-namespace

# Create the Template in the run-namespace
oc create -f target/classes/META-INF/fabric8/openshift.yml
OR
oc create -f template.yml


# Create the Openshift Objects via new-app
[user@localhost deploy]$ oc new-app --template=fis2-generic-runtemplate -p APPLICATION_NAME=my-app -p REGISTRY_IP=$(oc get svc/docker-registry -n default -o json | jq -r .spec.clusterIP) -p IMAGESTREAM_NAMESPACE=build-namespace -p IMAGESTREAM_TAG=develop
--> Deploying template "run-namespace/fis2-generic-runtemplate" to project run-namespace

     * With parameters:
        * APPLICATION_NAME=my-app
        * REGISTRY_IP=172.30.1.1
        * IMAGESTREAM_NAMESPACE=build-namespace
        * IMAGESTREAM_TAG=develop

--> Creating resources ...
    service "my-app" created
    deploymentconfig "my-app" created
--> Success
    Run 'oc status' to view your app.
```

When a new build is triggered in the build-namespace and the imagestream is tagged with `develop`, then the latest develop image is deployed automatically via the image change triggers in the `deploymentConfig`