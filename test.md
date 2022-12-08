# PR的review及合入过程介绍

## 背景

此文用于帮助PR作者和代码审查者熟悉整个PR合入流程和注意事项，提高PR合入效率。

## 实现原理

gitee或其他代码托管平台可配置各种事件的webhook回调，事件包括push、PR、issue、评论等。
机器人系统监听PR评论的回调，根据评论者和评论内容（命令），执行不同的操作（修改PR标签）。


## 命令说明

社区的成员角色分为3类，分别是一般贡献者、reviewer 和 approver，他们拥有不同的权力，并通过不同的命令表达其对PR的检视意见。

一般贡献者的检视意见无法通过命令来作用于PR，换句话说，其无法影响PR的合入；只有reviewer和approver的指令可以影响PR的合入。

approver拥有指定模块（目录）或全部模块的合入权限， reviewer无合入权限。

reviewer和approver通常配置在代码库的owners文件中，身份的取得请咨询代码库维护者。

| 命令                       | 描述                         | 谁能有效使用                                 | 使用场景           |
| :------------------------- | :--------------------------- | ---------------------------------------- | ---  
|/can-review                 | 代码可以开始被review         | PR作者                                   | 质量检视前|
| /lgtm --- looks good to me | 认可代码                     | reviewer或approver                  | 代码的质量检视    |
| /lbtm --- looks bad to me  | 不认可代码，希望修改代码     | reviewer或approver                                    | 代码的质量检视     |
| /approve                   | 认可代码                     | approver，根据配置决定是否包含PR作者自己     | 合入代码的指令     |
| /reject                    | 代码需要做重大修改           | approver，但不包括PR作者                 | 拒绝代码合入的指令 |

注意:
- 评论命令不区分大小写
- 每位reviewer/approver评论多条命令时取最新的一条
- 任何一位approver评论了/reject，PR流程将会停止。需要PR的作者与该approver沟通，让其取消reject（评论/lgtm，/lbtm，/approve），否则该PR将无法合入
- 不是说一定要等作者执行/can-review命令后，其他人才可以来review。/can-review命令只是打上can-review标签，机器人提示并引导PR作者和reviewer/approver完成整个PR合入过程。reviewer/approver可跳过这个过程随时来review代码，并执行相应的命令

## 标签说明

标签功能是平台提供的用于标识当前PR包含的内容或状态，代码库维护者可自定义，可随时手动修改。通常由机器人自动修改标签。


| 标签                       | 描述                         | 前置条件                                | 存续说明            |
| :------------------------- | :--------------------------- | ---------------------------------------- |--- 
|ci-pipeline-passed          | ci标签                       | 系统自动对代码进行ci检查，需要一定时间   |通过后一直存在
|cla|cla标签|系统自动对代码进行cla检查，需要一定时间 | 通过后一直存在
|can-review|可以开始review的的标记（象征意义）|同时具备ci和cla标签，且PR作者评论/can-review命令后|与lgtm、approved、reqest-change互斥，不会同时存在
|lgtm                        | 来自reviewer的认可 | 一定数量的reviewer评论了/lgtm命令（数量据配置而定）|可与approved同时存在，与can-reivew和request-change互斥         
|approved                    | 来自approver的认可    |  所有PR涉及的目录都被approved，且一定数量的approver评论了/approve命令（数量据配置而定），两者条件缺一不可   |可与lgtm同时存在，与can-reivew和request-change互斥    
|request-change | 代码不被认可，需要作者重新修改|approver评论了/reject命令，或者评论/lbtm的reviewer人数大于评论了/lgtm与/approve的人数之和|与can-review、lgtm、approved互斥，不会同时存在

注意：
- 不同标签轮转时机器人会在评论给出相应的提示，用户可参考执行
- PR可被合入的条件是同时具备lgtm和approved标签
- 机器人无法进行合入操作，只是给出可合入的提示，仍由维护者进行合入操作
- PR作者一旦在PR分支提交新的修改，所有标签都有被删除，需重新走整个流程



**一个PR顺利合入的流程图：**

```
flowchart TD
    start(PR提交) -->B[ci/cla标签]-->C[can-review标签]
    C-->E[LGTM标签]
    C-->F[APPROVED标签]
    E-->H[同时具备]
    F-->H
    H-->I(可合入)
```

