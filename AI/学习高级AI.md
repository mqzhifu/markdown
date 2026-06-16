# 概览-进货史


2024年：GitHub Copilot，最早推出可写代码的AI，主要是以IDE插件的形式，以提示词的方式辅助写代码


openclaw 是2026-2月开始火爆的
2025年下半年：Cursor（最早整合了IDE写代码）  Claude Code


claude code:
- Claude 4.x 这类模型，在训练时被加入了大规模的强化学习（RL）和自我博弈（Self-Play）。它在回答你的编程问题前，会在后台自己跟自己探讨不同的架构方案，自己推翻自己的错误假设。这就导致它在代码的逻辑严密性、边界情况（Edge Cases）处理上，表现得像一个经验丰富的老架构师 [N/A]。
- 原生多模态长文本 (Long-Context) + 自主执行沙箱闭环
- 多 agent 协作，agent 有等级，高级驱动低级



# MCP

MCP：HOST -> LLM ->Client ->server

HOST ->LLM -> HOST -> Client -> Server ->Host- >LLM->Host 结束  


# openClaw

先在：飞书 微信 TG 上设置一个聊天机器人，并设置一个钩子，收到消息后，会请求S端
S端：网关在接收这些事件消息：格式化消息，生成统一标准的消息
网关：接收标准格式消息，串行的指令队列
Agent 智能体层：
- 加载各种基础 skill.md
- 加载工具指令
- 将指令 生成 提示器
- 将这份指令推送给LLM进行分析预测
如果LLM需要调用 工具时：
- 安全审批
- 沙箱执行
工具执行的结果返回给LLM，让LLM分析，是否正确：返回给网关
如果错误：再试一次，重度次数过多，就需要人类介入
