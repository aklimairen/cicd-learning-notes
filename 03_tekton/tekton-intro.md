# Tekton

## Overview

Tekton is an open-source, **cloud-native CI/CD framework** that runs on Kubernetes. Unlike GitHub Actions (which is GitHub-specific), Tekton is platform-agnostic — it works on any Kubernetes cluster (OpenShift, GKE, AKS, minikube).

All pipeline components are defined as **YAML manifests** (Kubernetes CRDs) and applied to the cluster using `kubectl`.

---

## Core Concepts

Each pipeline is built from the following components in order:

**Task → TaskRun → Pipeline → PipelineRun**

### Task

The basic building block. Contains one or more  **Steps** , each running inside a container image.

* Defines `params` (inputs), `workspaces` (shared storage), and `results` (outputs)
* Applied to the cluster: `kubectl apply -f task.yaml`

### TaskRun

An instance that executes a specific `Task` and supplies runtime parameter values.

### Pipeline

A collection of **Tasks** arranged in a specific order.

* Tasks can run **sequentially** or **in parallel**
* Use `runAfter:` to enforce ordering

### PipelineRun

An instance that executes a specific `Pipeline` and provides runtime parameters.

---

## Basic Task YAML Structure

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: nose                        # Task name
spec:
  workspaces:
    - name: source                  # Shared storage (e.g. cloned repo)
  params:
    - name: args                    # Input parameter
      description: Arguments to pass to nose
      type: string
      default: "-v"
  steps:
    - name: nosetests               # Step name
      image: python:3.9-slim        # Container image
      workingDir: $(workspaces.source.path)
      script: |
        #!/bin/bash
        set -e
        python -m pip install --upgrade pip wheel
        pip install -r requirements.txt
        nosetests $(params.args)    # Reference params with $(params.<name>)
```

---

## Other Important Points

* **Workspaces:** Shared volumes passed between tasks (e.g. to share cloned source code between a `git-clone` task and a test task)
* **Security Context:** Use `securityContext` on a step to control user permissions — e.g. `runAsUser: 0` runs as root (needed for cleanup tasks that delete files)
* **Tekton Hub:** Community-maintained catalog of reusable Tasks — similar to the GitHub Actions Marketplace. Browse at [hub.tekton.dev](https://hub.tekton.dev/) and install with `tkn hub install task <name>`
* **Triggers:** Automate pipeline execution on GitHub events (push, PR) using `EventListener` → `TriggerBinding` → `TriggerTemplate`
* **Image Building:** Use `buildah` or `kaniko` catalog tasks to build and push Docker images inside Kubernetes — no Docker daemon required
* **Deploying:** Use `kubectl set image` (Kubernetes) or `oc set image` (OpenShift) in a final pipeline task to deploy the built image
* **RBAC:** The `ServiceAccount` running the pipeline needs the correct role bindings to apply or update deployments
* **`tkn` CLI:** Manage Tekton resources from the terminal:
  ```bash
  tkn pipeline listtkn pipelinerun logs <run-name> -ftkn hub install task git-clone
  ```

---

## Real Example: Cleanup Task

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  description: This task will clean up a workspace by deleting all the files.
  workspaces:
    - name: source
  steps:
    - name: remove
      image: alpine:3
      env:
        - name: WORKSPACE_SOURCE_PATH
          value: $(workspaces.source.path)      # Pass workspace path as env var
      workingDir: $(workspaces.source.path)
      securityContext:
        runAsNonRoot: false
        runAsUser: 0                            # Root required to delete all files
      script: |
        #!/usr/bin/env sh
        set -eu
        echo "Removing all files from ${WORKSPACE_SOURCE_PATH} ..."
        if [ -d "${WORKSPACE_SOURCE_PATH}" ] ; then
          rm -rf "${WORKSPACE_SOURCE_PATH:?}"/*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/.[!.]*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/..?*
        fi
```
