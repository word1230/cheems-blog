# Claude Code学习


<!--more-->
官方文档: https://code.claude.com/docs/zh-CN/best-practices
https://code.claude.com/docs/zh-CN/sub-agents
练习方式:
claude code(下文简称 cc) 设计练习

## 1 第一阶段: 认识界面与基础对话
### 1.1 打开claude code 熟悉一下快捷键
打开终端,输入下面的命令
```shell
mkdir cc-practice      # 创建练习文件夹
cd cc-practice       
claude  # 启动claude code
```
在打开的claud code 对话框中尝试一下这几个命令:
- `/help`          
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326095148605.png)
<br/>
- `?`
唤出快捷键列表
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326095213690.png)
<br/>
- `/clear` 
这是一个命令, 清除对话历史并且清空上下文. 相当于在当前文件夹重开了一个对话
与之相对的是 `/compact  你想要让模型总结的侧重点`   清空对话历史, 总结之前的聊天记录. 


>介绍一下这些快捷键

- `!` 可以进入bash 模式(你打开终端,默认就是这个模式,可以正常执行终端命令)
- `双击esc` 清除当前输入. ==当你输入错误,想要删掉已输入的内容时可以用这个== 当然也可以`ctrl + c` 来清除
- `/` 进入命令模式, 可以执行`help`命令里的第二栏与第三栏的命令
- `alt + m` 给模型开放自动编辑文件的权限. 避免每次确认
- `alt + v` 粘贴图片.  你==直接ctrl +v 是没法粘贴图像==的.要用这个
- `@` 提供文件路径. 也就是引用文件
- `ctrl + o` 一般模型的输出都是收起的,你想要查看完整的输出,可以用这个 
- `alt + p` 切模型
- `btw question` 在模型执行任务时,可以通过这个命令问一些其他问题,不会打断主流程.但是这个模型没有编辑能力,只能回答
- `ctrl + w `或者 `\ + enter` 在输入框换行. 类似于其他输入框的`shift +enter`

<br/>


### 1.2 1.2进行初次对话
然后你可以输入你的第一句话: `你好，请介绍一下你能帮我做什么？`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326101136365.png)


## 2 阶段二:初始化一个项目
### 2.1 让cc 帮你初始git仓库 

输入框 输入: 
`帮我初始化git仓库，创建一个合适的 .gitignore 文件（nodejs 项目）`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326101651502.png)
这个过程中会要求你给予编辑文件的权限.
使用`alt + m` 切换到自动确认编辑的模式, 可以给予模型在当前对话中,拥有所有的文件编辑权限,避免反复手动确认

执行完后可以看到
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326102022265.png)


### 2.2 创建 CLAUDE.md项目规范文件
可以使用`/init` 命令,它会系统扫描整个代码库,自动识别项目类型,生成CLAUDE.md
也可以手动让他生成, 输入: "帮我分析这个项目，生成一个 CLAUDE.md 文件"
<br/>
一开始可以使用`/init`, 后续需要补充,可以手动进行
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326102418050.png)


验证：检查根目录是否生成了 CLAUDE.md
让 Claude 复述规则：
  请告诉我 CLAUDE.md 里的项目规范是什么？

## 3 阶段三: 构建一个项目
体验 Claude Code 的完整开发流程（规划 → 编码 → 测试 → 提交）。
这里我们做一个 todo -cli , 命令行todo工具
### 3.1 使用plan mode 规划任务并进行编码
首先: 按`alt + m` 切换到plan mode
输入框输入:
```text
创建一个todo-cli工具,要求:
- 包含: add,list,remove,done功能
- 只能使用node.js 内置模块,不允许使用任何第三方库
- 文件编码统一使用 UTF-8。
```
<br/>
在这个模式下,cc会先给出计划,询问你是否可以执行/有没有要补充的
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326103253203.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326103323113.png)

第一个选项是 进入自动确认编辑的模式, 自动执行
第二个是 手动进行批准
第三个:补充一些缺失的约束/流程



### 3.2 测试你的程序
这一步其实在模型的计划里是有的.模型在编码后,已经进行测试了


