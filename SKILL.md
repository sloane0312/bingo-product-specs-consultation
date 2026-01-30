---
name: bingo-product-specs-consultation
description: |
  品高产品控标参数咨询比对工具。当产品经理 AI agent 接收到有关【品高容器云产品】、AiInfra、品高智算调度平台的控标参数咨询时自动触发。
  触发场景：咨询者发送批量或单个项目的功能参数和技术参数，询问标准产品是否符合。
  Use when receiving product specification consultation requests, comparing bid control parameters, or analyzing product compliance.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
argument-hint: "[咨询参数文件/文本] [产品类型:容器云|AiInfra|智算调度]"
---

# 品高产品控标参数咨询比对工具

## 功能概述

本 skill 用于处理【品高容器云产品】、AiInfra、品高智算调度平台的控标参数咨询需求。当咨询者发送项目的功能参数和技术参数，询问标准产品是否符合时，自动执行参数比对分析流程。

## 适用产品范围

| 产品线 | 知识库文件 | 说明 |
|--------|-----------|------|
| 品高容器云平台 | `/root/knowledge/品高容器云平台-V1.7.0-控标参数-v20260115.md` | 容器云控标参数文档 |

> **注意**：只读取文件名中包含【控标参数】的文档，其他文档（白皮书、用户手册等）不在查询范围内。

## 工作流程

### 第 1 步：接收并解析咨询参数

**输入格式支持**：
- 文本描述（直接粘贴的参数列表）
- Excel/CSV 文件（批量参数表）
- PDF 文档（需 OCR 提取）
- Word 文档
- 图片（参数截图）

**解析要点**：
1. 识别咨询的产品类型（容器云/AiInfra/智算调度）
2. 提取所有功能参数和技术参数
3. 标准化参数名称和数值单位
4. 记录参数来源（项目名称、文档页码等）

```
示例输入：
- CPU：≥64核
- 内存：≥256GB
- 支持 GPU 虚拟化：是
- 容器编排：支持 Kubernetes 1.28+
- 多租户隔离：网络级别隔离
```

### 第 2 步：查询标准产品控标参数

**知识库查询路径**：`root/knowledge/`

**查询策略**：
1. 根据产品类型定位对应知识库目录
2. 搜索标准产品规格文档
3. 提取标品的完整参数列表
4. 获取参数的版本信息和更新时间

**知识库文件**：
```
/root/knowledge/品高容器云平台-V1.7.0-控标参数-v20260115.md
```

**查询规则**：
- 只读取文件名包含「控标参数」的 `.md` 文件
- 查询命令：`ls /root/knowledge/*控标参数*.md`

**查询命令示例**：
```bash
# 查找所有控标参数文档
ls /root/knowledge/*控标参数*.md

# 读取控标参数文档全文
cat /root/knowledge/品高容器云平台-V1.7.0-控标参数-v20260115.md

# 搜索特定参数
grep -n "CPU\|处理器\|核数" /root/knowledge/*控标参数*.md
grep -n "Kubernetes\|K8s" /root/knowledge/*控标参数*.md
```

### 第 3 步：逐项参数比对

**比对逻辑**：

| 参数类型 | 比对规则 | 示例 |
|---------|---------|------|
| 数值型（≥） | 标品值 ≥ 咨询值 则满足 | CPU ≥64核，标品128核 → 满足 |
| 数值型（≤） | 标品值 ≤ 咨询值 则满足 | 延迟 ≤10ms，标品5ms → 满足 |
| 布尔型 | 标品支持该功能则满足 | 支持GPU虚拟化：是 → 检查标品是否支持 |
| 版本型 | 标品版本 ≥ 咨询版本 | K8s 1.28+，标品1.30 → 满足 |
| 枚举型 | 标品包含该选项则满足 | 隔离级别含"网络级" → 满足 |
| 文本描述型 | 语义匹配分析 | 需人工复核标注 |

**比对输出格式**：
```json
{
  "parameter": "CPU核数",
  "requirement": "≥64核",
  "standard_product_value": "128核（可扩展至256核）",
  "status": "满足",
  "match_type": "数值型",
  "detail": "标品支持128核，超出要求64核，满足需求"
}
```

### 第 4 步：生成比对结果报告

**输出内容**：

#### 4.1 满足的参数列表
```markdown
## ✅ 标品能够满足的控标参数

| 序号 | 参数名称 | 咨询要求 | 标品规格 | 符合说明 |
|------|---------|---------|---------|---------|
| 1 | CPU核数 | ≥64核 | 128核 | 标品支持128核，满足≥64核要求 |
| 2 | 内存容量 | ≥256GB | 512GB | 标品支持512GB，满足≥256GB要求 |
| 3 | K8s版本 | 1.28+ | 1.30 | 标品版本1.30，满足1.28+要求 |
```

