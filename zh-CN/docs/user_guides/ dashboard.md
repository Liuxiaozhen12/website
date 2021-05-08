---
id: 仪表板
title: Chaos 仪表板
---

Chaos 仪表板是用于管理、设计和监测Chaos Mesh上的混乱实验的一个步骤的 Web UI。 本文件对如何使用Chaos 仪表板作了逐步介绍。

## 安装Chaos 仪表板

Chaos 仪表盘默认在 v1.2.0后安装。 您可以运行以下命令来检查Chaos 仪表板Pod的状态：

```bash
# chaos-testing: 安装Chaos Mesh
kubectl 获取pod-n chaos-test-l app.kubernetes.io/component=chaos-dashboard 的默认命名空间
```

预期输出：

```bash
chaos-dashboard-b8767fbcd-46cnd 1/1 运行 031m
```

如果您没有得到Chaos仪表盘，您可以通过执行来添加它：

```bash
helm upgrade chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-testing --set dashboard.create=true
```

## 启用/禁用安全模式

Chaos 仪表盘支持安全模式，它要求用户使用 Kubernetes 生成的令牌登录。 每个令牌都连接到 `服务账户`。 您只能在与服务帐户关联的角色允许的范围内执行操作。

如果您通过 Helm 安装，默认情况下将启用安全模式。 您可以通过执行来禁用它：

```bash
helm upgrade chaos-mesh chaos-mesh/chaos-mesh --namespace=chaos-test-set dashboard.securityMode=false
```

**注：**

> - 对于实际测试场景，我们强烈建议您启用安全模式。
> - 如果您通过 `install.sh`来安装Chaos Mesh，安全模式将被禁用，适合尝试Chaos Mesh。

## 访问Chaos仪表板

访问Chaos 仪表板的典型方式是使用 `kubectl port-forward`:

```bash
kubectl port-forward - n chaos-testing svc/chaos-ashboard 2333:2333
```