当然我们也可以手动进行: 
根据claude 官方的提示: 当 Claude 能够验证自己的工作时，例如运行测试、比较屏幕截图和验证输出，它的表现会显著提高。
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326211908779.png)



比如输入: `执行几组测试,如果遇到执行错误或者与预期不符的bug,帮我一起修复了` 
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326104107734.png)


### 3.3 让cc提交代码(git 工作流)
让cc 创建一个分支,然后提交代码
`按照 CLAUDE.md 的规范，创建一个新分支，将当前代码提交并写一个合适的提交信息`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326104212824.png)



## 4 阶段四: 批处理

### 4.1 管道命令
首先看一下cc的单次执行:
 - cc的单次执行: 打开终端后,直接输入 `claude -p "任务"`  不需要打开cc界面
 - 可以让cc执行一次任务,而不是连续对话
接下来再讲管道:
- 我们在终端可以 通过管道将一个命令的返回值传递给另一个命令
- 比如: 先获取文件内容然后进行查找,`cat 文件名 | grep "要查找的内容"`
这里我们也可以通过管道将信息传给cc ,配合cc的单次执行来完成一些任务

#### 4.1.1 用法:进行code review

> 比如,我们可以: 将`git diff` 命令返回的信息通过管道传给cc,让他进行code review

直接在终端输入:
`git diff main --name-only | claude -p "审查这些被修改的文件， 找出潜在的安全漏洞和代码质量问题，给出详细报告"`

![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326110710597.png)
这个与在GitHub Actions上配置code review工作流类似

我们还可以管道叠加管道, 这里是codereview完成了, 我们可以把结果通过管道再传给cc,让它直接改


当然也可以直接输入:
`对比 main 分支和当前分支的改动，帮我做一次代码审查，找出潜在问题`  也能完成
#### 4.1.2 分析线上日志

当线上报500时, 我们可以将日志下载到本地,然后通过管道传给cc 
跟上面的用法差不多

### 4.2 创建自定义slash 命令
我们有一些常用的流程,可以将其创建成一个命令,需要执行时,直接执行这个命令即可. 不需要重复输入提示词

步骤:
- 在项目里新建目录和文件夹
	- `mkdir .claude\commands`
- cc中输入: 帮我创建一个自定义 slash 命令 /run-tests，功能是运行nodetest 并把结果汇总
- 然后 输入 /run-tests 即可执行



## 5 阶段五: 高效提示 + git回滚 + 子agent 

这里我们练习一下: 不是从零开始,而是接手一个陌生项目,要如何做

这里我就选择 cherry me 佬的grok2api 
### 5.1 先探索-再规划- 再编码 -提交
>这里主要是 这种思路, 提示词可以自行选择.

我们先clone 这个项目 然后打开claude
```shell
git clone https://github.com/chenyme/grok2api
cd grok2api
claude
```
<br/>

#### 5.1.1 先探索: 
我们可以通过cc 快速了解这个项目:

可以先执行一下: `/init`  命令.给模型足够的上下文
<br/>
然后可以让它输出项目的核心功能,算法实现等
我的提示词:
```text
你现在是一位拥有 10 年经验的资深开源项目架构师和技术评审专家，精通多种编程语言、架构设计、代码质量、安全审计和社区运营。你曾经为多家顶级公司做过开源项目尽调。

现在需要你完整分析这个开源项目

请从以下几个方面进行分析：

1. 项目整体功能
	* 项目解决了什么问题
	* 主要应用场景

2. 技术架构
	* 使用了哪些技术栈
	* 系统架构（模块划分、核心组件）

3. 代码结构
	* 目录结构说明
	* 关键模块作用

4. 核心实现
	* 主要算法或关键逻辑
	* 重要设计模式

```
![](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326115248541.png)
#### 5.1.2 再规划 -编码 -提交
了解完整个项目后, 我希望增加一个可以通过上传txt文件导入token的功能(实在想不到要扩展啥了,随便整一个吧)

