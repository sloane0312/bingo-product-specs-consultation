# 邮件模板

本文档定义控标参数比对结果邮件的标准格式。

## 邮件配置

```yaml
# SMTP 服务器配置
smtp_server: smtp.163.com
smtp_port: 465
smtp_ssl: true  # 465 端口使用 SSL 加密

# 发件人配置（需要在 163 邮箱开启 SMTP 服务并获取授权码）
sender_email: "LucyChan0312@163.com"
sender_password: "${SMTP_PASSWORD}"  # 使用环境变量存储授权码，勿明文写入

# 收件人配置
recipient: chensilu@bingosoft.net
cc: []  # 可选抄送
subject_template: "[控标参数比对] {product_name} - {project_name} - {date}"
```

### 163 邮箱 SMTP 配置说明

1. 登录 163 邮箱 → 设置 → POP3/SMTP/IMAP
2. 开启 SMTP 服务
3. 生成授权码（用于替代密码）
4. 将授权码设置为环境变量 `SMTP_PASSWORD`

## 邮件正文模板

```
主题：[控标参数比对] {product_name} - {project_name} - {date}

{recipient_name}，您好！

附件为【{project_name}】的控标参数比对结果，针对{product_name}进行分析。

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【比对结论摘要】

咨询参数总数：{total_count} 项
├─ ✅ 满足参数：{match_count} 项（{match_percent}%）
├─ ❌ 不满足参数：{unmatch_count} 项（{unmatch_percent}%）
└─ ⚠️ 待复核参数：{review_count} 项（{review_percent}%）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【主要不满足项】

{#unmatch_items}
{index}. {param_name}
   要求：{requirement}
   标品：{standard_value}
   差异：{gap_description}
{/unmatch_items}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【建议措施】

{#recommendations}
{index}. {recommendation}
{/recommendations}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【待复核项说明】

{#review_items}
{index}. {param_name}：{review_reason}
{/review_items}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

详细比对结果请查阅附件 Excel 表格：
{excel_filename}

如有疑问，请随时沟通。

---
此邮件由产品经理 AI Agent 自动生成
生成时间：{timestamp}
参考知识库版本：{knowledge_version}
```

## 实际邮件示例

```
主题：[控标参数比对] 品高容器云产品 - XX市政务云平台项目 - 2026-01-30

陈思露，您好！

附件为【XX市政务云平台项目】的控标参数比对结果，针对品高容器云产品进行分析。

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【比对结论摘要】

咨询参数总数：15 项
├─ ✅ 满足参数：12 项（80.0%）
├─ ❌ 不满足参数：2 项（13.3%）
└─ ⚠️ 待复核参数：1 项（6.7%）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【主要不满足项】

1. GPU 虚拟化支持
   要求：支持 GPU 虚拟化，实现 GPU 资源细粒度分配
   标品：GPU 直通模式，虚拟化功能 Beta 状态
   差异：当前版本 v3.2 的 GPU 虚拟化为 Beta 功能，不建议生产环境使用

2. 存储 IOPS
   要求：≥100万 IOPS
   标品：单卷最大 80万 IOPS
   差异：存在 20万 IOPS（20%）的差距

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【建议措施】

1. GPU 虚拟化：建议确认项目上线时间，v3.3 版本（预计 2026 Q2）将正式支持 GPU 虚拟化功能；如项目紧急，可先采用 GPU 直通方案过渡

2. 存储 IOPS：可通过以下方式达成：
   - 方案A：增加存储节点数量（推荐）
   - 方案B：升级至企业级 NVMe SSD
   - 方案C：采用分布式存储多卷聚合

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

【待复核项说明】

1. 等保三级认证：标品支持等保合规部署，但具体认证证书需与商务确认是否已取得

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

详细比对结果请查阅附件 Excel 表格：
品高容器云_控标参数比对_XX市政务云平台项目_20260130.xlsx

如有疑问，请随时沟通。

---
此邮件由产品经理 AI Agent 自动生成
生成时间：2026-01-30 15:30:00
参考知识库版本：v2026.01
```

