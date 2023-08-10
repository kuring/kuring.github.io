title: OCI
date: 2022-06-23 22:25:29
tags:
author:
---
OCI（Open Container Initiative）为Linux基金会下的子项目，成立于2015年，由docker、coreos等公司发起，用来制定开放的容器和容器镜像标准，同时docker将其镜像格式和容器运行时runc捐献给了OCI，因此OCI中包含了容器镜像标准和容器运行时标准两部分。

## 容器镜像的发展历史

2013年，docker横空出世，docker的核心能力之一为将应用封装为镜像，便于打包、发布。

2014年，docker将镜像格式定位为docker镜像规范v1版本。

2016年，docker制定了新的镜像格式规范v2，解决v1版本的部分设计缺陷。

2017年，OCI发布了OCI镜像规范1.0版本，整个规范以docker镜像规范v2为基础，两者的规范比较类似。

## 容器镜像标准

官方定义：[OCI Image Format Specification](https://github.com/opencontainers/image-spec)

容器镜像标准包含了镜像索引（可选）、manifest、层文件和配置文件四部分内容。包含了容器的构建、分发和准备可以运行的镜像。

![https://github.com/opencontainers/image-spec/blob/main/img/media-types.png](https://kuring.oss-cn-beijing.aliyuncs.com/common/oci-media-types.png)

skopeo为一个容器镜像和镜像仓库的命令行操作工具，可以使用该工具来学习OCI容器镜像规范，可以直接使用 `yum install skopeo -y` 进行安装。

使用如下命令可以将docker的镜像转换为oci格式，并将其保存到/tmp/local_nginx目录下。

```bash
skopeo copy docker://nginx oci:/tmp/local_nginx
```

/tmp/local_nginx的目录包含如下结构，参考链接：[OCI Image Layout Specification](https://github.com/opencontainers/image-spec/blob/main/image-layout.md)

- blobs 目录：包含了镜像Manifest、镜像层和镜像配置信息，均已sha256值命名的文件形式存储，文件可以为文本文件，也可以为gzip压缩的二进制文件。
- oci-layout 文件：定义了当前目录结构的版本信息
- index.json 文件：定义了镜像索引信息

```
.
├── blobs
│   └── sha256
│       ├── 4a7307612456a7f65365e1da5c3811df49cefa5a2fd68d8e04e093d26a395d60
│       ├── 67e9751bc5aab75bba375f0a24702d70848e5b2bea70de55e50f21ed11feed14
│       ├── 8f46223e4234ce76b244c074e79940b9ee0a01b42050012c8555ebc7ac59469e
│       ├── 935cecace2a02d2545e0c19bd52fe9c8c728fbab2323fc274e029f5357cda689
│       ├── b85a868b505ffd0342a37e6a3b1c49f7c71878afe569a807e6238ef08252fcb7
│       ├── efb56228dbd26a7f02dafc74a2ca8f63d5e3bb6df9046a921f7d8174e5318387
│       ├── f4407ba1f103abb9ae05a4b2891c7ebebaecab0c262535fc6659a628db25df44
│       └── fe0ef4c895f5ea450aca17342e481fada37bf2a1ee85d127a4473216c3f672ea
├── index.json
└── oci-layout

2 directories, 10 files
```

### 镜像索引

非必须部分。如果包含镜像索引，用来解决多架构问题。不同的平台上，可以使用同一个镜像tag，即可以获取到对应平台的镜像。

镜像索引为 json 格式的文件，查看 index.json 文件，内容如下：

```
{
  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:15beb598b14fca498f13a46923de0614a17012abf675ba06e364904d642d8a61",
      "size": 1183
    }
  ]
}
```

镜像索引文件可以包含多种架构。其中digest对应的sha256值指向 blobs/sha256 下的文件，其文件为Manifest文件。

需要注意的是：docker 可以使用 `docker manifest`命令来查看镜像的manifest信息，但格式并非为OCI Manifest格式，而更类似于OCI index的信息，下面的命令中，可以看到rancher镜像为多镜像。

```
$ docker manifest inspect rancher/rancher
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 4732,
         "digest": "sha256:b8f1fdb8228d32ae5fc6f240503cd8e22b214fcfd4ad2a8a0b03274f3ead4e95",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 4731,
         "digest": "sha256:ae0fa74e8dce9b72bdc6611815deb16bbddc8fe0a555052ccc8127fdc1b76980",
         "platform": {
            "architecture": "arm64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 4519,
         "digest": "sha256:94c03afba43e81885c3cd2f5065032d1b7f8f540860fcc1fce1bbd7f1068d3db",
         "platform": {
            "architecture": "s390x",
            "os": "linux"
         }
      }
   ]
}
```

### Manifest

参考：[OCI Image Manifest Specification](https://github.com/opencontainers/image-spec/blob/main/manifest.md)

Manifest为json格式的描述文件，包含了如下三个用途：

1. 每一个镜像都有一个唯一的 id 标识
2. 对于同一个镜像tag，可以支持多架构镜像
3. 可以直接转换为OCI的运行时规范

查看 blobs/sha256/15beb598b14fca498f13a46923de0614a17012abf675ba06e364904d642d8a61 内容如下：

```
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:67e9751bc5aab75bba375f0a24702d70848e5b2bea70de55e50f21ed11feed14",
    "size": 6567
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:b85a868b505ffd0342a37e6a3b1c49f7c71878afe569a807e6238ef08252fcb7",
      "size": 31379408
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:f4407ba1f103abb9ae05a4b2891c7ebebaecab0c262535fc6659a628db25df44",
      "size": 25354178
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:4a7307612456a7f65365e1da5c3811df49cefa5a2fd68d8e04e093d26a395d60",
      "size": 603
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:935cecace2a02d2545e0c19bd52fe9c8c728fbab2323fc274e029f5357cda689",
      "size": 893
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:8f46223e4234ce76b244c074e79940b9ee0a01b42050012c8555ebc7ac59469e",
      "size": 666
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:fe0ef4c895f5ea450aca17342e481fada37bf2a1ee85d127a4473216c3f672ea",
      "size": 1394
    }
  ]
}
```

包含了config 和 layers 两部分信息，其中 config 信息为运行镜像的配置，layers 为镜像中的层信息，其中gzip说明镜像的层为gzip压缩格式，每个层一个压缩文件。

### 镜像配置

参考：[OCI Image Configuration](https://github.com/opencontainers/image-spec/blob/main/config.md)

通过 manifest 中的config信息，可以找到镜像的配置信息 blobs/sha256/67e9751bc5aab75bba375f0a24702d70848e5b2bea70de55e50f21ed11feed14：

```
{
  "created": "2022-06-23T04:13:24.820503805Z",
  "architecture": "amd64",
  "os": "linux",
  "config": {
    "ExposedPorts": {
      "80/tcp": {}
    },
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "NGINX_VERSION=1.23.0",
      "NJS_VERSION=0.7.5",
      "PKG_RELEASE=1~bullseye"
    ],
    "Entrypoint": [
      "/docker-entrypoint.sh"
    ],
    "Cmd": [
      "nginx",
      "-g",
      "daemon off;"
    ],
    "Labels": {
      "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
    },
    "StopSignal": "SIGQUIT"
  },
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:08249ce7456a1c0613eafe868aed936a284ed9f1d6144f7d2d08c514974a2af9",
      "sha256:d5b40e80384bb94d01a8d2d8fb2db1328990e7088640132c33d3f691dd8a88ee",
      "sha256:b2f82de68e0d9246de01fa8283876427af5d6f3fe21c4bb04785892d5d071aef",
      "sha256:41451f050aa883f9102df03821485fc2e27611da05689c0ba25f69dcda308988",
      "sha256:44193d3f4ea2bae7a5ae5983f2562f551618b787751a6abfb732b6d17393bb88",
      "sha256:e7344f8a29a34b4861faf6adcf072afb26fadf6096756f0e3fc4c289cdefb7c2"
    ]
  },
  "history": [
    {
      "created": "2022-06-23T00:20:27.020952309Z",
      "created_by": "/bin/sh -c #(nop) ADD file:8adbbab04d6f84cd83b5f4205b89b0acb7ecbf27a1bb2dc181d0a629479039fe in / "
    },
    {
      "created": "2022-06-23T00:20:27.337378745Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"bash\"]",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:05.737870066Z",
      "created_by": "/bin/sh -c #(nop)  LABEL maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:05.834940798Z",
      "created_by": "/bin/sh -c #(nop)  ENV NGINX_VERSION=1.23.0",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:05.931909571Z",
      "created_by": "/bin/sh -c #(nop)  ENV NJS_VERSION=0.7.5",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:06.026686816Z",
      "created_by": "/bin/sh -c #(nop)  ENV PKG_RELEASE=1~bullseye",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:23.901038357Z",
      "created_by": "/bin/sh -c set -x     && addgroup --system --gid 101 nginx     && adduser --system --disabled-login --ingroup nginx --no-create-home --home /nonexistent --gecos \"nginx user\" --shell /bin/false --uid 101 nginx     && apt-get update     && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates     &&     NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62;     found='';     for server in         hkp://keyserver.ubuntu.com:80         pgp.mit.edu     ; do         echo \"Fetching GPG key $NGINX_GPGKEY from $server\";         apt-key adv --keyserver \"$server\" --keyserver-options timeout=10 --recv-keys \"$NGINX_GPGKEY\" && found=yes && break;     done;     test -z \"$found\" && echo >&2 \"error: failed to fetch GPG key $NGINX_GPGKEY\" && exit 1;     apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/*     && dpkgArch=\"$(dpkg --print-architecture)\"     && nginxPackages=\"         nginx=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${PKG_RELEASE}     \"     && case \"$dpkgArch\" in         amd64|arm64)             echo \"deb https://nginx.org/packages/mainline/debian/ bullseye nginx\" >> /etc/apt/sources.list.d/nginx.list             && apt-get update             ;;         *)             echo \"deb-src https://nginx.org/packages/mainline/debian/ bullseye nginx\" >> /etc/apt/sources.list.d/nginx.list                         && tempDir=\"$(mktemp -d)\"             && chmod 777 \"$tempDir\"                         && savedAptMark=\"$(apt-mark showmanual)\"                         && apt-get update             && apt-get build-dep -y $nginxPackages             && (                 cd \"$tempDir\"                 && DEB_BUILD_OPTIONS=\"nocheck parallel=$(nproc)\"                     apt-get source --compile $nginxPackages             )                         && apt-mark showmanual | xargs apt-mark auto > /dev/null             && { [ -z \"$savedAptMark\" ] || apt-mark manual $savedAptMark; }                         && ls -lAFh \"$tempDir\"             && ( cd \"$tempDir\" && dpkg-scanpackages . > Packages )             && grep '^Package: ' \"$tempDir/Packages\"             && echo \"deb [ trusted=yes ] file://$tempDir ./\" > /etc/apt/sources.list.d/temp.list             && apt-get -o Acquire::GzipIndexes=false update             ;;     esac         && apt-get install --no-install-recommends --no-install-suggests -y                         $nginxPackages                         gettext-base                         curl     && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list         && if [ -n \"$tempDir\" ]; then         apt-get purge -y --auto-remove         && rm -rf \"$tempDir\" /etc/apt/sources.list.d/temp.list;     fi     && ln -sf /dev/stdout /var/log/nginx/access.log     && ln -sf /dev/stderr /var/log/nginx/error.log     && mkdir /docker-entrypoint.d"
    },
    {
      "created": "2022-06-23T04:13:24.128160562Z",
      "created_by": "/bin/sh -c #(nop) COPY file:65504f71f5855ca017fb64d502ce873a31b2e0decd75297a8fb0a287f97acf92 in / "
    },
    {
      "created": "2022-06-23T04:13:24.233980553Z",
      "created_by": "/bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b03c4e6c8c513ae014f691fb05d530257dfffd07035c1b75da in /docker-entrypoint.d "
    },
    {
      "created": "2022-06-23T04:13:24.337299368Z",
      "created_by": "/bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7de297435e32af634f29f7132ed0550d342cad9fd20158258 in /docker-entrypoint.d "
    },
    {
      "created": "2022-06-23T04:13:24.441125652Z",
      "created_by": "/bin/sh -c #(nop) COPY file:09a214a3e07c919af2fb2d7c749ccbc446b8c10eb217366e5a65640ee9edcc25 in /docker-entrypoint.d "
    },
    {
      "created": "2022-06-23T04:13:24.534829205Z",
      "created_by": "/bin/sh -c #(nop)  ENTRYPOINT [\"/docker-entrypoint.sh\"]",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:24.627520512Z",
      "created_by": "/bin/sh -c #(nop)  EXPOSE 80",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:24.724935944Z",
      "created_by": "/bin/sh -c #(nop)  STOPSIGNAL SIGQUIT",
      "empty_layer": true
    },
    {
      "created": "2022-06-23T04:13:24.820503805Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"nginx\" \"-g\" \"daemon off;\"]",
      "empty_layer": true
    }
  ]
}
```

其中包含了如下几个关键信息：

1. config：运行镜像的参数，比如entrypoint、labels等，跟通过 docker inspect 命令看到的信息比较类似。
2. rootfs：镜像的层信息
3. history：镜像的历史构建信息，如果empty_layer的值为true，说明未产生新的层

### 镜像层

镜像层同样存在于 blobs/sha256 目录下，且以压缩格式存储，一个层一个压缩文件。manifests文件中的 `application/vnd.oci.image.layer.v1.tar+gzip` 说明镜像层的压缩格式为gzip。

## 容器运行时标准

用来定义容器的配置、运行环境和声明周期。runc为容器运行时的官方实现，其主要代码来源为docker的容器运行时，kara-containers也有对应的OCI实现。

参考文档：[opencontainers/runtime-spec](https://github.com/opencontainers/runtime-spec)

### 容器配置

定义在config.json文件中，定义了创建容器的字段。由于runc更具体的操作系统环境有关，其中部分的规范是跟具体操作系统有关的。执行`runc spec`可以获取到默认的config.json文件，文件内容如下：

```json
{
        "ociVersion": "1.0.2-dev",
        "process": {
                "terminal": true,
                "user": {
                        "uid": 0,
                        "gid": 0
                },
                "args": [
                        "sh"
                ],
                "env": [
                        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                        "TERM=xterm"
                ],
                "cwd": "/",
                "capabilities": {
                        "bounding": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ],
                        "effective": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ],
                        "permitted": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ],
                        "ambient": [
                                "CAP_AUDIT_WRITE",
                                "CAP_KILL",
                                "CAP_NET_BIND_SERVICE"
                        ]
                },
                "rlimits": [
                        {
                                "type": "RLIMIT_NOFILE",
                                "hard": 1024,
                                "soft": 1024
                        }
                ],
                "noNewPrivileges": true
        },
        "root": {
                "path": "rootfs",
                "readonly": true
        },
        "hostname": "runc",
        "mounts": [
                {
                        "destination": "/proc",
                        "type": "proc",
                        "source": "proc"
                },
                {
                        "destination": "/dev",
                        "type": "tmpfs",
                        "source": "tmpfs",
                        "options": [
                                "nosuid",
                                "strictatime",
                                "mode=755",
                                "size=65536k"
                        ]
                },
                {
                        "destination": "/dev/pts",
                        "type": "devpts",
                        "source": "devpts",
                        "options": [
                                "nosuid",
                                "noexec",
                                "newinstance",
                                "ptmxmode=0666",
                                "mode=0620",
                                "gid=5"
                        ]
                },
                {
                        "destination": "/dev/shm",
                        "type": "tmpfs",
                        "source": "shm",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "mode=1777",
                                "size=65536k"
                        ]
                },
                {
                        "destination": "/dev/mqueue",
                        "type": "mqueue",
                        "source": "mqueue",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev"
                        ]
                },
                {
                        "destination": "/sys",
                        "type": "sysfs",
                        "source": "sysfs",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "ro"
                        ]
                },
                {
                        "destination": "/sys/fs/cgroup",
                        "type": "cgroup",
                        "source": "cgroup",
                        "options": [
                                "nosuid",
                                "noexec",
                                "nodev",
                                "relatime",
                                "ro"
                        ]
                }
        ],
        "linux": {
                "resources": {
                        "devices": [
                                {
                                        "allow": false,
                                        "access": "rwm"
                                }
                        ]
                },
                "namespaces": [
                        {
                                "type": "pid"
                        },
                        {
                                "type": "network"
                        },
                        {
                                "type": "ipc"
                        },
                        {
                                "type": "uts"
                        },
                        {
                                "type": "mount"
                        }
                ],
                "maskedPaths": [
                        "/proc/acpi",
                        "/proc/asound",
                        "/proc/kcore",
                        "/proc/keys",
                        "/proc/latency_stats",
                        "/proc/timer_list",
                        "/proc/timer_stats",
                        "/proc/sched_debug",
                        "/sys/firmware",
                        "/proc/scsi"
                ],
                "readonlyPaths": [
                        "/proc/bus",
                        "/proc/fs",
                        "/proc/irq",
                        "/proc/sys",
                        "/proc/sysrq-trigger"
                ]
        }
}
```

### 运行时和生命周期

在config.json文件中可以声明跟容器生命周期相关的部分，比如prestart、poststop等。

定义了很多子命令，比如状态查询的`state <container-id>`，删除容器的`delete <container-id>`等，这些子命令runc部分均有实现。通过 `runc state mycontainerid` 来查看的输出结果如下：

```json
{
  "ociVersion": "1.0.2-dev",
  "id": "mycontainerid",
  "pid": 40805,
  "status": "running",
  "bundle": "/mycontainer",
  "rootfs": "/mycontainer/rootfs",
  "created": "2022-06-29T13:51:54.795617419Z",
  "owner": ""
}
```

## 镜像分发规范

### pull 

#### pull manifest

接口定义：`GET /v2/<name>/manifests/<reference>`

`<name>`： 镜像的 namespace。
`<reference>`：镜像的 tag 或者摘要信息。

#### pull bolb

接口定义：`GET /v2/<name>/blobs/<digest>`

### push

`POST /v2/<name>/blobs/uploads/?digest=<digest>`
`PUT /v2/<name>/manifests/<reference>`

### list tag

接口定义：`GET /v2/<name>/tags/list`

返回格式如下：

```
{
  "name": "<name>",
  "tags": [
    "<tag1>",
    "<tag2>",
    "<tag3>"
  ]
}
```

### list references

接口定义： `GET /v2/<name>/referrers/<digest>`

### delete tag

接口定义： `DELETE /v2/<name>/manifests/<tag>`

### delete manifest

接口定义： `DELETE /v2/<name>/manifests/<digest>`

### delete blobs

接口定义： `DELETE /v2/<name>/blobs/<digest>`

## k8s支持情况

K8s可以通过 pod 的 spec.runtimeClassName 来指定 oci runtime 的实现方式。

## 参考

- [Understanding OCI Image Spec](https://vividcode.io/understanding-oci-image-spec/) （简单明了，推荐优先阅读）
- https://opencontainers.org/
- [云原生制品那些事(2)：OCI 镜像规范](https://mp.weixin.qq.com/s/TVIz8p8nOj4ffYaECW7gVg)
- [skopeo](https://github.com/containers/skopeo)
- [解读 OCI Image Spec](https://ata.alibaba-inc.com/articles/146823?spm=ata.23639746.0.0.7a343566jpFBZQ)
