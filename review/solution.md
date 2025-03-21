# OpenShift WildFly Demo Deployment

This guide provides step-by-step instructions to deploy a WildFly application on OpenShift using Helm and configure it with environment variables and resource limits.

## Prerequisites
- OpenShift CLI (`oc`) installed
- Helm installed
- Access to an OpenShift cluster

## Steps

### 1. Log in to OpenShift
```bash
# Log in as a developer user
oc login -u developer -p developer https://api.ocp4.example.com:6443
```

### 2. Create a new OpenShift project
```bash
# Create a new OpenShift project called wildfly-demo
oc new-project wildfly-demo
```

### 3. Add the WildFly Helm repository and search for available charts
```bash
# Add the WildFly Helm chart repository
helm repo add wildfly https://docs.wildfly.org/wildfly-charts/

# Search for available WildFly Helm charts
helm search repo wildfly
```

### 4. Install the MicroProfile Config Example application
```bash
# Deploy the microprofile-config-app using the WildFly Helm chart
helm install microprofile-config-app wildfly/wildfly -f https://raw.githubusercontent.com/wildfly/wildfly-charts/main/examples/microprofile-config/microprofile-config-app.yaml
```

### 5. Monitor the build and deployment process
```bash
# Watch the build process
oc get build -w

# Watch the deployment status
oc get deployment microprofile-config-app -w
```

### 6. Check application logs
```bash
# View logs of the build process
oc logs -f build/microprofile-config-app-1
```

### 7. Monitor the running pods and retrieve the application route
```bash
# Watch for running pods
oc get pods -w

# Retrieve the application route
oc get route
```

### 8. Test the application endpoint
```bash
# Perform a test request to the application endpoint
curl -k https://microprofile-config-app-wildfly-demo.apps.ocp4.example.com/config/value
```

Expected: "Hello from OpenShift"

### 9. Create a ConfigMap and inject it into the deployment
```bash
# Create a ConfigMap with a custom environment variable
oc create configmap democonfig --from-literal=CONFIG_PROP="Hello D280!"

# Set the environment variable from the ConfigMap in the deployment
oc set env --from=configmap/democonfig deployment/microprofile-config-app
```

### 10. Monitor pod restarts and verify changes
```bash
# Check the status of the pods
oc get pods

# Watch for new pod creation after environment update
oc get pods -w
```

### 11. Verify the updated configuration in the application
```bash
# Perform a test request to verify the new configuration value
curl -k https://microprofile-config-app-wildfly-demo.apps.ocp4.example.com/config/value
```

Expected: "Hello D280!"

### 12. Set resource limits for the deployment
```bash
# Set memory limit for the deployment
oc set resources deployment microprofile-config-app --limits=memory=512Mi
```

### 13. Scale the deployment
```bash
# Scale up the deployment to 4 replicas
oc scale deployment microprofile-config-app --replicas 4
```

### 14. Log in as an administrator and monitor cluster resource usage
```bash
# Log in as an admin user
oc login -u admin -p redhatocp

# Check resource usage of OpenShift nodes
oc adm top nodes
```

### 15. Scale down the deployment
```bash
# Scale down the deployment to 0 replicas
oc scale deployment microprofile-config-app --replicas 0
```

### 16. Deploy the MicroProfile Health application
```bash
# Deploy the microprofile-health example using Helm
helm install microprofile-health wildfly/wildfly -f https://raw.githubusercontent.com/wildfly/quickstart/refs/heads/main/microprofile-health/charts/helm.yaml
```

### 17. Create an additional service for the Health Service on Port 9990

```bash
oc expose deployment microprofile-health --port=9990 --target-port=9990 --name=microprofile-health-service
```

### 18. Expose and test the additional service

```bash
oc expose service microprofile-health-service 
oc get route
curl microprofile-health-service-wildfly-demo.apps.ocp4.example.com/health
```

Expected:

```
"status":"UP","checks":[{"name":"deployments-status","status":"UP","data":{"microprofile-health.war":"OK"}},{"name":"suspend-state","status":"UP","data":{"value":"RUNNING"}},{"name":"server-state","status":"UP","data":{"value":"running"}},{"name":"boot-errors","status":"UP"},{"name":"Simple health check","status":"UP"},{"name":"Health check with data","status":"UP","data":{"foo":"fooValue","bar":"barValue"}},{"name":"Database connection health check","status":"UP"},{"name":"started-deployment.microprofile-health.war","status":"UP"}]}[
```
  
