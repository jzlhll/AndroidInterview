**AI编程**

* 辅助代码阅读

* 新思路指引

* 人为干预和识别接受后代码的风险

* 辅助编写文章

* 自动提示辅助

  

目前最强编程组合：

rules + project rules + Skills + plan

模型：

gemini3pro，kimi，GLM4.7，GPT5.2，CC

检查：

code review bot



**Ai编程技巧**

> Boris Cherny团队11条workflow。plan，staff reviewer，persistent记忆，可视化。

* Git worktrees，多分支并行工作

* PlanMode 

  复杂任务拆解，实现；同时，再开一个干净的会话来reviewer。

  流程变更为：计划->执行->实现->review

  跑偏立刻暂停，回到PlanMode，重新定义边界再继续。

* 资产化记忆

  *施工手册，代码风格，目录约定，常用命令，边界，坑*

  每次完成开发后，跑一遍“技术债清扫”，小问题列出来，补充到md中

* MCP协议直连

  一些issue(如github链接)，figma等使用mcp来减少自己截图，复制等操作

* 问题修复

  直接把问题丢给AI，让他列出疑似点，不断对话，让它查找问题可能的原因。

## Skills

skill-creator，其实不用装，直接让AI工具自带的对话帮你写即可

Planning-with-files: 持久化markdown规划模式，会落地到本地文件

superpower：完整能力包，全流程，偏重。

code-review-expert：sanyuan0704， preflight（了解改动范围，git diff），solid 架构检查，可删除代码，安全扫描 ，代码质量，结构化输出。https://github.com/sanyuan0704/sanyuan-skills/tree/main/skills/code-review-expert

code Simplifier：聚焦修改过的代码，重复逻辑，github /anthropics



