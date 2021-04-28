<!--
 * @Author: jhliang
 * @Date: 2020-11-27 16:55:03
 * @LastEditTime: 2020-11-27 16:58:32
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \MyBook\Kubernetes\Kubernetes对象.md
-->

# Kubernetes 对象

## 理解 Kubernetes 对象

在 Kubernetes 系统中，Kubernetes 对象 是持久化的实体。 Kubernetes 使用这些实体去表示整个集群的状态。特别地，它们描述了如下信息：

- 哪些容器化应用在运行（以及在哪些节点上）
- 可以被应用使用的资源
- 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略

Kubernetes 对象是 “目标性记录” —— 一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。 通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的， 这就是 Kubernetes 集群的 **期望状态（Desired State）**。

操作 Kubernetes 对象 —— 无论是创建、修改，或者删除 —— 需要使用 Kubernetes API。 比如，当使用 kubectl 命令行接口时，CLI 会执行必要的 Kubernetes API 调用， 也可以在程序中使用 客户端库直接调用 Kubernetes API。

## 对象规约（Spec）与状态（Status）

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置： 对象 *spec*（规约） 和 对象 *status*（状态） 。 对于具有 spec 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征：**期望状态（Desired State）** 。

status 描述了对象的 当前状态（Current State），它是由 Kubernetes 系统和组件 设置并更新的。在任何时刻，Kubernetes 控制平面 都一直积极地管理着对象的实际状态，以使之与期望状态相匹配。
