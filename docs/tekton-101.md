# Tekton 101

## Overview

Tekton is unlike other familiar ci/cd tools like jenkins where Tekton is run in a k8s cluster or other k8s distribution, it works inside k8s where our ci/cd process is run inside a pod as a container. Since it runs as a container, Tekton is highly customizable and reusable where an organization can create and reuse its ci/cd component across multiple projects and teams.

## History of Tekton

Tekton started as internal Google tools in knative project and then donated to Continuous Delivery Foundation, where now we can use it in various Kubernetes distribution.

## Component of Tekton

### 1. Task

Task is the smallest component to construct a ci/cd solution inside Tekton, inside task there's steps where we can define and arrange steps in a specific order of execution as part of your continuous integration flow. Users can re-use tasks on one or more pipelines to make it versatile and don't require user to modify entire task. below is simple example of a Task.

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: task-name
spec:
  params:
    - name: parameter-name
      default: parameter-value
  workspaces:
    - name: workspace-name
  steps:
    - name: build-sources
      image: ubuntu
      script: |
       #!/usr/bin/env bash
       echo $(params.parameter-name)
```

### 2. Pipeline

An pipeline is a collection of task that user can configure to suit its ci/cd needs, for example in this project [pipeline](../tekton/pipeline-run.yaml) the task arranged to first clone the source code then it build an container image push it to registry then the last step to deploy the container to the cluster.

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: pipeline-name
spec:
  workspaces:
    - name: workspace-name
  params:
   - name: parameter-name
     value: parameter-value
  tasks:
    - name: taskName1
      taskRef:
        kind: Task
        name: task-name
      params:
        - name: parameter-name
          value: "$(params.parameter-name)"
      workspaces:
        - name: workspace-name
          workspace: workspace-name
```

### 3. Workspace

Workspaces allow Tasks to declare parts of the filesystem that need to be provided at runtime by TaskRuns. using a read-only ConfigMap or Secret, an existing PersistentVolumeClaim shared with other Tasks to allow each task to do modification during ci/cd process is also available. Example below is how to use workspace with pvc for sharing data among Tasks.

```yaml
workspaces:
  - name: workspace-name
    persistentVolumeClaim:
      claimName: pvc-name
    subPath: my-subdir
```

### 4. Parameters

You can specify parameters, such as compilation flags or artifact names, that you want to supply to the Task at execution time. The following example illustrates the use of Parameters in a Task

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: task-with-parameters
spec:
  params:
    - name: someURL
      type: string
  steps:
    - name: build
      image: my-builder
      args: [
        "build",
        'url=$(params.someURL)',
      ]
```
