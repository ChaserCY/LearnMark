# 一次简单的SKILL使用，所有的流程
![](图片/AI/SKILL/SKILL与MCP/IMG-20260203171002586.png)
# Renference ，读(消耗Token)
设计多个SKILL选择，条件触发，Reference
即在总的SKILL中，设立条件，来触发特殊的SKILL
![](未命名.canvas)
# Script， 跑(不会消耗Token)
## 可运行文件类型
- 直接：
	- SKILL 源文件（核心类型）
	- SKILL 编译文件（高性能）
	- Cadence EDA 配置文件（特殊 SKILL 脚本）
- 间接：
	- 脚本类文件(sh,py,cmd....)
	- 可执行二进制文件(exe)
	- 文本类文件（非 “运行”，但可读取处理）
## 不可运行的文件类型
- 非 SKILL 语法的乱码 / 二进制文件（如`.o`、`.so`、`.dll`，需编译为可执行文件后间接运行）；
- 加密文件、非标准格式的自定义配置文件（SKILL 无解析能力）；
- 他 EDA(电子设计自动化) 工具的专属文件（如`.oas`、`.gds`，只能通过 Cadence 内置函数加载，而非 “运行”）。
## 特点：
Agent SKILL 中的Script 只会执行，不会读取
同样是，总的SKILL条件触发

# 渐进式披露机制
![](图片/AI/SKILL/SKILL与MCP/IMG-20260203171002639.png)

# SKILL与MCP
- **MCP（Model Context Protocol，模型上下文协议/应用层）** 是 Anthropic 于 2024 年 11 月推出的==**开源标准化通信协议**==，核心作用是为大语言模型（LLM）与外部工具、数据源和服务之间提供**统一的交互接口**，解决 AI 应用开发中 "集成碎片化" 和 "能力孤岛" 问题，被誉为 AI 领域的**USB-C 通用接口**Anthropic。
- **AI Agent 是介于用户和大模型之间的「智能中介 + 自主执行者」**，它不是简单的 “传声筒”，而是具备**目标导向、自主决策、工具调用能力**的 “工作智能助手”，能直接对接外部工具 / 系统，完成复杂任务闭环。
![](图片/AI/SKILL/SKILL与MCP/IMG-20260203171002696.png)
在代码执行方面：
安全，稳定性    MCP>Agent SKill， Skill适合运行轻量脚本