#### 4.2 不满足的参数列表
```markdown
## ❌ 标品未能满足的控标参数

| 序号 | 参数名称 | 咨询要求 | 标品规格 | 差异分析 |
|------|---------|---------|---------|---------|
| 1 | GPU型号 | NVIDIA A100 | NVIDIA V100 | **不符合点**：咨询要求A100型号，标品仅支持V100。A100相比V100在AI训练性能上提升约2倍，内存从32GB提升到80GB。**建议**：需评估是否可接受V100或需定制升级。 |
| 2 | 存储IOPS | ≥100万 | 80万 | **不符合点**：标品最大IOPS为80万，低于要求的100万。差距20万IOPS（20%）。**建议**：可通过增加存储节点或升级SSD型号达成。 |
```

#### 4.3 需人工复核的参数
```markdown
## ⚠️ 需人工复核的参数

| 序号 | 参数名称 | 咨询要求 | 标品描述 | 复核原因 |
|------|---------|---------|---------|---------|
| 1 | 安全合规 | 等保三级 | 支持等保合规 | 具体等级需确认证书 |
```

### 第 5 步：创建 Excel 结果表格

**使用 xlsx skill 创建表格**：

```
/xlsx create
```

**Excel 表格结构**：

| Sheet名称 | 内容 |
|-----------|------|
| 比对总览 | 汇总统计和结论 |
| 满足参数明细 | 所有满足的参数详情 |
| 不满足参数明细 | 所有不满足的参数及差异分析 |
| 待复核参数 | 需人工确认的参数 |
| 原始数据 | 咨询参数原文和标品参数原文 |

**表格列定义（主表）**：

| 列 | 字段名 | 说明 |
|----|-------|------|
| A | 序号 | 自增序号 |
| B | 参数分类 | 功能参数/技术参数/性能参数 |
| C | 参数名称 | 具体参数项 |
| D | 咨询要求 | 咨询者提供的参数值 |
| E | 标品规格 | 标准产品对应参数值 |
| F | 比对结果 | 满足/不满足/待复核 |
| G | 详细说明 | 具体符合或不符合的说明 |
| H | 建议措施 | 针对不满足项的解决建议 |

**命名规则**：
```
[产品类型]_控标参数比对_[咨询项目名]_[日期].xlsx

示例：
- 品高容器云_控标参数比对_XX市政务云项目_20260130.xlsx
- 智算调度平台_控标参数比对_XX银行AI中台_20260130.xlsx
```

### 第 6 步：保存并发送邮件

**SMTP 服务器配置**：
- **SMTP 服务器**：smtp.163.com
- **SMTP 端口**：465（SSL 加密）
- **发件邮箱**：LucyChan0312@163.com
- **授权码**：通过环境变量 `SMTP_PASSWORD` 配置

**邮件发送配置**：
- **收件人**：chensilu@bingosoft.net
- **邮件主题**：`[控标参数比对] {产品名称} - {项目名称} - {日期}`
- **邮件正文**：包含比对结论摘要
- **附件**：Excel 比对结果表格

**邮件模板**：
```
主题：[控标参数比对] 品高容器云产品 - XX市政务云项目 - 2026-01-30

正文：
您好，

附件为【XX市政务云项目】的控标参数比对结果，针对品高容器云产品进行分析。

【比对结论摘要】
- 咨询参数总数：25 项
- 满足参数：20 项（80%）
- 不满足参数：3 项（12%）
- 待复核参数：2 项（8%）

【主要不满足项】
1. GPU型号：要求A100，标品V100
2. 存储IOPS：要求≥100万，标品80万
3. ...

【建议措施】
针对不满足项，建议：
1. GPU型号可通过定制方案升级
2. 存储IOPS可增加存储节点达成
...

详细比对结果请查阅附件 Excel 表格。

---
此邮件由产品经理 AI Agent 自动生成
```

**发送命令**：
```python
# Python 发送邮件（使用 163 邮箱 SMTP）
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders
import mimetypes
import os

# SMTP 配置
SMTP_SERVER = "smtp.163.com"
SMTP_PORT = 465
SENDER_EMAIL = "LucyChan0312@163.com"
SMTP_PASSWORD = os.environ.get("SMTP_PASSWORD")  # 163邮箱授权码

# 构造邮件与附件（确保 .xlsx 文件名与正确 MIME 类型）
msg = MIMEMultipart()
msg["From"] = SENDER_EMAIL
msg["To"] = "chensilu@bingosoft.net"
msg["Subject"] = "[控标参数比对] 发送结果"

# 正文
msg.attach(MIMEText("附件为控标参数比对结果。", "plain", "utf-8"))

# 附件（自动识别 MIME 类型并进行 Base64 编码）
attachment_path = "result.xlsx"
ctype, encoding = mimetypes.guess_type(attachment_path)
if ctype is None or encoding is not None:
    ctype = "application/octet-stream"
maintype, subtype = ctype.split("/", 1)
with open(attachment_path, "rb") as f:
    part = MIMEBase(maintype, subtype)
    part.set_payload(f.read())
    encoders.encode_base64(part)
    filename = os.path.basename(attachment_path)
    part.add_header("Content-Disposition", f"attachment; filename=\"{filename}\"")
    msg.attach(part)

# 发送邮件
with smtplib.SMTP_SSL(SMTP_SERVER, SMTP_PORT) as server:
    server.login(SENDER_EMAIL, SMTP_PASSWORD)
    server.sendmail(SENDER_EMAIL, "chensilu@bingosoft.net", msg.as_string())
```