## 邮件发送脚本

如需通过命令行发送邮件，可参考以下脚本：

### 方式一：Python 脚本（推荐）

```python
#!/usr/bin/env python3
# send_comparison_email.py
# 发送控标参数比对结果邮件

import smtplib
import os
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# SMTP 配置
SMTP_SERVER = "smtp.163.com"
SMTP_PORT = 465  # SSL 端口
SENDER_EMAIL = "LucyChan0312@163.com"
SENDER_PASSWORD = os.environ.get("SMTP_PASSWORD")  # 从环境变量获取授权码

# 收件人配置
RECIPIENT = "chensilu@bingosoft.net"

def send_email(subject, body, attachment_path=None):
    """发送邮件"""
    msg = MIMEMultipart()
    msg['From'] = SENDER_EMAIL
    msg['To'] = RECIPIENT
    msg['Subject'] = subject

    # 添加正文
    msg.attach(MIMEText(body, 'plain', 'utf-8'))

    # 添加附件
    if attachment_path and os.path.exists(attachment_path):
        with open(attachment_path, 'rb') as f:
            part = MIMEBase('application', 'octet-stream')
            part.set_payload(f.read())
            encoders.encode_base64(part)
            filename = os.path.basename(attachment_path)
            part.add_header('Content-Disposition', f'attachment; filename="{filename}"')
            msg.attach(part)

    # 使用 SSL 发送
    with smtplib.SMTP_SSL(SMTP_SERVER, SMTP_PORT) as server:
        server.login(SENDER_EMAIL, SENDER_PASSWORD)
        server.sendmail(SENDER_EMAIL, RECIPIENT, msg.as_string())
        print(f"邮件已发送至 {RECIPIENT}")

if __name__ == "__main__":
    import sys
    subject = sys.argv[1] if len(sys.argv) > 1 else "控标参数比对结果"
    body_file = sys.argv[2] if len(sys.argv) > 2 else None
    attachment = sys.argv[3] if len(sys.argv) > 3 else None

    body = ""
    if body_file and os.path.exists(body_file):
        with open(body_file, 'r', encoding='utf-8') as f:
            body = f.read()

    send_email(subject, body, attachment)
```

### 方式二：Bash 脚本

```bash
#!/bin/bash
# send_comparison_email.sh
# 发送控标参数比对结果邮件

SMTP_SERVER="smtp.163.com"
SMTP_PORT="465"
SENDER_EMAIL="LucyChan0312@163.com"
RECIPIENT="chensilu@bingosoft.net"

SUBJECT="$1"
BODY_FILE="$2"
ATTACHMENT="$3"

# 使用 mailx 发送（需配置 SMTP SSL）
mailx -s "$SUBJECT" \
  -a "$ATTACHMENT" \
  -S smtp="smtps://${SMTP_SERVER}:${SMTP_PORT}" \
  -S smtp-use-starttls \
  -S ssl-verify=ignore \
  -S smtp-auth=login \
  -S smtp-auth-user="$SENDER_EMAIL" \
  -S smtp-auth-password="$SMTP_PASSWORD" \
  -S from="$SENDER_EMAIL" \
  "$RECIPIENT" < "$BODY_FILE"

echo "邮件已发送至 $RECIPIENT"
```

### 使用方法

```bash
# 设置环境变量（163邮箱授权码）
export SMTP_PASSWORD="your_163_authorization_code"

# Python 方式发送
python3 send_comparison_email.py "[控标参数比对] 品高容器云 - XX项目 - 2026-01-30" body.txt result.xlsx

# Bash 方式发送
./send_comparison_email.sh "[控标参数比对] 品高容器云 - XX项目 - 2026-01-30" body.txt result.xlsx
```

## 附件命名规范

```
{产品简称}_{比对类型}_{项目简称}_{日期YYYYMMDD}.xlsx

示例：
- 容器云_控标参数比对_XX市政务云_20260130.xlsx
- AiInfra_控标参数比对_XX银行AI中台_20260130.xlsx
- 智算调度_控标参数比对_XX大学算力中心_20260130.xlsx
```
