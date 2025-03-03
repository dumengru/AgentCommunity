---
date: 2024-12-20 
authors:
  - anthropic
categories:
  - 智能体
  - 白皮书
links:
  原文链接: https://www.anthropic.com/research/building-effective-agents
---

# 构建高效的智能体

## 什么是智能体

“智能体”定义多样，在Anthropic，将所有相关变体归为智能体系统，并区分工作流和智能体：

<!-- more -->

- **工作流**：通过预定义代码路径编排LLM和工具的系统。
- **智能体**：LLM动态指导自身流程和工具使用，掌控任务完成方式的系统。

## 何时（以及何时不）使用智能体

使用LLM构建应用时，建议优先寻求简单解决方案，仅在必要时增加复杂性。因为智能体系统常以延迟和成本换取更好的任务性能。工作流适用于定义明确的任务，可提供可预测性和一致性；而在需要大规模的灵活性和模型驱动决策时，智能体是更好的选择。很多应用中，通过检索和上下文示例优化单个LLM调用通常已足够。

## 何时以及如何使用框架

有许多框架可简化智能体系统的实现，如LangGraph、Amazon Bedrock的AI Agent框架等。这些框架虽便于上手，但会增加抽象层，导致底层提示和响应难以调试，还可能诱使开发者增加不必要的复杂性。建议开发者先直接使用LLM API，若使用框架，务必理解底层代码。

## 构建模块、工作流和智能体

### 构建模块：增强型LLM

增强型LLM是智能体系统的基本构建块，通过检索、工具和内存等增强功能，当前模型能主动利用这些能力。实现时应根据具体用例定制这些功能，并为LLM提供简单且有文档说明的接口，如通过Model Context Protocol集成第三方工具。

### 工作流

- **提示链接**：将任务分解为一系列步骤，每个LLM调用处理上一个的输出，可在中间步骤添加编程检查。适用于可轻松分解为固定子任务的场景，目的是以延迟换取更高准确性，如生成营销文案并翻译、撰写文档大纲并基于大纲创作等。
- **路由**：对输入进行分类并引导至特定后续任务，实现关注点分离和构建更专业的提示。适用于复杂任务中有不同类别需分别处理，且分类准确的情况，如分类处理不同类型的客户服务查询、根据问题难度选择不同模型等。
- **并行化**：
    - **切片**：将任务分解为独立子任务并行运行，如实施防护措施时由不同模型实例分别处理用户查询和内容筛查、自动化评估LLM性能等。
    - **投票**：多次运行同一任务获取不同输出，如审查代码漏洞、评估内容是否合适等。
- **协调器 - 工作器**：中央LLM动态分解任务，分配给工作LLM并合成结果。适用于无法预测所需子任务的复杂任务，如复杂的编码任务、从多源收集和分析信息的搜索任务等。
- **评估器 - 优化器**：一个LLM生成响应，另一个进行评估和反馈循环。适用于有明确评估标准且迭代优化有价值的场景，如文学翻译、复杂搜索任务等。

### 智能体

随着LLM关键能力成熟，智能体开始在实际应用中出现。智能体从人类用户的命令或交互讨论开始工作，明确任务后独立规划和操作，在执行过程中从环境获取“基本事实”评估进展，并可在检查点或遇到障碍时暂停获取人类反馈。适用于难以预测步骤数量、无法硬编码固定路径的开放式问题，在可信环境中可实现任务扩展，但因其自主性会带来更高成本和错误累积风险，建议在沙盒环境中进行广泛测试并设置适当的防护措施，如用于解决SWE-bench任务的编码智能体、Claude使用计算机完成任务的“计算机使用”参考实现等。

## 组合和定制这些模式

这些构建块并非固定不变，开发者可根据不同用例进行塑造和组合。成功的关键是衡量性能并迭代实现，仅在增加复杂性能明显改善结果时才进行。

## 总结

在LLM领域取得成功的关键是构建符合需求的系统。从简单提示开始，通过全面评估进行优化，仅在简单解决方案不足时才添加多步骤的智能体系统。实现智能体时遵循三个核心原则：保持设计简单；通过明确展示规划步骤提高透明度；通过完善的工具文档和测试精心打造智能体 - 计算机接口（ACI）。框架有助于快速起步，但在投入生产时应根据实际情况减少抽象层，使用基本组件构建。

## 附录1：智能体的实际应用

### 客户支持

结合聊天机器人界面和工具集成的增强功能，支持交互自然且需外部信息和操作，可集成工具获取数据、处理操作，成功可通过用户定义的解决方案衡量，一些公司采用基于使用情况的定价模式体现了对智能体有效性的信心。

### 编码智能体

软件开发领域对LLM功能需求增长，代码解决方案可通过自动化测试验证，智能体可根据测试结果迭代，问题空间明确且结构化，输出质量可客观衡量。Anthropic的智能体可基于拉取请求描述解决SWE-bench Verified基准中的实际GitHub问题，但仍需人工审核确保符合系统要求。

## 附录2：工具的提示工程

工具是智能体的重要组成部分，工具定义和规范应与整体提示一样受到重视。选择工具格式时，要给模型足够思考空间，保持格式接近模型在互联网文本中常见的形式，避免过多格式“开销”。创建良好的ACI可从以下方面着手：从模型角度思考工具使用的便捷性；优化参数名称和描述；测试模型使用工具的情况；对工具进行防错设计，如Anthropic在构建SWE-bench智能体时优化工具，将相对文件路径改为绝对文件路径以避免模型出错。 