## 完整执行示例

### 输入
```
咨询者消息：
"请帮我看下这些参数品高容器云能不能满足：
1. 支持 Kubernetes 1.28 及以上版本
2. 单集群支持≥1000个节点
3. 支持 GPU 资源池化和虚拟化
4. 支持多租户网络隔离
5. 提供图形化运维界面
6. 支持应用商店和 Helm Chart
7. CPU 架构支持 x86 和 ARM"
```

### 执行流程
```
1. [解析] 识别产品类型：品高容器云产品
2. [解析] 提取参数：7项功能/技术参数
3. [查询] 读取 /root/knowledge/*控标参数*.md 文档
4. [比对] 逐项对比标品参数
5. [生成] 创建比对报告
6. [Excel] 生成表格：品高容器云_控标参数比对_咨询项目_20260130.xlsx
7. [邮件] 发送至 chensilu@bingosoft.net
```

### 输出
```markdown
## 控标参数比对结果

**产品**：品高容器云产品
**咨询参数数量**：7 项
**比对时间**：2026-01-30

### 比对结论
- ✅ 满足：6 项（85.7%）
- ❌ 不满足：1 项（14.3%）
- ⚠️ 待复核：0 项

### 详细结果

#### ✅ 满足的参数

| 参数 | 要求 | 标品 | 说明 |
|------|------|------|------|
| K8s版本 | 1.28+ | 1.30 | 标品版本1.30，满足要求 |
| 集群节点数 | ≥1000 | 5000 | 标品支持5000节点，超出要求 |
| 多租户隔离 | 网络隔离 | 支持 | 标品支持网络级+命名空间级隔离 |
| 运维界面 | 图形化 | 支持 | 标品提供完整Web控制台 |
| 应用商店 | 支持 | 支持 | 标品内置应用商店+Helm支持 |
| CPU架构 | x86+ARM | 支持 | 标品支持双架构混合部署 |

#### ❌ 不满足的参数

| 参数 | 要求 | 标品 | 差异分析 |
|------|------|------|---------|
| GPU资源池化 | 支持 | 部分支持 | **不符合点**：标品当前支持GPU直通，但GPU池化和虚拟化功能在v3.2版本中为Beta状态，生产环境建议谨慎使用。**建议**：确认项目上线时间，v3.3版本（预计Q2发布）将正式支持GPU虚拟化。 |

---
Excel 文件已生成：品高容器云_控标参数比对_咨询项目_20260130.xlsx
邮件已发送至：chensilu@bingosoft.net
```

## 错误处理

| 场景 | 处理方式 |
|------|---------|
| 知识库中无对应产品 | 提示用户确认产品类型，或标注为"无标品参数" |
| 参数名无法匹配 | 尝试语义相似度匹配，匹配失败则列入"待复核" |
| 参数值格式异常 | 提示用户确认参数格式，记录原始值 |
| 邮件发送失败 | 保存本地副本，提示手动发送 |
| Excel 生成失败 | 降级为 Markdown 表格输出 |

## 配置项

```yaml
# 可在调用时覆盖的配置

# 知识库配置
knowledge_base_path: "/root/knowledge"
knowledge_file_pattern: "*控标参数*.md"  # 只读取文件名包含"控标参数"的文档

# SMTP 邮件配置
smtp_server: "smtp.163.com"
smtp_port: 465
smtp_ssl: true
sender_email: "LucyChan0312@163.com"
smtp_password_env: "SMTP_PASSWORD"   # 授权码存储在此环境变量中

# 收件人配置
email_recipient: "chensilu@bingosoft.net"

# 输出配置
excel_save_path: "./output/"
auto_send_email: true
include_raw_data: true
```

## 相关工具依赖

- **xlsx skill**：创建和编辑 Excel 表格
- **Read/Grep/Glob**：知识库文件搜索和读取
- **邮件工具**：发送比对结果邮件（需配置 SMTP 或邮件 MCP）

## 注意事项

1. **知识库时效性**：标品参数可能随版本更新变化，请确保知识库为最新版本
2. **参数歧义处理**：遇到表述模糊的参数，优先标注为"待复核"而非强行匹配
3. **敏感信息**：比对结果可能包含商业敏感信息，邮件传输注意安全
4. **批量处理**：大批量参数（>100项）建议分批处理，避免遗漏
