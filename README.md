📖 项目简介
本项目基于 Kaggle 平台完成，实现了一个面向企业出海场景的战略分析智能体。
智能体通过多轮交互收集 7 项关键业务参数（产品、候选国家、目标客群、价格区间、进入周期、渠道资源、风险偏好），
自动调用世界银行 API 拉取候选国家的 9 项宏观指标，并结合 SKILL.md 中定义的 overseas-feasibility 技能，
由大语言模型完成六维度评分、加权汇总、最优市场推荐与行动清单生成，最终输出 Markdown 格式的出海可行性决策报告。
完整代码、Prompt 组装、数据拉取与示例输出见 notebook。

📂 文件说明
ai-interviewer.ipynb：Jupyter Notebook，项目全部代码与实验输出（文件名沿用仓库现有 notebook，可按需重命名）

SKILL.md：必需依赖，定义 overseas-feasibility 技能的评分维度、权重与报告结构，需上传至 /kaggle/working/SKILL.md

出海可行性报告.md：运行后自动生成的报告文件，保存于 /kaggle/working/

📌 Notebook 在线查看链接
👉 点击查看完整项目

⚙️ 环境依赖
python
Python 3.13
openai          # 通过 OpenAI 兼容接口调用智谱 GLM
requests        # 调用世界银行 API
kaggle_secrets  # Kaggle 环境读取 API Key；本地运行可改为环境变量
需要智谱开放平台 API Key（ZHIPU_API_KEY），示例使用 glm-4-flash。
notebook 中另执行了 pip install zhipuai，但实际调用走的是 openai 兼容接口，该包非必需；pandas 虽被导入但未实际使用。

✨ 项目亮点
技能文档驱动的分析流程：将 SKILL.md 全文注入 System Prompt，强制模型按技能规定的 6 个维度、权重和报告结构输出，避免报告结构漂移。

真实宏观数据接入：通过世界银行 API 批量拉取 9 项指标（GDP 总量、人均 GDP、GDP 增速、CPI、人口、城镇化率、互联网用户占比、进口占 GDP 比重、失业率），取各指标最新年份数据，并在 Prompt 中明确「value=None 标注数据缺失，禁止编造」。

结构化业务信息收集：OverseaAgent 以交互式问答收集 7 项必填字段，国家字段自动拆分为 ISO3 列表，收集完成后校验完整性，不完整则终止流程。

一键生成并落盘报告：多国横向对比评分后自动选出最高分市场，输出推荐理由、风险提示、下一步行动清单与免责声明，并保存为 Markdown 文件。

🧠 工作流程
text
触发指令 → 交互收集7项业务参数 → 校验完整性
        → 世界银行API拉取9项宏观指标（多国）
        → 注入SKILL.md + 业务参数 + 宏观数据
        → LLM生成六维评分与决策报告
        → 打印并保存 出海可行性报告.md
📊 数据来源与输入说明
本项目不依赖 Kaggle 表格数据集，输入由两部分构成：

1. 用户交互输入（7 项）
产品类型、候选国家 ISO3 代码（逗号分隔）、目标客群、海外终端价格区间、期望进入周期、现有海外渠道资源、风险偏好。

2. 世界银行公开宏观数据（9 项指标）

指标代码	含义
NY.GDP.MKTP.CD	GDP 总量
NY.GDP.PCAP.CD	人均 GDP
NY.GDP.MKTP.KD.ZG	GDP 年增长率
FP.CPI.TOTL.ZG	CPI 通货膨胀率
SP.POP.TOTL	总人口
SP.URB.TOTL.IN.ZS	城镇化率
IT.NET.USER.ZS	互联网用户占比
NE.IMP.GNFS.ZS	进口商品服务占 GDP 比重
SL.UEM.TOTL.ZS	失业率
示例运行输入为：智能煎烤机/小家电，候选国 GBR、USA、JPN、SAU，目标客群为厨房小白与中产家庭，终端价 120 美元，期望 6 个月快速切入，渠道为海外本地经销商，中等风险。示例输出推荐 USA，报告自动保存至 /kaggle/working/出海可行性报告.md。

⚠️ 已知限制与改进方向
评分汇总方式：示例报告中「加权总分」为 6 个分项得分的直接相加（满分 600），尚未按 SKILL.md 权重做归一化加权，后续将改为权重乘加并输出 0–100 标准分。

缺失指标推断：部分国家部分指标返回 None，示例报告的推荐理由中仍出现了对缺失指标的定性引用，后续将增加「缺失指标不得作为推荐依据」的硬校验。

技能文档未入库：SKILL.md 需手动上传至 /kaggle/working/，否则 notebook 无法运行；建议将其纳入仓库并在代码中使用相对路径读取。

📝 使用说明
下载仓库代码，并将 SKILL.md 上传至 Kaggle 工作目录 /kaggle/working/SKILL.md

安装上述依赖，配置 ZHIPU_API_KEY（Kaggle 环境使用 Secrets；本地运行改为 os.getenv 读取环境变量）

打开 notebook，逐块运行代码

在触发提示处输入 开始出海分析，依次回答 7 项业务信息

等待智能体拉取世界银行数据并生成报告，结果打印在 notebook 中，同时保存至 /kaggle/working/出海可行性报告.md
