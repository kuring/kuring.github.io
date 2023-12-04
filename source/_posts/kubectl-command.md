title: kubectl常用命令
date: 2022-01-18 15:10:14
tags:
---
本文记录常用的kubectl命令，不定期更新。

## 1. 统计k8s node上的污点信息

```
kubectl get nodes -o=custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect
```

## 2. 查看不ready的pod

```
kubectl get pod --all-namespaces -o wide -w | grep -vE "Com|NAME|Running|1/1|2/2|3/3|4/4"
```

## 3. pod按照重启次数排序

```
kubectl get pods -A  --sort-by='.status.containerStatuses[0].restartCount' | tail
```

## 4. kubectl proxy命令代理k8s apiserver

该命令经常用在开发的场景下，用来测试k8s api的结果。执行如下命令即可在本地127.0.0.1开启10999端口。

```
kubectl proxy --port=10999
```

在本地即可通过curl的方式来访问k8s的apiserver，而无需考虑鉴权问题。例如，`curl http://127.0.0.1:10999/apis/batch/v1`，即可直接返回结果。

## 5. `--raw`命令

该命令经常用在开发的场景下，用来测试k8s api的结果。执行如下的命令，即可返回json格式的结果。

```
kubectl get --raw /apis/batch/v1
```

## 6. 查看集群中的pod的request和limit情况

```
kubectl get pod -n kube-system  -o=custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,PHASE:.status.phase,Request-cpu:.spec.containers\[0\].resources.requests.cpu,Request-memory:.spec.containers\[0\].resources.requests.memory,Limit-cpu:.spec.containers\[0\].resources.limits.cpu,Limit-memory:.spec.containers\[0\].resources.limits.memory
```

得到的效果如下：

```
NAME                                              NAMESPACE     PHASE       Request-cpu   Request-memory   Limit-cpu   Limit-memory
cleanup-for-np-processor-9pjkm                    kube-system   Succeeded   <none>        <none>           <none>      <none>
coredns-6c6664b94-7rnm8                           kube-system   Running     100m          70Mi             <none>      170Mi
coredns-6c6664b94-djxch                           kube-system   Failed      100m          70Mi             <none>      170Mi
coredns-6c6664b94-khvrb                           kube-system   Running     100m          70Mi             <none>      170Mi
```

## 7. 修改对象的status

因为status资源实际上为对象的subResource资源，实际上无法通过kubectl来完成，但该操作还是放到了该文档中。

下述命令的需求为修改service的status

```
APISERVER=https://kube-cn-nb-nbsjzxpoc-d01-a.intra.nbsjzx.ali.com:6443
TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1jazdkciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjhkZWE4MWQ4LTU2YTgtMTFlYy05MDMyLTgwNjE1ZjA4NDI0YSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.O25G_MSmKRU_pIPO_9tFqDvbZm9SOM_Mix7jCJeFiZHzLiSc7n5RanP3QoEldR1IcFN4AZXzlzI1Rb0GyFQH7XmS1eLESMbKnrTR3N5s3wlRp-D5QT0c_RAVAiLKrP7KsSKNcLjQkIO8-Csf_kmizIh6fP0-b7Mp90cw0J8oSlM-ScEeUEksQyXvyisVN9qvvTIkmbsgh7pX5IFcB4mGbvV9loOUC3-LdiiaCkMJzMisEeSJRaajmLlwpWXtb2BSi9HmBMktUE9IVB8ryZ06wbPRianbjoBwtAhcXuRyj1LaEog3aJHsyMA_DOZJtvjYis60AIRZ1iBnc-gZtEFCxw
obj=nginx-ingress-lb
curl -XPATCH -H "Accept: application/json" -H  "Content-Type: application/merge-patch+json" --header "Authorization: Bearer $TOKEN" --insecure -d '{"status": {"loadBalancer": {"ingress": [{"ip": "10.209.97.170"}]}}}' $APISERVER/api/v1/namespaces/acs-system/services/nginx-ingress-lb/status
```

APISERVER变量可以通过kubeconfig文件获取到。

TOKEN变量可以直接使用kube-system下的admin账号。
1. 执行 `kubectl get secrets -n kube-system  | grep admin` 获取到sa对应的secret
2. 执行 `kubectl get secrets -n kube-system xxxx -o yaml` 获取到base64之后的token
3. 执行 `echo "xxxx" | base64 -d` 即可获取到对应的token

obj变量为要修改的service对象名称。

## 8. 查看 secret 内容

```
kubectl get secret -n ark-system ark.cmdb.https.origin.tls -o jsonpath='{.data.ca\.pem}' | base64 -d
```

## 9. 修改 secret 或 cm 的内容

很多场景下使用 `kubectl edit` 修改不能完全满足需求，比如某个 key 对应的 value 非常长且包含空格，很难直接编辑。可以通过导出 key 对应的 value 到文件，然后再重新 apply 的方式合入。

导出 configmap 中特定的 key：

```
kubectl get cm -n kube-system  networkpolicy-config -o jsonpath='{.data.config\.yaml}' -o yaml
```

修改完成后，将文件重新 apply cm

```
kubectl create --save-config cm  networkpolicy-config -n kube-system --from-file /tmp/config.yaml -o yaml --dry-run | kubectl apply -f -
```

## 9. 删除所有 pod（或特定状态 pod）

```
kubectl get pods --all-namespaces -o wide --no-headers | grep -v Running | awk '{print $1 " " $2}' | while read AA BB; do kubectl get pod -n $AA $BB --no-headers; done
```

## 10. kubectl debug

常用于网络问题排查，其中镜像 netshoot 中包含了丰富的网络命令。

```
kubectl exec -it nginx-statefulset-0 bash
```
