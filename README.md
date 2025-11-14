# OpenShift 4 Troubleshooting Lab — Quote of the Day (QOTD)

This lab guides you through deploying a multi-component application and troubleshooting common issues encountered on OpenShift.
You will deploy a backend Python service that serves random quotes, and a frontend web UI that consumes the backend.

The lab is intentionally broken in several places so you can practice diagnosing and resolving real-world issues.

## 1. Create a New Project

Create a dedicated project for your work:

```
oc new-project quotes-$USER
```

Verify:

```
oc project
```

## 2. Review & Deploy the Backend (Python API)

Before deploying, inspect the manifests provided in the k8s directory:

* Check image references
* Check labels
* Check Service selectors
* Check Route configuration
* Check Deployment resource definitions

Deploy the backend components:

```
oc apply -f qotd-python/k8s/quota.yaml
oc apply -f qotd-python/k8s/quotes-deployment.yaml
oc apply -f qotd-python/k8s/route.yaml
oc apply -f qotd-python/k8s/service.yaml
```

The backend endpoint is expected at:
```
http://<route>/quotes/random
```

## 3. Troubleshoot: Backend Pod Fails to Start
Your backend will **fail intentionally**. Your task is to determine why.

### Check for image pull issues

```
oc get events
```

Look for:

* `Failed to pull image`
* `ImagePullBackOff`
* `ErrImagePull`

### Inspect the pod state

```
oc get pods
oc describe pod <podname>
```

Identify:

* CrashLoopBackOff?
* Wrong image tag?
* Wrong registry?
* Missing pull secret?


### Check pod logs

```
oc logs <podname>
```

## 4. Determine the Incorrect Configuration

Using the information gathered, answer:

* Is the image reference correct?
* Is the registry reachable?
* Does the tag exist?
* Is a different tag expected?

Browse to the referenced registry and inspect the available image tags.

## 5. Fix & Redeploy the Backend

Edit the deployment manifest:

```
vi qotd-python/k8s/quotes-deployment.yaml
```

After fixing:

```
oc apply -f qotd-python/k8s/quotes-deployment.yaml
```

Validate:

```
oc get pods
oc logs <pod>
oc get events
```

Confirm that:

* The pod starts successfully
* The endpoint works:

```
curl http://<route>/quotes/random
```

## 6. Troubleshoot “Application is Not Available”
Attempt to access the backend route. The route may return: `Application is not available`

### Verify backend pod is running

```
oc get pods
```

Inspect backend pod configuration

```
oc get pod <pod> -o yaml
```

Check:

* Labels
* Container ports
* Readiness and liveness probes

### Inspect the route

```
oc get route
```

Identify:

* Which Service the Route targets

### Inspect the Service

```
oc get svc
oc get svc <svcname> -o yaml
```


Check:

* Selector labels match the Deployment’s pod template labels
* Port and targetPort alignment

Fix the issue and reapply manifests.
Verify the endpoint works:

```
curl http://<route>/quotes/random
```

## 7. Deploy the Frontend Web Application

Apply the frontend manifests:

```
oc apply -f quotesweb/k8s/quotes-deployment.yaml
oc apply -f quotesweb/k8s/route.yaml
oc apply -f quotesweb/k8s/service.yaml
```

Validate the deployment

```
oc get deployment
oc get pods
oc logs <pod>
oc get events
```

You should observe failures. Continue investigating.

## 8. Troubleshoot: Frontend Deployment Blocked by ResourceQuota

List quotas:

```
oc get quota

```

Determine:

* Which resource quota is causing the failure?
* Is it CPU, memory, number of pods, etc.?

Fix the quota:

```
oc edit quota <name>
```

After adjusting resources, restart the deployment. This is not necessariliy needed, but speeds up detection by the scheduler:

```
oc rollout restart deployment/quotesweb
```


Verify everything is running:

```
oc get pods
```

## 9. Use the Frontend Application

Open the frontend route and test the UI.

Enter the backend URL into the form:

```
http://<backend-route>/quotes/random
```

You should now see quotes displayed through the frontend.



Inspired by: https://github.com/redhat-developer-demos/qotd-python & 
