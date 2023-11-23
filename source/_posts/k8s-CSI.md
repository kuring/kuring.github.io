title: k8s CSI
date: 2023-11-23 22:56:40
tags:
author:
---
[CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md)：Container Storage Interface (CSI)

Volume 的三个阶段：
1. Provision and Delete：负责卷的创建以及销毁。
2. Attaching and Detaching：将卷设备挂载到本地或者从本地卸载。
3. Mount and Umount：将 Attaching 的块设备以目录形式挂载到 pod中，或者从 pod 中卸载块设备。

# 开发
CSI 插件分为三个部分：

1. CSI Identity：用来获取 CSI 的身份信息
2. CSI Controller
3. CSI Node

参考 k8s 官方的 hostpath 项目：[https://github.com/kubernetes-csi/csi-driver-host-path](https://github.com/kubernetes-csi/csi-driver-host-path)

为了方便开发，在每个阶段 k8s 官方均实现了对应的 SideCarSet 容器。要想研发，仅需要实现 grpc server，又 SideCarSet 容器调用自研的容器。

自研的容器需要实现如下的接口即可。

## CSI Identity
```protobuf
service Identity {
  // 返回插件名字以及版本号
  rpc GetPluginInfo(GetPluginInfoRequest) returns (GetPluginInfoResponse) {}

  // 返回插件的包含的功能
  rpc GetPluginCapabilities(GetPluginCapabilitiesRequest) returns (GetPluginCapabilitiesResponse) {}

  rpc Probe (ProbeRequest)
    returns (ProbeResponse) {}
}
```
External provisioner 会调用该接口。

## CSI Controller
实现 Volume 中的 Provisioning and Deleting 和 Attaching and Detaching 两个阶段的功能。只有块存储 CSI 插件才需要 Attach 功能。
该部分以中心化组件的方式部署，比如 Deployment。Provision 功能对应的 SidecarSet 为 [external-provisioner](https://github.com/kubernetes-csi/external-provisioner)，Attach 对应的 SidecarSet 为 [external-attacher](https://github.com/kubernetes-csi/external-attacher)。
```protobuf
service Controller {
  // Provisioning， External provisioner调用
  // hostPath 实现中会调用 fallocate 实现
  rpc CreateVolume (CreateVolumeRequest)
    returns (CreateVolumeResponse) {}

  // Deleting， External provisioner调用
  rpc DeleteVolume (DeleteVolumeRequest)
    returns (DeleteVolumeResponse) {}

  // Attaching, 只有块设备才需要实现，比如云盘，由External attach 调用
  rpc ControllerPublishVolume (ControllerPublishVolumeRequest)
    returns (ControllerPublishVolumeResponse) {}

  // Detaching，External attach 调用
  rpc ControllerUnpublishVolume (ControllerUnpublishVolumeRequest)
    returns (ControllerUnpublishVolumeResponse) {}

  rpc ValidateVolumeCapabilities (ValidateVolumeCapabilitiesRequest)
    returns (ValidateVolumeCapabilitiesResponse) {}

  rpc ListVolumes (ListVolumesRequest)
    returns (ListVolumesResponse) {}

  rpc GetCapacity (GetCapacityRequest)
    returns (GetCapacityResponse) {}

  rpc ControllerGetCapabilities (ControllerGetCapabilitiesRequest)
    returns (ControllerGetCapabilitiesResponse) {}

  // 创建快照功能
  rpc CreateSnapshot (CreateSnapshotRequest)
    returns (CreateSnapshotResponse) {}

  // 删除快照功能
  rpc DeleteSnapshot (DeleteSnapshotRequest)
    returns (DeleteSnapshotResponse) {}

  rpc ListSnapshots (ListSnapshotsRequest)
    returns (ListSnapshotsResponse) {}

  rpc ControllerExpandVolume (ControllerExpandVolumeRequest)
    returns (ControllerExpandVolumeResponse) {}

  rpc ControllerGetVolume (ControllerGetVolumeRequest)
    returns (ControllerGetVolumeResponse) {
        option (alpha_method) = true;
    }

  rpc ControllerModifyVolume (ControllerModifyVolumeRequest)
    returns (ControllerModifyVolumeResponse) {
        option (alpha_method) = true;
    }
}
```

## CSI Node
实现 Volume 中的Mount 和 Umount 阶段，由 kubelet 负责调用。
该部分以 DaemonSet 的形式部署在每个 k8s node 上，对应的 SidecarSet 容器为 [node-driver-registrer](https://github.com/kubernetes-csi/node-driver-registrar)。
```protobuf
service Node {
  // 针对块存储类型，将块设备格式化后先挂载到一个临时全局目录
  rpc NodeStageVolume (NodeStageVolumeRequest)
    returns (NodeStageVolumeResponse) {}

  rpc NodeUnstageVolume (NodeUnstageVolumeRequest)
    returns (NodeUnstageVolumeResponse) {}

  // 如果是块存储设备，在执行完 NodeStageVolume 后，使用 linux 的 bind mount 技术将全局目录挂载到pod 中的对应目录
  rpc NodePublishVolume (NodePublishVolumeRequest)
    returns (NodePublishVolumeResponse) {}

  rpc NodeUnpublishVolume (NodeUnpublishVolumeRequest)
    returns (NodeUnpublishVolumeResponse) {}

  rpc NodeGetVolumeStats (NodeGetVolumeStatsRequest)
    returns (NodeGetVolumeStatsResponse) {}


  rpc NodeExpandVolume(NodeExpandVolumeRequest)
    returns (NodeExpandVolumeResponse) {}


  rpc NodeGetCapabilities (NodeGetCapabilitiesRequest)
    returns (NodeGetCapabilitiesResponse) {}

  rpc NodeGetInfo (NodeGetInfoRequest)
    returns (NodeGetInfoResponse) {}
}
```

# 资料
协议：[https://github.com/container-storage-interface/spec/blob/master/spec.md](https://github.com/container-storage-interface/spec/blob/master/spec.md)