使用`alt + m` 切换到plan mode
描述我的需求:
```text
当前项目支持的token导入,都是需要手动输入token进去.我希望可以通过上传txt文件进行导入
流程:
1. 点击通过文件导入按钮后,弹出窗口,可以选择目标pool,以及上传txt文件按钮
2. 点击上传txt文件,弹出文件资源管理器进行选择
3. 对文件进行解析, 一行一个token
要求:
- 完成之后进行测试,没问题再交付
```

cc会先给出计划:
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326120330293.png)

然后执行
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326120833975.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326120841851.png)
可以看到已经完成了 ,也没啥问题

然后可以让cc进行提交
### 5.2 检查点与回滚
为了防止cc 在做新功能时, 将已经做好的功能给修改掉. 我们要先存档,当结果不行时,回退.
#### 5.2.1 方式一: 每完成一个功能都commit
==每次完成一个功能后, 都提交一次代码,保证有还原点==

使用cc 进行编码

需要回滚:
##### 5.2.1.1 Claude 修改了文件，还没提交，想全部撤销
```shell
# 撤销所有未提交的工作区修改 
git checkout . 
# 或者更彻底（包括新增文件） 
git clean -fd 
git checkout .
```

##### 5.2.1.2 Claude 已经 commit 了，想撤销最近几次提交
```shell
# 撤销最近1次 commit，保留文件改动（最安全）
git reset --soft HEAD~1

# 撤销最近1次 commit，丢弃文件改动
git reset --hard HEAD~1

# 撤销最近3次 commit
git reset --hard HEAD~3
```

##### 5.2.1.3 只想撤销某一个特定的 commit，不影响其他
比如:
```text
A -- B -- C -- D ← master

          ↑
commit-hash = C
```

c有bug ,不能用reset ,会将c与d一起干掉
就要使用`git revert C`  这个命令会创建一个全新的提交,包含撤销c的变更


#### 5.2.2 方式二:每次开始开发前创建还原点(不推荐,麻烦)

比如你要开始做一个登录功能

1. 先提交一次: `git add . && git commit -m "before claude : implement  login"`
2. 开始编码
3. 满意提交
4. 不满意回退
<br/>
与之前是类似的, 只不过多了一次开始的commit.  
你可以直接让cc帮你完成 : `claude "在开始修改之前，先执行 git add . && git commit -m 'checkpoint'， 然后再实现新功能，完成后告诉我 commit hash"`  或者自定义一个slash命令
<br/>
但是这样做会污染commit 
如何解决:
##### 5.2.2.1 方法一: 用完压缩
工作流变成了: 
```text
# Claude 工作期间打了多个 checkpoint commit：
abc1234 checkpoint: before auth
def5678 checkpoint: after middleware
ghi9012 checkpoint: fixed tests
jkl3456 feat: complete user auth  ← 真正有意义的 commit

# 最后压缩成一个干净的 commit
git rebase -i HEAD~4
```

在 rebase 界面里把 checkpoint 都标记为 `squash` 或 `fixup`：
```shell
pick abc1234 checkpoint: before auth
squash def5678 checkpoint: after middleware
squash ghi9012 checkpoint: fixed tests
squash jkl3456 feat: complete user auth
```

##### 5.2.2.2 方法二: 用分支隔离,合并时一键压缩
```shell
# Claude 在独立分支上随意 commit
git checkout -b claude/user-auth

# Claude 工作，产生一堆 checkpoint commits...

# 合并回 main 时一键压缩
git checkout main
git merge --squash claude/user-auth
git commit -m "feat: implement user authentication"

# 删掉临时分支
git branch -D claude/user-auth
```

##### 5.2.2.3 方法三: git stash

1. `git status` 确认当前状态干净
	1. 如果有未提交的改动, 可以手动提交/让cc帮你提交.  
	2. 或者先 stash 当前改动`git stash push -m "my current work`
	3. 确保显示:nothing to commit, working tree clean
2. 打stash保护点:  
	1. ==随便改动东西==,确认有改动,比如 随便加个空格啥的(不改动建立不成功)
	2. `git stash push -m "before claude: implement file token parse "`