现在您可以在浏览器中访问 [`http://localhost:2333`](http://localhost:2333)

## 登录

默认情况下，当使用头盔安装Chaos Mesh时，安全模式将被启用 并且您需要登录Chaos 仪表盘并带有帐户 `name` 和 `Token`。 本节介绍如何创建帐户和代币。 如果您已禁用安全模式，您可以跳过此步骤。

### 创建帐户

Chaos 仪表板支持基于角色的访问控制([RBAC](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/))。

**注：**

> - 如果你想稍后创建帐户并快速开始使用Chaos 仪表板，你可以使用Chaos Mesh的令牌。 通过执行命令获取令牌： `kubectl -n chaos-test描述密钥 $(kubectl -n chaos-testing get secret | grep chaos-controller-manager | awk '{print $1}')`然后您可以直接前往 [填充](#fill-in)

要创建帐户：

1. 创建 `角色`。 这里是角色配置样本，您可以从中选择并编辑以满足您的特定要求。 您需要将配置保存到 YAML 文件，然后使用 `kubectl 应用程序 -f {YAML-File}` 来创建它。

   - **集群管理器**

     这个角色拥有对Kubernetes集群中所有命名空间下的chaos实验的管理权限，包括创建、更新、归档和查看chaos实验。

     示例配置文件：

     ```yaml
     kind: ClusterRole
     apiVersion: rbac.authorization.k8s. o/v1
     元数据：
       名称：集群-角色管理器
     规则：
       - apiGroups：['']
         资源：['pods' '命名空间']
         动词: ['get', 'list', '观察']
       - apiGroups：
           - chaos-mesh。 rg
         资源：['*']
         动词：
           - get
           - list
           - watch
           - 创建
           - 更新
           - 补丁
           - 删除
     ```

   - **集群查看器**

     这个角色允许查看Kubernetes集群中所有命名空间下的混乱试验。

     示例配置文件：

     ```yaml
     kind: ClusterRole
     apiVersion: rbac.authorization.k8s. o/v1
     元数据：
       名称：集群角色查看器
     规则：
       - apiGroups：['']
         资源：['pods' '命名空间']
         动词: ['get', 'list', '观察']
       - apiGroups：
           - chaos-mesh。 rg
         资源：['*']
         动词：
           - get
           - 列表
           - 观察
     ```

   - **命名空间管理器**

     这个角色拥有在 Kubernetes 集群中指定命名空间下的 chaos 实验的行政权限，包括创建、更新、归档和查看chaos 实验。

     示例配置文件如下：

     ```yaml
     kind: 角色
     apiVersion: rbac.authorization.k8s. o/v1
     元数据：
       名称：namespace-test-role-manager
       namespace: test
     rules:
       - apiGroups: ['']
         资源：['pods', '命名空间']
         动词: ['get', 'list', '观察']
       - apiGroups：
           - chaos-mesh。 rg
         资源：['*']
         动词：
           - get
           - list
           - watch
           - 创建
           - 更新
           - 补丁
           - 删除
     ```

   - **命名空间查看器**

     这个角色可以访问Kubernetes集群指定命名空间下的混乱试验。

     示例配置文件：

     ```yaml
     kind: 角色
     apiVersion: rbac.authorization.k8s. o/v1
     元数据：
       名称：namespace-test-role-viewer
       namespace: test
     rules:
       - apiGroups: ['']
         资源：['pods', '命名空间']
         动词: ['get', 'list', '观察']
       - apiGroups：
           - chaos-mesh。 rg
         资源：['*']
         动词：
           - get
           - 列表
           - 观察
     ```

2. 创建一个 `服务帐户`, 并将其绑定到一个特定的 `角色`。 更多详情请参阅 [RBAC](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/)。 这里是您可以从中选择和编辑的样本配置来满足您的特定要求。 您需要将配置保存到 YAML 文件，然后使用 `kubectl 应用程序 -f {YAML-File}` 来创建它。

   - **创建服务帐户并绑定集群角色**

     创建一个服务帐户并将其绑定为 `集群管理器` 或 `集群查看器`。

     示例配置文件：

     ```yaml
     kind: ServiceAccount
     apiVersion: v1
     metetdate:
       name: account-cluster-manager # 使用 "account-cluster-viewer" for viewer
       namespace: chaos-testing

     -
     apiVersion: rbac. k8 s。 o/v1
     种类型：ClusterRoleBinding
     元数据：
       名称：bind-cluster-manager # 为查看器使用 "bind-cluster-viewer"
     主题：
       - 类型：ServiceAccount
         name: account-cluster-manager # 使用 "account-cluster-viewer" 为查看器
         namespace: chaos-testation
     roleRef：
       kind: ClusterRole
       name: cluster-role-manager # 使用 "cluster-role-viewer"
       apiGroup: rbac. k8s.io
     ```

   - **创建服务帐户并绑定命名空间角色**

     创建一个服务帐户并绑定它的角色 `命名空间管理器` 或 `命名空间查看器`。

     示例配置文件：

     ```yaml
     类型: ServiceAccount
     apiVersion: v1
     metadata:
       name: acent-namespace-test-manager # 为查看器使用 "account-namespace-test-viewer"
       namespace: test

     -
     apiVersion: rbac. k8 s。 o/v1
     种类型：RoleBinding
     元数据：
       名称：bind-namespace-test-manager # 为查看器使用 "bind-namespace-test-viewer"
       namespace: test
     subjects:
       - kind: ServiceAccount
         namespace-test-manager # 为查看器使用 "account-namespace-test-viewer"
         namespace: test
     roleRef:
       kind: Role
       namespace-test-role-manager # 为查看器使用 "namespace-test-role-viewer"
       apiGroup: rback k8s.io
     ```

### 获取令牌

代币由 Kubernetes 生成。 要获取上述 `服务账户` 的令牌，请运行下面的命令：

```bash
kubectl -n ${namespace} 描述密钥 $(kubectl -n ${namespace} 获得密钥 | grep ${service-account-name} | awk '{print $1}')
```

**注：**

> - 您需要将 `${namespace}` 和 `${service-account-name}` 替换为实际值。 例如， 执行命令 `kubectl -n chaos-test描述密钥 $(kubectl -n chaos-testing get secret | grep cluster-role-manager | awk '{print $1}')` 获取 `cluster-role-manager`

更多详情请参阅 [geting-a-bearer-token](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md#getting-a-bearer-token)。

### 填充于

用生成的令牌，你需要用 `名称` 来识别它。 推荐一个有意义的名称，例如 `Cluster-Manager`，以表明令牌在集群中有权限管理的混乱试验。

用 `名称` 和 `令牌` 填充表单：

![仪表板登录](/img/user_guides/dashboard-login.png)

## 创建一个混乱测试

要在 Chaos 仪表盘上创建一个混乱实验：

1. 点击 **新专家** 按钮来创建一个新的混乱模式实验：

   ![仪表板-新测试](/img/user_guides/dashboard-new-exp.png)

2. 配置chaos实验，包括实验类型、名称、调度信息等。

   ![仪表板填充测试](/img/user_guides/dashboard-fill-exp.png)

## 管理一个混乱测试

要管理特定的混乱模式实验：

1. 点击 **实验** 按钮查看所有混乱试验。

   ![仪表板测试](/img/user_guides/dashboard-experiments.png)

2. 选择目标实验以查看细节、存档、暂停或更新。

   ![仪表板实验细节](/img/user_guides/dashboard-exp-detail.png)

## 快速查看

通过刚才采取的步骤，你已经知道如何创建一个实验并查看它的细节。 但这只是仪表板的主要特征之一。

接下来，您可以点击主页上的 **TUTORIAL** 按钮来了解仪表板的所有功能。

![仪表板-主页](/img/user_guides/dashboard-home.png)

## 管理现有令牌

创建和管理现有令牌：

![仪表板设置](/img/user_guides/dashboard-settings.png)

**注：**

> 如果设置了 `仪表板.securityMode=false` ，此部分将不可见。
