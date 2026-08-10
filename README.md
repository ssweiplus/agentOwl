# agentOwl

一个面向**授权 Agent 安全测试**的最小 Demo，参考 AgentVigil 的“Seed → 执行 → 评分 → 变异 → 再执行”闭环，但第一版刻意保留人工操作目标对话框。

## V1 能力

1. 创建测试会话并定义正常任务、测试目标、成功证据。
2. 复制当前节点 Prompt 到目标对话框。
3. 将目标回复粘贴回 Demo。
4. 程序按关键词或正则自动判定 `SUCCESS / FAIL / UNKNOWN`，并保存可解释原因。
5. 根据结果自动推荐下一轮优化方向：`Rephrase / Expand / Shorten / Contextualize / Generate similar`。
6. 从当前节点创建子节点，形成一棵可回溯的测试树。
7. 所有节点保存在 SQLite，可在页面中复盘并导出 JSON。

> 仅用于你有权限测试的模型、Agent 和业务系统。

## 为什么第一版采用规则 Judge

为了让结果可复现、可审计，V1 不依赖额外的 Judge LLM。每个测试会话显式配置“成功证据”：

- `keyword`：多个关键词全部出现则判成功。
- `regex`：正则命中则判成功。
- 同时检测常见拒绝信号，为失败原因提供解释。

后续可以在不改变节点数据模型的前提下接入 LLM-as-a-Judge。

## 运行

```bash
python -m venv .venv
# Windows
.venv\\Scripts\\activate
# Linux / macOS
# source .venv/bin/activate

pip install -r requirements.txt
streamlit run app.py
```

浏览器打开 Streamlit 输出的本地地址即可。

## 数据结构

### Session

- `user_task`：正常用户任务
- `attack_goal`：授权测试目标
- `success_pattern`：成功证据
- `pattern_mode`：keyword / regex

### Node

- `parent_id`：父节点
- `mutation`：本次变异类型
- `prompt`：本轮 Prompt
- `response`：目标回复
- `verdict`：PENDING / SUCCESS / FAIL / UNKNOWN
- `reason`：自动判定理由
- `direction`：下一轮优化建议

因此一轮测试天然形成：

```text
Root Seed
├── Rephrase → FAIL
│   ├── Contextualize → SUCCESS
│   └── Generate similar → FAIL
└── Expand → SUCCESS
    └── Shorten → SUCCESS
```

## V1 与 AgentVigil 的对应关系

| AgentVigil 概念 | agentOwl V1 |
|---|---|
| Seed Corpus | SQLite 中的 Prompt 节点 |
| Mutation | 手工编辑 + 自动推荐方向 |
| Target Agent | 人工复制到目标对话框 |
| Scorer | keyword / regex Judge |
| Search Tree | parent-child 节点树 |
| Coverage / ASR | V1 暂未加入，V2 可扩展 |
| MCTS | V1 暂未加入，V2 可根据历史分数做节点调度 |

## 建议的 V2

- LLM Judge：结合攻击目标和完整回复判断成功/部分成功/失败。
- 自动 Mutation：接一个 Helper LLM 自动生成候选子节点。
- Coverage：按测试任务维度统计覆盖率。
- ASR：统计 Seed 在多个测试任务中的成功率。
- MCTS/UCB：自动决定下一次从哪个节点继续探索。
- Target Adapter：HTTP / Browser / MCP / API 等自动执行适配器。
- 树节点增加标签、备注、模型版本和截图/证据附件。