3. 让cc 开始工作 ,并且禁止它使用commit
4. 审查claude 改动
	1. `git status` 查看改动了哪些文件
	2. `git diff` 查看具体的改动 
5. 判断结果
	1. 对于满意的结果: 提交并清除stash `git add . && git commit -m "feat(token): implement file parse" && git stash drop`
	2. 对于不满意的结果:  回滚: `git checkout . && git clean -fd && git stash pop`



### 5.3 用subAgent 做调查
#### 5.3.1 5.3subAgent是什么?
subAgent 是 Claude Code 在执行主任务时，**动态派生出来的独立 Claude 实例**，专门负责某个具体的子任务，完成后将结果汇报给主 Agent，然后自动销毁。

每个 subagent：
- 运行在**独立的上下文窗口**中，避免主会话 token 膨胀
- 拥有**自定义的系统提示**（你完全控制它的行为规则）
- 只能使用**你授权的工具**（可限制为只读，或允许编辑+bash）
- 可以指定**具体的模型**（Sonnet、Opus 等）
- 执行完毕后向主 agent 返回一个干净的摘要

#### 5.3.2 为什么需要subAgent
核心问题是**上下文窗口消耗**：

```
# 没有子 Agent 时：
主 Agent 读取大量文件 → 消耗 70,000~100,000 tokens
→ 上下文窗口快满了，还没开始干活

# 有子 Agent 时：
主 Agent 派出子 Agent 去"侦察"
→ 子 Agent 消耗自己的上下文窗口，爆掉也无所谓
→ 子 Agent 汇报精简结果（"这5个文件最关键"）
→ 主 Agent 上下文窗口保持充裕，继续主任务
```

#### 5.3.3 如何使用
第一种:claude code 会根据任务的复杂度进行判断,自行生成. 不需要任何处理
第二种: 手动创建

也是直接在对话框里输入:

```text

  用子 agent 并行分析：
  1. 这个项目的所有 API 端点列表
  2. 所有数据库模型定义
  3. 所有测试的覆盖情况
  然后汇总报告给我
```
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326171038809.png)

或者创建多个agent 并行执行任务

```text
创建一个 Agent 团队：
- "backend" 负责实现 /api/users 接口
- "frontend" 负责构建对应的 UI 组件
两者协商好接口数据结构后各自实现
```


## 6 阶段六: hooks 自动化

### 6.1 什么是hooks
官方定义: `是用户定义的shell命令,再cc的生命周期的特定点执行,它们对 Claude Code 的行为提供确定性控制，确保某些操作始终发生，而不是依赖 LLM 选择运行它们` 
<br/>
类似于: 爬虫中,在网页的js文件中,写一个输出变量值的语句, 当加载页面,执行js代码会顺带执行这一句, 这一句打印代码就是hooks 就是钩子.   
就是在程序/cc 正常执行某个节点的前后插入一个自定义的动作
### 6.2 为什么需要hooks
因为hooks 是确定性的, 你在CLAUDE.md 里写的指令, cc可能会忽略. 但hooks一定会执行.
所以hooks用来执行一些强制指令,自动化流程, 比如cc要删库跑路 , 直接通过hooks拦截. 或者编辑完文件,格式化一下

### 6.3 hooks 的作用是什么
典型用途包括：
- **自动格式化**：每次编辑文件后自动跑 Prettier / Black 等格式化工具
- **拦截危险操作**：阻止 Claude 修改 `.env`、`secrets` 等敏感文件
- **消息通知**：Claude 需要用户输入时弹出系统通知
- **自动跑测试**：写入文件后在后台异步执行测试套件
- **强制收尾检查**：任务完成前用 LLM Prompt 验证是否满足测试/文档要求
- **对接外部系统**：通过 HTTP Hook 将事件推送到企业内部服务


### 6.4 支持的生命周期事件
既然是在cc的生命周期前后插入动作. 那就需要查看有哪些生命周期
通过/hooks 命令可以查看
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326213644317.png)

调用工具前后, subagent 执行前后等20多出可以插入hooks 的地方

