<!--
 * @Author: jhliang
 * @Descripttion: In User Settings Edit
 * @LastEditTime: 2021-04-28 11:45:14
 * @Date: 2021-04-28 10:47:49
 * @LastEditors: jhliang
 * @FilePath: \MyBook\Yearning\Yearning二次开发分析.md
-->

# 二次开发分析

## 回滚

binlog回滚不适合二次开发，Yearnig把关于回滚的代码全部闭源封装了，根本不能做二次开发。

## 检测语句

检测的功能同样封装在了Juno里面，并不能二次开发，详见/handle/fetch/fetch.go FetchSQLTest()

## 检测规则

检测规则也是封装在JUNO中，不能做二次开发

## DML和DDL混合

想要支持DML和DDL混合脚本的支持，也不能勇Juno做二次开发，Yearning的执行工单的sql语句的方法主要是src/handle/order/audit/executor.go Executor()，该方法的语句执行也是直接调用的Juno，并不能修改。

## 检测sql语句功能

检测sql语句功能也是封装在Juno里面，开源的部分只是调用了他的接口，不能按照我们的需求进行二次开发。详见src/handle/fetch/fetch.go FetchSQLTest()
