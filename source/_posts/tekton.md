title: tekton（个人笔记）
date: 2022-04-16 22:56:52
tags:
author:
---
# 快速开始

## 部署
从[https://github.com/tektoncd/cli/releases](https://github.com/tektoncd/cli/releases)下载tkn工具，该工具为tekton的命令行工具。

执行如下命令安装dashboard
```yaml
kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
```
dashboard安装完成后仅申请了ClusterIP类型的Service，可以在本地通过kubectl端口转发的方式来对外提供服务，既可以通过ip:9097的方式来对外提供服务了。
```powershell
kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097 --address 0.0.0.0 
```
执行如下命令，在k8s中提交yaml
```powershell
kubectl apply --filename \ https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```
会在k8s中提交如下两个deployment，此时tekton即安装完毕。
```powershell
$ kubectl  get deploy -n tekton-pipelines 
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
tekton-pipelines-controller   1/1     1            1           4m
tekton-pipelines-webhook      1/1     1            1           4m
```
## 创建task
创建如下yaml，提交一个任务定义，提交完成后任务并不会被执行
```powershell
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: echo
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello World"  
```
继续提交如下的yaml，提交一次任务运行实例
```powershell
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: hello-task-run
spec:
  taskRef:
    name: hello
```
查看任务执行情况
```powershell
$ kubectl get taskrun hello-task-run
NAME             SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
hello-task-run   True        Succeeded   45s         22s

# 可以看到创建出了任务执行的pod
$ kubectl  get pod 
NAME                 READY   STATUS      RESTARTS   AGE
hello-task-run-pod   0/1     Completed   0          2m10s
```
## 创建流水线任务pipeline
创建新的task goodye
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: goodbye
spec:
  steps:
    - name: goodbye
      image: ubuntu
      script: |
        #!/bin/bash
        echo "Goodbye World!"
```
提交如下yaml，创建pipeline定义
```powershell
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-goodbye
spec:
  tasks:
    - name: hello
      taskRef:
        name: hello
    - name: goodbye
      runAfter:
        - hello
      taskRef:
        name: goodbye
```
提交如下yaml，创建pipeline实例。
```powershell
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-goodbye-run
spec:
  pipelineRef:
    name: hello-goodbye
```
可以使用tkn来查看pipeline拉起的pod日志，该工具会将pod的日志合并到一起。
```yaml
$ tkn pipelinerun logs hello-goodbye-run -f -n default
[hello : echo] Hello World

[goodbye : goodbye] Goodbye World!
```