### 6.5 hooks 的类型
| 类型        | 说明                                   |
| --------- | ------------------------------------ |
| `command` | 执行任意 shell 脚本，最常用                    |
| `prompt`  | 让 LLM 做一次性判断，返回 `{"ok": true/false}` |
| `agent`   | 启动完整子 Agent，可访问代码库工具，适合复杂检查          |
| `http`    | POST 到任意 HTTP 端点，适合对接微服务             |
|           |                                      |

### 6.6 如何定义hooks
#### 6.6.1 第一种:  直接让cc给你加 
发送`我希望你能加一个hooks,功能是每当 Claude 完成工作或者需要我的的输入时获进行桌面通知`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326224147215.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326224228740.png)


可以看到cc 先写了桌面通知的测试脚本,没问题后写入脚本文件,然后在 `~/claude/setting.json`中进行配置

#### 6.6.2 第二种 手动配置
第一步: 打开 `~/claude/setting.json` 文件
第二步:加入配置
```json
{
  "hooks": {
    "<事件名>": [
      {
        "matcher": "<正则，匹配工具名或文件名，可选>",
        "hooks": [
          {
            "type": "command",
            "command": "<你的命令>"
          }
        ]
      }
    ]
  }
}
```

比如编辑文件后自动Prettier 格式化 
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write --log-level silent"
          }
        ]
      }
    ]
  }
}
```


==如果命令很长,可以放到一个文件中==

### 6.7 其他有用的hooks

比如异步后台跑测试
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/run-tests.sh",
            "async": true,
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

其他的可以在官方文档中查看: https://code.claude.com/docs/zh-CN/hooks-guide#windows-powershell


## 7 阶段七: mcp集成 和skill 集成







### 7.1 比较好玩的mcp

#### 通过mcp 调用其他ai
通过mcp 去调用codex 写代码
通过mcp 调用 grok 搜索  https://linux.do/t/topic/1674101


### 操作数据库的mcp






## 8 自定义sub-agent

前面我们简单介绍了一下subagent,但是claude自己生成的subagent终究是不可控的,我们希望我们自己来主导提示词,模型等. 从而创建一个专业团队
比如: agent a 专门负责跑测试, agent b 专门负责更新文档
<br/>

所以这里我们就学习一下如何自定义subagent

### 8.1 方式一:  /agents命令(有bug,没法用,详见github,直接看方式二吧)
bug 地址: https://github.com/anthropics/claude-code/issues/4351  还没解决
输入`/agent` 命令 ,选择 create new agent
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230204128.png)
选择创建agent 的位置. 选择 是给项目创建还是全局创建
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230346353.png)
选择创建方式: 通过与claude沟通创建/手动创建
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230413019.png)

这里选择1之后,就是描述你的subagent 了
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230457303.png)

比如这里我希望有一个code-review 的智能体
```text
Create a code-review subagent with the following settings:

Name: code-reviewer
Description: Use this agent when the task involves reviewing code for quality, security, performance, or best practices. Automatically trigger when asked to review, audit, or inspect code.
Tools: Read, Glob, Grep (read-only, no file editing)
Model: sonnet

Instructions:
You are a senior software engineer specializing in code review. Your job is to:
1. Identify bugs, logic errors, and edge cases
2. Flag security vulnerabilities (e.g., injection, auth issues, exposed secrets)
3. Point out performance bottlenecks
4. Check code style and naming consistency
5. Suggest improvements with specific line-level comments

Rules:
- Never modify files directly
- Always provide specific, actionable feedback with line references
- Rate overall code quality on a scale of 1–10
- Be concise but thorough
```

### 8.2 方式二: 引导创建

输入`/agent` 命令 ,选择 create new agent
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230204128.png)
选择创建agent 的位置. 选择 是给项目创建还是全局创建
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230346353.png)
选择创建方式: 通过与claude沟通创建/手动创建
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326230413019.png)
选择2,输入你的subagent 名称
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326231922580.png)
输入你的系统提示词
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326231945970.png)
```text
---
name: code-reviewer
description: Use this agent when the task involves reviewing code for quality, security, performance, or best practices. Trigger when asked to review, audit, or check code.
tools: Read, Glob, Grep
model: claude-sonnet-4-5
color: blue
---

