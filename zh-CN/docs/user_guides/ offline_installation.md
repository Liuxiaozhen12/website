---
id: 离线安装
title: 离线安装
---

此文档描述了如何在离线环境中安装Chaos Mesh。

## 必备条件

在部署Chaos Mesh之前，请确保以下物品已经安装：

- Kubernetes 版本 >= 1.12。
- [RBAC](https://kubernetes.io/docs/admin/authorization/rbac) 已启用 (可选)
- 停靠栏

## 准备安装文件

若要离线安装Chaos Mesh，您需要通过互联网连接获取安装图像。 采取以下步骤：

1. 指定您想要安装的版本：

   ```bash
   导出 CHAOS_MESH_VERSION="v1.1.0"
   ```

   > **注：**
   > 
   > 建议您使用稳定的发布方式。 或者如果您想要体验正在开发中的最新功能，您可以将版本设置为 `最新的`。

2. 存档Chaos Mesh的停泊器图像：

   ```bash #pull images of Chaos Mesh
   Docker prapingcap/chaos-mesh:${CHAOS_MESH_VERSION}
   docker prapingcap/chaos-daemon:${CHAOS_MESH_VERSION}
   docker prapingcap/chaos-dashboard:${CHAOS_MESH_VERSION}
   docker prapingcap/coredns:v0.2.0
   ```

   ```bash #save images of Chaos Mesh to files
   码头保存 -o ./image-chaos-mesh pingcap/chaos-mesh:${CHAOS_MESH_VERSION}
   码头保存 -o . image-chaos-daemon pingcap/chaos-daemon:${CHAOS_MESH_VERSION}
   docker save -o 。 image-chaos-dashboard pingcap/chaos-dashboard:${CHAOS_MESH_VERSION}
   docker save -o ./image-chaos-coredns pingcap/coredns:v0.2.0
   ```

3. 下载Chaos Mesh仓库到您的本地：

   ```bash
   wget "https://github.com/chaos-mesh/chaos-mesh/archive/${CHAOS_MESH_VERSION}.zip"
   ```

   或者您可以下载最新的不稳定版本：

   ```bash
   wget https://github.com/chaos-mesh/chaos-mesh/archive/master.zip
   ```

4. 复制 `./image-chaos-mesh`, `./image-chaos-daemon`, `./image-chaos-dashboard`, 和 `{CHAOS_MESH_VERSION}.zip` 进入离线环境。

## 离线安装 Chaos Mesh

现在您已经有了离线环境中的图像和Repo归档文件，开始安装 Chaos Mesh。

1. 指定您要在离线环境中安装的版本：

   ```bash
   导出 CHAOS_MESH_VERSION="v1.1.0"
   ```

2. 从归档文件加载图像：

   ```bash
   docker load -i ./image-chaos-mesh
   docker load -i ./image-chaos-daemon
   docker load -i ./image-chaos-daemon
   docker load -i ./image-chaos-coredns
   ```

3. 推送Chaos Mesh图像。 您可以选择将他们推送到Docker 注册表或Docker Hub。

   - 将图像推送到基座注册表

     a. 设置 Docker 注册表变量，例如：

     ```bash
     导出 DOCKER_REGISTRY=localhost:5000
     ```

     b. 会议文件。 用 `$DOCKER_REGISTRY标记这些图像`

     ```bash
     导出 CHAOS_MESH_IMAGE=$DOCKER_REGISTRY/pingcap/chaos-mesh:${CHAOS_MESH_VERSION}
     导出 CHAOS_DAEMON_IMAGE=$DOCKER_REGISTRY/pingcap/chaos-daemon:${CHAOS_MESH_VERSION}
     导出 CHAOS_DASHBOARD_IMAGE=$DOCKER_REGISTRY/pingcap/chaos-dashboard:${CHAOS_MESH_VERSION}
     导出 CHAOS_COREDNS_IMAGE=$DOCKER_REGISTRY/ping/cap/coredns:v0。 0
     docker image tag pingcap/chaos-mesh:${CHAOS_MESH_VERSION} $CHAOS_MESH_IMAGE
     docker image tag pingcap/chaos-daemon:${CHAOS_MESH_VERSION} $CHAOS_DAEMON_IMAGE
     docker image tag pingcap/chaos-dashboard:${CHAOS_MESH_VERSION} $CHAOS_DASHBOARD_IMAGE
     docker image tag pingcap/coredns:v0. 0 $CHAOS_COREDNS_IMAGE
     ```

     b. 会议文件。 推送这些图像到 Docker 注册表：

     ```bash
     停泊船推送 $CHAOS_MESH_IMAGE
     停泊船推送 $CHAOS_DAEMON_IMAGE
     停泊船推送 $CHAOS_DASHBOARD_IMAGE
     停泊船推送 $CHAOS_COREDNS_IMAGE
     ```

     > **注：**
     > 
     > 如果Docker 注册表只能在本地工作，您需要在每个K8s节点上加载和推送这些图像。

   - 将图像推送到Docker Hub

     a. 设置 Docker Hub 变量，例如:

     ```bash
     导出 DOCKER_HUB=中心
     ```

     b. 会议文件。 用 `$DOCKER_REGISTRY标记这些图像`

     ```bash
     导出 CHAOS_MESH_IMAGE=$DOCKER_HUB/chaos-mesh:${CHAOS_MESH_VERSION}
     导出 CHAOS_DAEMON_IMAGE=$DOCKER_HUB/chaos-daemon:${CHAOS_MESH_VERSION}
     导出 CHAOS_DASHBOARD_IMAGE=$DOCKER_HUB/chaos-dashboard:${CHAOS_MESH_VERSION}
     导出 CHAOS_COREDNS_IMAGE=$DOCKER_HUB/coredns:v0.2.
     docker image tag pingcap/chaos-mesh:${CHAOS_MESH_VERSION} $CHAOS_MESH_IMAGE
     docker image tag pingcap/chaos-daemon:${CHAOS_MESH_VERSION} $CHAOS_DAEMON_IMAGE
     docker image tag pingcap/chaos-dashboard:${CHAOS_MESH_VERSION} $CHAOS_DASHBOARD_IMAGE
     docker image tag pingcap/coredns:v0. 0 $CHAOS_COREDNS_IMAGE
     ```

     b. 会议文件。 推送这些图像到 Docker 注册表：

     ```bash
     停泊船推送 $CHAOS_MESH_IMAGE
     停泊船推送 $CHAOS_DAEMON_IMAGE
     停泊船推送 $CHAOS_DASHBOARD_IMAGE
     停泊船推送 $CHAOS_COREDNS_IMAGE
     ```

4. 通过以下步骤安装Chaos Mesh 离线：

   a. 解压缩仓库文件到路径：

   ```bash
   unzip ${CHAOS_MESH_VERSION}.zip -d chaos-mesh && cd chaos-mesh/*/
   ```

   b. 会议文件。 创建一个用于安装Chaos Mesh的命名空间：

   ```bash
   kubectl 创建命名空间chaos-测试
   ```

   b. 会议文件。 用头盔安装Chaos Mesh

   ```bash
   helm install chaos-mesh helm/chaos-mesh --namespace=chaos-test。
      --set 仪表板 reate=true \
      --set dnsServer.create=true \
      --set chaosDaemon mage=$CHAOS_DAEMON_IMAGE \
      --set ControllerManager.image=$CHAOS_MESH_IMAGE \
      --set 仪表板 mage=$CHAOS_DASHBOARD_IMAGE \
      --set dnsServer.image=${CHAOS_COREDNS_IMAGE}
   ```

   b. 平均输出功率超过1瓦； 检查Chaos Mesh pod是否安装：

   ```bash #get pods of Chaos Mesh
   kubectl 获取 pods --namespace chaos-test-l app.kubernetes.io/instance=chaos-mesh
   ```

   预期输出：

   ```bash
   命名重命名STATUS RESTARTS AGE
   chaos-controller-manager-6d6d95cd94kl8gs 1/1 Running 0 3m40s
   chaos-daemon-5shkv 1/1 Running 0 3m40s
   chaos-d98856f6-vgrjs 1/1 Running 0 3m40s
   ```

   在执行上述命令后，您应该能够看到所有Chaos Mesh pods都已上线并运行的输出。 Otherwise, check the current environment according to the prompt message or create an [issue](https://github.com/chaos-mesh/chaos-mesh/issues) for help.
