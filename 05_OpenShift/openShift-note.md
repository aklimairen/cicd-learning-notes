# OpenShift

## Overview

OpenShift is  **Kubernetes with extra features built on top** , made by Red Hat. Everything you can do with `kubectl` in Kubernetes, you can do with `oc` in OpenShift — it just adds a built-in image registry, automatic public URLs (Routes), stricter security, and a developer web console.

In a CI/CD pipeline, OpenShift is typically where your final built image gets  **deployed and runs** .

---

## Core Concepts

OpenShift adds its own resources on top of standard Kubernetes:

**Project → ImageStream → DeploymentConfig → Route**

### Project

OpenShift's version of a Kubernetes Namespace — an isolated space where your app and its resources live.

### ImageStream

A pointer that tracks a container image and its tags inside OpenShift. Instead of hardcoding an image URL everywhere, a DeploymentConfig references the ImageStream — when the image updates, OpenShift can automatically trigger a new rollout.

### DeploymentConfig (DC)

Similar to a Kubernetes `Deployment`. Describes how your app should run — which image, how many replicas, environment variables, and rollout triggers.

### Route

Gives your app a **public URL** so it can be accessed from outside the cluster. Created by exposing a Service.

---

## Basic `oc` CLI Commands

```bash
# Login & project setup
oc login <cluster-url>              # Log in to the cluster
oc new-project my-project           # Create a new project
oc project my-project               # Switch to a project

# Viewing resources
oc get all                          # See everything in the current project
oc get pods                         # List running pods
oc get routes                       # List public URLs

# Deploying & updating
oc apply -f deployment.yaml         # Apply a manifest (same as kubectl apply)
oc set image dc/myapp myapp=<image> # Update the image in a deployment
oc rollout latest dc/myapp          # Trigger a new rollout manually
oc expose service myapp             # Create a public Route for a service

# Debugging
oc logs <pod-name>                  # View pod logs
oc describe pod <pod-name>          # Detailed info about a pod
```

---

## Other Important Points

* **`oc` vs `kubectl`:** `oc` is a superset of `kubectl` — all `kubectl` commands work with `oc`, plus OpenShift-specific ones like `oc new-project`, `oc expose`, and `oc rollout latest`
* **Internal Registry URL:** When building and pushing images inside OpenShift, you use the built-in registry with this format:
  ```
  image-registry.openshift-image-registry.svc:5000/<project>/<image>:<tag>
  ```
* **Security (SCCs):** OpenShift blocks containers that run as root by default. Security Context Constraints (SCCs) control what a pod is allowed to do — the default `restricted` SCC prevents root access. This is the most common reason things that work in plain Kubernetes fail in OpenShift
* **ServiceAccount Permissions:** The pipeline's ServiceAccount needs the correct role to deploy. Grant it with:
  ```bash
  oc adm policy add-role-to-user edit -z pipeline
  ```
* **Deploying from Tekton:** Use the `openshift-client` catalog task to run `oc` commands inside your pipeline:
  ```yaml
  - name: deploy  taskRef:    name: openshift-client  params:    - name: SCRIPT      value: |        oc set image deployment/myapp myapp=image-registry.openshift-image-registry.svc:5000/$(context.pipelineRun.namespace)/myapp:latest        oc rollout status deployment/myapp
  ```
* **Routes:** After deploying, expose your service to get a public URL: `oc expose service myapp` — then `oc get routes` to see the assigned URL