You are a senior software engineer specializing in code review. Your responsibilities:

1. Identify bugs, logic errors, and edge cases
2. Flag security vulnerabilities (injection, auth issues, exposed secrets)
3. Point out performance bottlenecks
4. Check naming conventions and code style
5. Suggest concrete, actionable improvements with line references

Rules:
- Never modify files directly
- Always reference specific file paths and line numbers
- Rate overall code quality 1–10 at the end
```

什么时候claude应该使用这个subagent
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232110265.png)
`Trigger when asked to review, audit, or check code.`
选择可以使用的工具
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232244190.png)
选择模型
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232302532.png)
背景颜色
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232326759.png)
agent的memory 位置
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232404399.png)
确认并保存
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232428814.png)

可以看到我们创建的agent了
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326232446619.png)


### 8.3 方式三: 手动创建

在项目目录下创建文件 `.claude/agents/my-agent.md`（项目级，可 git 共享） 或 `~/.claude/agents/my-agent.md`（个人全局级）

文件格式为 YAML 头部 + Markdown 指令正文：

```
---
name: code-reviewer
description: Use this when the task involves reviewing code for quality, security, or best practices. Only suggest changes, never edit files.
tools: Read, Glob, Grep
model: sonnet
color: blue
---

You are a senior engineer focused solely on code quality.
Be extremely critical and specific.
Never write new code unless explicitly asked.
```



==既然这种方式存在== ,那我们就可以类似定义hooks那样,直接告诉cc,我要定义一个subagent,让cc直接去创建文件.

### 8.4 如何调用sub-agent

第一种:根据你在系统提示词中的设定,claude会自动匹配并触发
第二种: 显式调用 `@code-reviewer 帮我检查auth模块`
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260326235351858.png)

第三种: 告诉claude ,让它调用: `在后台运行 test-runner 来跑单元测试`



比如: 这里给一个比较综合的提示词, 结合了: 
- 先探索-再规划-再开发
- 测试
- git工作流
- hooks
- sub-agent

```text
帮我从零构建一个任务管理 REST API 项目，要求：

  技术栈：FastAPI + SQLite + pytest

  功能需求：
  - CRUD 操作：任务的增删改查
  - 任务有：标题、描述、状态（todo/doing/done）、优先级、截止日期
  - 支持按状态和优先级筛选

  工程化要求：
  - 完整的单元测试和集成测试，覆盖率 > 80%
  - 自动生成 API 文档
  - Git 规范提交（每个功能一个 commit）
  - 配置 pre-commit hook 自动 lint
  - README 说明如何启动和测试

  执行方式：
  1. 先用 Plan Mode 规划完整架构
  2. 用 sub-agent 并行开发和测试
  3. 完成后运行所有测试验证
  4. 生成最终项目文档
```
进入计划模式
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327084425137.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327084559514.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327084702948.png)

可以看到不到四分钟,开发任务已完成, 剩余git提交 和跑测试
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327084816349.png)

跑完测试,发现问题,进行修复
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327084948321.png)
继续跑测试
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327085009594.png)

七分半开始提交git
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327085241360.png)

八分钟全部完成
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327085405750.png)

可以看到 状态,优先级,截止日期,筛选都有.
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327085820495.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327085934395.png)

## 9 多agent协作
### 9.1 什么是多agent 协作

是指由一个**主 Agent（Orchestrator）** 统一调度，派生出多个**子 Agent（Subagent）** 并行或串行工作，最终汇总结果的工作模式。

也就是上面 调用sub-agent 的第一种方式.
### 9.2 两种形式

- subagent 并行:  主 Agent 通过 `Task` 工具派生子 Agent，各自独立工作后汇报结果
- agent team:  多个完全独立的 Claude 实例，可以互相直接通信，协同完成任务(实验性功能)

### 9.3 为什么需要多agent协作
1. 串行执行太慢了, 比如我搭一个网站, 要写前端,后端,数据库,串行就要消耗并行三倍的时间
2. 再sub-agent中提到的 上下文窗口会被迅速耗尽的问题, 这里就不重复了

### 9.4 使用场景
- **并行代码分析**：同时扫描安全漏洞、性能问题、代码风格，各由一个专用 Agent 负责
- **全栈并行开发**：frontend Agent 和 backend Agent 同时工作
- **多视角代码审查**：安全 Agent + 性能 Agent + 可读性 Agent 同时 review 同一 PR
- **大型重构**：将模块拆分后分配给多个 Agent 并行处理
- **研究汇总**：同时调研多个技术方案，汇总对比


### 9.5 如何使用

#### 9.5.1 方式一: 直接告诉cc,并提及agent,并行执行
比如: 
```text
  用多个sub-agent 并行分析：
  1. 这个项目的所有 API 端点列表
  2. 所有数据库模型定义
  3. 所有测试的覆盖情况
  然后汇总报告给我
