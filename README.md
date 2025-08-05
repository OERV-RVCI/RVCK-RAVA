# OERV-RVCI workflow

## 工作流简介

工作流做简单分层

- `funcs` 为若干可复用工作流。传入指定参数后，即可执行，与具体仓库解耦
- `actions` 作为具体检查工作流入口(如`rvck, lava ...`), 负责解析请求任务，组织和调用`func`中可复用工作流完成检查。

```mermaid
flowchart TB
    subgraph actions
    direction TB
    rava-actions["`**rava-actions.yml**
    rava 工作流入口
    接受请求，组织和调用其余可复用工作流完成检查
    `"]
    rvck-actions["`**rvck-actions.yml**
    rvck 工作流入口
    接受请求，组织和调用其余可复用工作流完成检查
    `"]
    docker-publish["`**docker-publish.yml**
    构建所有任务通用docker镜像并发布
    `"]
    end
    subgraph funcs
    direction TB
    check-patch["`**check-patch.yml**
    检查pr请求patch
    `"]
    codelint["`**codelint.yml**
    代码规范检查: shellcheck, yamllint
    `"]
    kernel-build["`**kernel-build.yml**
    内核构建
    `"]
    kunit-test["`**kunit-test.yml**
    kunit 测试
    `"]
    lava-trigger["`**lava-trigger.yml**
    触发远程lava构建
    等待结果返回
    解析lava结果
    `"]
    parse-rvck["`**parse-rvck.yml**
    解析rvck请求参数
    `"]
    update-status["`**update-status.yml**
    更新任务状态信息
    issue或comment 在原来消息中追加内容
    pr 请求新建一条评论，展示消息
    `"]
    end
    rvck-actions --- parse-rvck
    rvck-actions --- kunit-test
    rvck-actions --- kernel-build
    rvck-actions --- check-patch
    rvck-actions --- lava-trigger
    rvck-actions --- update-status

    rava-actions --- lava-trigger
    rava-actions --- update-status
    rava-actions --- codelint 
```

## rava actions

