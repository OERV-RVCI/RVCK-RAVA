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

```mermaid
flowchart TB
    subgraph requests
    pr["pull_request_target"]
    issue["issue"]
    comment["issue_comment"]
    other_req["..."]
    end

    subgraph actions
    direction TB
    rava-actions["`rava-actions
    接收、解析请求
    `"]
    end
    
    pr --> rava-actions
    issue --> rava-actions
    comment --> rava-actions
    other_req --> rava-actions

    subgraph funcs
    direction TB
    
    codelint
    
    lava-trigger
    
    update-status1["update-status
    if: 非pr请求
    "]

    update-status2["update-status
    更新结果:
    pr 新增一条评论
    issue|comment 在原消息后面追加
    "]
    
    codelint --> update-status2
    lava-trigger --> update-status2
    update-status2 --> End([end])

    end
    
    rava-actions --> lava-trigger
    rava-actions --> codelint
    rava-actions --> update-status1
```

## rvck actions


```mermaid
flowchart TB
    subgraph requests
    pr["pull_request_target"]
    issue["issue"]
    comment["issue_comment"]
    other_req["..."]
    end

    subgraph actions
    direction TB
    rvck-actions["rvck-actions
    接收请求
    "]
    end
    
    pr --> rvck-actions
    issue --> rvck-actions
    comment --> rvck-actions
    other_req --> rvck-actions

    subgraph funcs
    direction TB
    
    parse-rvck["parse-rvck
    解析请求，传递参数到子任务
    "] --> check-patch["check-patch
    if: pr请求
    "]
    
    parse-rvck --> kunit-test

    parse-rvck --> kernel-build --构建成功的内核用于lava--> lava-trigger
    
    
    parse-rvck --> update-status1["update-status
    if: 非pr请求
    "]

    update-status2["update-status
    更新结果:
    pr 新增一条评论
    issue|comment 在原消息后面追加
    "]
    
    check-patch --> update-status2
    kunit-test --> update-status2
    lava-trigger --> update-status2
    update-status2 --> End([end])

    end
    
    rvck-actions --> parse-rvck
    
```

## 详细实现

### lava-trigger

```mermaid
flowchart TB
  A[接收请求] --> B[拉取 lava 代码] --> C[填写 template 文件] --> D[提交lava请求]
  subgraph lava-result
  
  subgraph wait1
    AA{{判断 cache标志位文件 是否存在}} --存在--> BB([end])
    AA --> CC["lavacli wait
    等待lava执行结果
    "] --> DD[lava是否返回结果] --是--> EE[设置cache文件作为标志位] --> BB
    DD --否 --> BB
  end
  subgraph wait2
    W2[...]
  end
  subgraph wait3
    W3[...]
  end
  subgraph waitN
    WN[...]
  end

  wait1 -.-> wait2 -.-> wait3 -."...".-> waitN
  
  end

  
  D --> lava-result  --> E[解析结果并输出] --> F([end])
  
  D -.-> on-cancel["on-cancel
  当任务中途推出时, 取消lava任务
  "] -.-> F
  
  lava-result -.-> on-cancel
```