```

#### 9.5.2 方式二: 先给出计划,根据计划中的任务,创建sub-agent并行执行
就是8.4最后的那段提示词

#### 9.5.3 方式三: 开启agent team(跨会话,不稳定)

启用
```json
# 在 .claude/settings.json 中添加：
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```
或者直接让cc帮你加

<br/>
直接描述需要组队的任务:
比如: 
```text
启动一个 Agent Team：
  - Lead：负责协调和最终汇总
  - Teammate A：专门分析 src/auth/ 目录的安全漏洞，重点关注 JWT 处理和输入验证
  - Teammate B：专门分析性能瓶颈，找出慢查询和内存泄漏风险
  两个 teammate 分析完互相review对方的发现，然后汇报给 Lead
```


![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327102321641.png)
`ctrl +t` 查看 , `shift + 上下箭头` 选择,enter 查看单个成员在干啥
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327102352164.png)

a 已完成
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327102441662.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327102504487.png)

完成了, 这也太耗token了
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260327102619570.png)


##### 9.5.3.1 用agent team 构建功能

比如我们再8.4 中搞了一个 任务管理restapi 项目

这里加一个认证系统
```text
 启动 Agent Team 完成用户认证系统：

  Lead：架构设计 + 最终集成测试

  Teammate A（后端开发）：
  - 实现 JWT 登录/注册接口
  - 密码 bcrypt 哈希
  - Token 刷新机制

  Teammate B（测试工程师）：
  - 与 Teammate A 同步接口定义后立即写测试
  - 覆盖正常流程、边界情况、安全攻击（SQL注入、暴力破解）

  Teammate C（文档）：
  - 监听 A 的进度，实时更新 API 文档
  - 生成 Postman collection

  等待所有 teammate 完成后，Lead 运行集成测试并修复冲突
```

这个我就不试了


## 10 CICD 自动化集成
### 10.1 自动 PR 代码审查
创建github workflow 自动进行pr代码审查(这个功能其实有现成模板)
输入: 
```text
帮我创建一个 GitHub Actions workflow，要求：
  - 每次有 PR 时自动触发
  - 用 Claude Code 审查变更的文件
  - 把审查意见作为 PR comment 发出
  - 如果发现安全问题，自动给 PR 打上 "security-review" 标签
```

### 10.2 自动进行测试修复
```text
帮我创建一个 GitHub Actions workflow：
  - 每次 push 到 main 分支时触发
  - 运行测试套件
  - 如果测试失败，自动让 Claude 分析失败原因并尝试修复
  - 如果修复成功，自动创建一个 fix PR
```




## 11 Channels 事件驱动编程

让外部事件(比如 telegram ) 直接触发claude code 执行任务 ,实现远程指挥

### 11.1 配置一个telegram bot


## 12 Plugins 插件


## 13 总结出工作流
要结合的功能
- 

---

> 作者: cheems  
> URL: http://localhost:59658/ai/%E5%B7%A5%E5%85%B7%E4%BD%BF%E7%94%A8/b89a7739/  

