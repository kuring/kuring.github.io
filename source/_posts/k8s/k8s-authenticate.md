---
url: k8s-authenticate
Status: public
date: 2024-04-22
title: k8s 中的用户认证方式
tags:
  - k8s
---

在 k8s 中包含两类用户：

1. ServiceAccount。又称服务账号，在运行 pod 时必须绑定 ServiceAccount，如果没有指定，则使用当前 namespace 下的 ServiceAccount default。是针对程序而言，用于 pod 中的程序访问 kube-apiserver。
2. 普通用户。在 k8s 中并没有使用单独的对象来存储，而是通过了分发证书、外部用户认证系统等方式实现，是针对用户而言。

# 1. X509 证书认证

使用场景：使用 kubectl 访问 k8s 集群即通过 X509 证书认证方式，kubeconfig 本质上是个证书文件。

客户端使用证书中的 Common Name 作为请求的用户名，organization 作为用户组的成员信息。

证书的签发可以使用 openssl、cfssl 等工具来签发，也可以使用 k8s 自带的 CertificateSigningRequest 对象来实现签发。

## 1.1. CertificateSigningRequest 签发证书

创建私钥信息：

```
openssl genrsa -out myuser.key 2048
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser"
```

创建如下的 CertificateSigningRequest 对象

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  # value 使用命令 cat myuser.csr | base64 | tr -d "\n" 获取
  request: xxx
  # 固定值
  signerName: kubernetes.io/kube-apiserver-client
  # 过期时间
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF
```

查看 csr 处于 Pending 状态：

```
$ kubectl get csr
NAME     AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
myuser   12s   kubernetes.io/kube-apiserver-client   kubernetes-admin   24h                 Pending
```

批准给证书签发请求：

```
kubectl certificate approve myuser
```

签发完成后的证书会存放到 status.certificate 字段中，至此证书签发完成。

# 2. ServiceAccount

使用场景：该方式使用较为常见，用于 pod 中访问 k8s apiserver。

原理：pod 可以通过 spec.serviceAccountName 字段来指定要使用的 ServiceAccount，如果没有指定则使用 namespace 下默认的 default ServiceAccount。kube-controller-manager 中的 ServiceAccount 控制器会在拉起的 pod 中自动注入如下的信息：

```
spec:
  volumeMounts: 
  - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
    name: kube-api-access-j6vpz
    readOnly: true
  volumes:
  - name: kube-api-access-j6vpz
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

即将信息注入到 pod 的 /var/run/secrets/kubernetes.io/serviceaccount 目录下，目录结构如下：

```
/var/run/secrets/kubernetes.io/serviceaccount
|-- ca.crt -> ..data/ca.crt
|-- namespace -> ..data/namespace
`-- token -> ..data/token
```

token 为 JWT 认证，对其格式解密后如下：

```
{
  "alg": "RS256",
  "kid": "u7rF5JCtJRNiMzSUOFAYvDpCwPqUII-N-OtxR59cnQ0"
}
```

```
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "default",
  "kubernetes.io/serviceaccount/secret.name": "ingress-token-s9gtm",
  "kubernetes.io/serviceaccount/service-account.name": "ingress",
  "kubernetes.io/serviceaccount/service-account.uid": "19ce4f11-7105-43ce-b189-f3d71a2ffc74",
  "sub": "system:serviceaccount:default:ingress"
}
```

## 2.1. Secret 存放 token

在 1.22 版本及之前版本中，token 以 Secret 的形式存在于 pod 所在的 namespace 下，且 token 不会过期。Secret 的名字存在于 ServiceAccount 的 spec 中，格式如下：

```
apiVersion: v1
kind: ServiceAccount
secrets:
- name: nginx-token-scjvn
```

而 Secret 通过 Annotation `kubernetes.io/service-account.name` 指定了关联的 ServiceAccount。

在后续版本中，为了兼容当前方案，如果 ServiceAccount 关联了 Secret，则认为仍然使用 Secret 中存放 token 的方式。如果 Secret 已经很长时间没有使用，则自动回收 Secret。

如果要手工创建一个 token Secret，可以创建如下的 Secret，k8s 自动会为 Secret 产生 token：

```
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    # 带有 annotation
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
```

## 2.2. TokenRequest API 产生 token

在 1.22 之后的版本中，kubelet 使用 TokenRequest API 获取有时间限制的临时 token，该 token

会在 pod 删除或者 token 生命周期（默认为 1h）结束后失效。

可以使用 `kubectl create token default` 来为 ServiceAccount default 创建 token，该命令实际上向 kube-apiserver 发送了请求 `/api/v1/namespaces/default/serviceaccounts/default/token`：

```
{
    "kind": "TokenRequest",
    "apiVersion": "authentication.k8s.io/v1",
    "metadata":
    {
        "creationTimestamp": null
    },
    "spec":
    {
        "audiences": null,
        "expirationSeconds": null,
        "boundObjectRef": null
    },
    "status":
    {
        "token": "",
        "expirationTimestamp": null
    }
}
```

kube-apiserver 支持如下参数：

1. --service-account-key-file：用来验证服务账号的 token。
2. --service-account-issuer：ServiceAccount token 的签发机构。
3. --service-account-signing-key-file：ServiceAccount token 的签发私钥。

# 3. 用户伪装

一个用户通过 Http Header Impersonation- 的方式来扮演另外一个用户的身份。

场景：跨 k8s 集群访问的网关服务

支持的 Http Header 如下：

1. Impersonate-User：要伪装的用户名。
2. Impersonate-Group：要伪装的组名。该 Header 可以为多个，即支持多个组。

# 4. bootstrap token

使用 kube-apiserver 参数 `--enable-bootstrap-token-auth=true` 启用功能，引导 token 以 Secret 的形式存放在 kube-system 下。

该功能仅用于节点初始化时加入到 k8s 集群中。

# 资料

- [用户认证](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/)