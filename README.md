# Spring-Boot ConfigMap Demo

This example demonstrates how you can use Kubernetes ConfigMap to configure Spring Boot and Apache Camel.

### Code generation
Project is generated with the command:

    mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate \
      -DarchetypeCatalog=https://maven.repository.redhat.com/earlyaccess/all/io/fabric8/archetypes/archetypes-catalog/2.2.0.fuse-000092-redhat-2/archetypes-catalog-2.2.0.fuse-000092-redhat-2-archetype-catalog.xml \
      -DarchetypeGroupId=org.jboss.fuse.fis.archetypes \
      -DarchetypeArtifactId=spring-boot-camel-archetype  \
      -DarchetypeVersion=2.2.0.fuse-000092-redhat-2


### Configuration
The only changes done to the above artifact are:

Tell Spring Boot Application.java to look for optional application.properties under subfolder: config

    @Configuration
    @PropertySources(value = {
            @PropertySource(value = "classpath:application.properties"),
            @PropertySource(value = "file:/etc/config/application.properties", ignoreResourceNotFound = true) })
    public class Application extends RouteBuilder {
    
        @Value("${host.name}")
        private String hostName;
        

Tell OpenShift to mount volume 'volume-demo' under /deployments/config/.
Tell OpenShift the volume 'volume-demo' comes from ConfigMap named 'demo'.

    deployment.yml

          volumeMounts:
            - mountPath: /deployments/config/
              name: volume-demo
      volumes:
        - configMap:
            name: demo
          name: volume-demo

### Building

    mvn clean install -s configuration/settings.xml

### Create ConfigMap
Create a ConfigMap with name 'demo' and application.properties as its content (Notice there is application.properties in resources that has the default values of the app, application.properties at top level which represents environment specific properties). When the ConfigMap is present in a namespace, it will be mounted as volume and override the default value from application.properties embedded in the jar file.

    oc create configmap demo --from-file=application.properties

### Deploying in OpenShift
It is assumed that OpenShift platform is already running and your system is configured for Fabric8 Maven Workflow.

    mvn fabric8:deploy
 
### Check Configurations are overridden
Check ConfigMap is mounted as a folder:

    oc rsh $(oc get pods -l app=spring-boot-configmap --template '{{range .items}}{{.metadata.name}}{{end}}') cat /etc/deployments/application.properties | grep host.name

Check Springs picks the environment specific values and uses them:

    oc logs $(oc get pods -l app=spring-boot-configmap --template '{{range .items}}{{.metadata.name}}{{end}}') | grep host.name

The value should not be the embeded value (default_value) but the overriden value: env_specific_value
