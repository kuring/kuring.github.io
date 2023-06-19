---
title: 手动更新Custom Resource中的status部分
date: 2020-06-21 21:27:09
tags:
---

在k8s中的内置资源很多都有status部分，比如deployment，用来标识当前资源的状态信息。同样在CRD的体系中，也都有status部分。这些status部分信息，是由operator来负责维护的。

如果直接采用kubectl edit的方式来修改status部分信息，会发现是无法直接修改status部分的，因为status是无法修改成功的，因为status部分是CR的一个子资源。

可以通过如下的方式来完成修改

1. 首先要准备一个完整的yaml文件，包含了status部分信息

这个的格式必须为json

```
{
  "apiVersion": "kuring.me/v1alpha1",
  "kind": "Certificate",
  "metadata": {
    "name": "test",
    "namespace": "default"
  },
  "spec": {
    "secretName": "test"
  },
  "status": {
    "phase": "pending"
  }
}
```

2. 获取系统的TOKEN信息

通常在kube-system下会有admin的ServiceAccount，会有一个对应的Secret来存放该ServiceAccount的token信息。执行`kubectl get secret -n kube-system admin-token-r2bvt   -o yaml`获取到token信息，并其中的token部分进行base64解码。

3. 执行如下的脚本

需要将其中的变量信息修改一下

```
#!/bin/bash

obj_file=$1
kind=certificates
APISERVER=https://10.0.0.100:6443
namespace=default
TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1yMmJ2dCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImRiY2IyNzUzLWE5OGMtMTFlYS04NGVjLTAwMTYzZTAwOGU3MCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.tX3jyNh-GEuZQg-hmy7igqh9vpTAz8Jh9uEv-diZ5XWjX9JYhxwD9nxTQvCcvzY7iPIbvxQfW2GHDZISPoopX0vQy9mQ7npVitrOvFovk06plefI5Gxjdft6vdpt-ArsGTpm7-s9G-3aBg5x41h3Cdgyv-W-ypFlCr9dKu9K7BcRIXSq_GQlq5TBmd-LKFXoer4QGwkn7geq5-ziMk_lY21jIGVdIkq9IRiH8NWuCl7l8i6nQESQDUUpMyKDCqkJqUFV8UkrQL7TfqurFP36_TUAQTh2ZAE8nFnrKRoa09BnjT-FoPO6Jnq6COQjk3PGDHV8LKNDAjCCrs0A53IYGw
obj=nginx-test

echo "begin to patch $obj the file "${obj_file}
curl -XPATCH -H "Accept: application/json" -H  "Content-Type: application/merge-patch+json" --header "Authorization: Bearer $TOKEN" --insecure -d  @${obj_file} $APISERVER/apis/kuring.me/v1alpha1/namespaces/${namespace}/${kind}/$obj/status
```

