---
name: ai-leave-workflow
description: 'AI 智能请假流水线。用企业微信 API 实现端到端请假自动化：收集→确认→提交→回执。默认发起人+审批人均为 your_name_here（自审）。已配置可信IP，无需预配成员。'
agent_created: false
---

# AI 智能请假流水线

用企业微信 API 实现端到端请假自动化。**不使用腾讯文档**，只走企微审批。

---

## 🚨 流程铁律：必须严格按步骤执行

**请假涉及考勤和薪资核算，任何一步都不能跳过、合并或颠倒顺序。**

```
Step 1 → Step 2 → Step 3 → Step 4
收集      确认      提交      回执
```

| 规则 | 说明 |
|------|------|
| 不可跳步 | 4 步必须全部执行，缺一不可 |
| 不可合并 | 每步独立完成，确认后再进入下一步 |
| Step 2 不可省略 | 用户确认识别结果是强制步骤，提交前必须展示 |
| Step 3a 不可省略 | 每次提交前必须调用 `gettemplatedetail` 获取实时选项 |
| 请假类型必填 | 请假类型（年假/事假/病假等）必须从模板 API 实时获取 |
| 极简文案 | 主题用「请假申请 - [姓名] - [请假类型]」，描述用最简形式 |

---

## 🔧 配置说明

**所有企业信息已预配置，无需重复操作。**
**默认发起人固定为：your_name_here；审批人固定为：your_name_here（自审）。**

### 已预配置好的
| 配置项 | 说明 |
|-------|------|
| ✅ CorpID / Secret | 企业信息，共用 |
| ✅ 审批模板ID | 请假模板 |
| ✅ 默认发起人 | **your_name_here** |
| ✅ 审批人 | **your_name_here（自审）** |
| ✅ 可信IP | 已在后台配置（your_trusted_ip_here） |

### 默认发起人与审批人（已预配置）
| 配置项 | 值 | 说明 |
|-------|-----|------|
| 默认发起人 `default_creator` | **your_name_here** | 默认以此账号发起请假 |
| 审批人 `default_approver` | **your_name_here** | 自审（发起人=审批人） |

> **默认行为**：发起请假时直接使用 `default_creator`（your_name_here），无需每次询问「你是谁」。
> **帮别人请假**：若用户明确说「帮 XXX 请假」并提供对方 UserID，则以指定 UserID 作为发起人覆盖默认值，审批人仍为 your_name_here。
> 发起人 userid 必须在应用可见范围内，否则 `applyevent` 会报 `not in app visible range`，需管理员在后台添加。

### 引导话术（仅首次/帮他人时）

**默认情况：**
> "请假 Skill 已配置好默认发起人（your_name_here）和审批人（your_name_here，自审）。直接说「帮我请X天XX假」即可发起。"

**帮他人请假：**
> "请告诉我对方的 **姓名** 和 **企业微信 UserID**（通讯录里的唯一标识），我将以该账号发起，审批人仍为 your_name_here。"

---

## ⚠️ 核心铁律：数据真实，绝不猜测

**请假涉及考勤和薪资，数据必须100%准确，一个字都不能编。**

- ✅ 请假日期、天数必须精确计算
- ✅ 请假原因如实记录，不修饰、不美化
- ✅ 紧急联系人信息必须由用户提供，禁止编造
- ❌ 禁止根据"常识"推测请假类型（如「不舒服」≠一定是病假）
- ❌ 禁止擅自修改日期格式或天数
- ❌ 禁止用近似值替代真实天数（如2.5天不能四舍五入为3天）

---

## ✅ 请假分类对照表（提交前必查）

### 请假类型映射（模板 API 实时获取，以下为常见对照）

```
- 年假（带薪年休假）
- 事假（个人事务）
- 病假（疾病/就医）
- 婚假（结婚）
- 产假（生育）
- 陪产假（配偶生育）
- 丧假（直系亲属去世）
- 调休假（加班调休）
- 哺乳假（哺乳期）
- 工伤假（工伤）
```

### 提交前最后核对
```
- [ ] 请假类型是否正确？
- [ ] 起止日期是否准确？
- [ ] 请假天数是否计算正确？（含不含周末/节假日？）
- [ ] 请假原因是否真实合理？
- [ ] 审批人是否正确？（your_name_here）
- [ ] 紧急联系人是否已填写？
```

---

## 🧠 系统固定配置

| 配置 | 值 |
|------|-----|
| 发起人 | **your_name_here（default_creator）** |
| 审批人 | **your_name_here（自审）** |
| 请假主题风格 | 统一格式：「请假申请 - 姓名 - 请假类型」 |
| 天数计算方式 | 由用户首次确认（自然日/工作日） |
| 腾讯文档 | **不使用** |

---

## 首次启动检测（自动执行）

**每次触发 skill 时，AI 首先执行此检查：**

```
检查是否有 leave_config.json 文件
```

| 情况 | 行为 |
|------|------|
| 文件存在 | 读取配置，进入正常请假流程 |
| 文件不存在 | 进入上方「引导话术」，逐项引导用户填写姓名 + UserID |

配置文件查找顺序：
1. `~/.workbuddy/skills/ai-leave-workflow/leave_config.json`
2. `(当前工作目录)/leave_config.json`
3. `scripts/leave_config.json`

找到任意一个即加载。

---

## 核心流程（4步，含强制验证）

### Step 1: 收集请假信息

通过对话了解用户的请假需求。需要收集以下信息：

| 字段 | 说明 | 获取方式 |
|------|------|---------|
| 请假人 | 谁请假 | **默认使用 `default_creator`（your_name_here）；若用户明确帮他人请假，则用用户提供的 UserID** |
| 请假类型 | 年假/事假/病假/婚假等 | 用户选择 |
| 开始日期 | 请假的起始日期 | 用户提供 |
| 结束日期 | 请假的结束日期 | 用户提供 |
| 请假天数 | 自动计算 | 根据开始/结束日期计算 |
| 请假原因 | 简要说明请假事由 | 用户提供 |
| 紧急联系人 | 姓名 + 手机号（可选但推荐填写） | 用户提供 |
| 工作交接人 | 请假期间工作交接对象（可选） | 用户提供 |

**天数计算公式：**
- 自然日天数：`(end_date - start_date).days + 1`
- 工作日天数：排除周末和法定节假日

**天数计算规则：**
- ⚠️ 必须明确询问用户：按 **自然日** 还是 **工作日** 计算？
- 如果是工作日，用 Python 计算排除周六日（见文末模板）
- 如果是自然日，直接用日期差 + 1

**批量/连续请假处理：**
- 如果用户一次性说「帮我请 3 天的年假，从下周一（2026-07-20）开始」，自动计算结束日期（2026-07-22）
- 如果用户说「请周一到周三的假」，确认是否含周末

### Step 2: 请假信息确认（必须，不可跳过）

**在提交审批之前，必须将信息完整展示给用户确认。**

```
📋 请假信息确认：

| 项目 | 内容 |
|------|------|
| 👤 请假人 | your_name_here（userid: your_name_here） |
| 👑 审批人 | your_name_here（自审） |
| 📂 请假类型 | 年假 |
| 📅 开始日期 | 2026-07-20（周一） |
| 📅 结束日期 | 2026-07-22（周三） |
| 🔢 请假天数 | 3 天（工作日） |
| 📝 请假原因 | 回家探亲 |
| 📞 紧急联系人 | 李四 13800138000 |
| 🔄 工作交接人 | 王五 |

以上信息是否准确？提交后将发送到 your_name_here 审批，无法撤回。
```

**用户确认后才能进入下一步。没有例外。**

### Step 3: 通过企业微信 API 提交真实审批

使用 Python + `urllib` 调用企微 `/cgi-bin/oa/applyevent` API。

**提交前再次核对：审批单中的每一条数据是否与用户确认的一致。**

#### 3a. ⚠️ 获取模板选项（必须，每次提交前执行）

**提交前必须先调用 `gettemplatedetail` API 获取模板的请假类型和控件 ID，禁止使用硬编码或缓存的 key 值。**

```python
import urllib.request, json

def get_token(corpid, secret):
    url = f'https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid={corpid}&corpsecret={secret}'
    return json.loads(urllib.request.urlopen(url, timeout=15).read())['access_token']

def get_template_options(token, template_id):
    """获取请假模板的控件ID和请假类型选项"""
    url = f'https://qyapi.weixin.qq.com/cgi-bin/oa/gettemplatedetail?access_token={token}'
    payload = json.dumps({'template_id': template_id}).encode('utf-8')
    req = urllib.request.Request(url, data=payload, headers={'Content-Type': 'application/json'})
    resp = json.loads(urllib.request.urlopen(req).read())

    tc = resp['template_content']
    result = {
        'controls': {},         # {control_id: {title, control, ...}}
        'leave_types': {},      # {请假类型中文名: vacation_id (int)}
        'leave_types_by_id': {} # {vacation_id: 请假类型中文名}
    }

    # Vacation 模板的请假类型在 vacation_list 中
    if 'vacation_list' in resp:
        for item in resp['vacation_list']['item']:
            name = item['name'][0]['text']
            vid = item['id']
            result['leave_types'][name] = vid
            result['leave_types_by_id'][vid] = name

    for ctrl in tc['controls']:
        prop = ctrl['property']
        result['controls'][prop.get('id', '')] = {
            'title': prop.get('title', '') if isinstance(prop.get('title'), str) else prop.get('title', [{}])[0].get('text', ''),
            'control': prop.get('control', ''),
            'id': prop.get('id', '')
        }
    return result

# 使用示例：
# opts = get_template_options(TOKEN, TEMPLATE_ID)
# opts['leave_types'] = {'年假': 1, '事假': 2, '病假': 3, '调休假': 4, ...}
# opts['controls'] = {'vacation-1563793073898': {title:'请假类型', control:'Vacation', ...}, ...}
```

**请假模板典型控件结构（以实际模板为准）：**

| 控件 | 说明 |
|------|------|
| `Vacation` (特殊内置类型) | 请假类型（年假/事假/病假等），id 固定为 `vacation-xxx` |
| `Textarea` | 请假原因 |
| `Text` / `Textarea` | 紧急联系人、工作交接人（如模板中存在） |

> **注意**：不同的请假模板控件 ID 和结构可能不同。必须通过 `gettemplatedetail` 实时获取，然后根据控件 title 匹配对应字段。

#### 3b. 提交审批 — Vacation 控件格式

> **⚠️ 关键：请假模板是特殊类型（control=Vacation），不是常规的 Selector + Date 控件组合。**
> 请假类型的选项来自 `gettemplatedetail` 返回值中的 `vacation_list`（而非 `controls[].config.selector.options`）。
> 时间范围使用 `attendance.date_range` 格式，`type=1` 表示请假。
>
> **审批人固定为 your_name_here（自审）。**

```python
import json, urllib.request

TOKEN = '（通过 corpid + secret 获取）'
TEMPLATE_ID = '你的请假审批模板ID'
CREATOR = '发起人UserID（默认 default_creator=your_name_here；帮他人请假时覆盖为指定 UserID）'

# ===== 请假类型 vacation_id（来自 Step 3a opts['leave_types']）=====
# 年假=1, 事假=2, 病假=3, 调休假=4, 婚假=5, 产假=6, 陪产假=7, 其他=8
# 所有 vacation_id 必须来自 gettemplatedetail 返回值中的 vacation_list，禁止硬编码

# ===== 时间戳计算示例 =====
# 2026-07-20 ~ 2026-07-22（3天）
# new_begin = 2026-07-20 00:00 UTC+8 = 1784476800
# new_end   = 2026-07-23 00:00 UTC+8 = 1784736000（第3天结束）
# new_duration = 3 * 86400 = 259200

payload = json.dumps({
    'creator_userid': CREATOR,
    'template_id': TEMPLATE_ID,
    'use_template_approver': 0,
    'process': {
        'node_list': [
            # 审批人：your_name_here（自审）
            {
                'type': 1,       # 1=审批人
                'apv_rel': 1,    # 1=会签
                'userid': ['your_name_here']
            }
        ]
    },
    # ⚠️ 使用 process.node_list 时，不要加 notifyer/notify_type（会报 errcode 40058）
    'apply_data': {
        'contents': [
            {
                # Vacation 控件 — 请假类型 + 时间范围（二合一）
                'control': 'Vacation',
                'id': 'vacation-1563793073898',
                'value': {
                    'vacation': {
                        'selector': {
                            'type': 'single',
                            'options': [{
                                'key': vacation_id,  # int：年假=1, 事假=2, ...
                                'value': [{'text': '请假类型中文名', 'lang': 'zh_CN'}]
                            }]
                        },
                        'attendance': {
                            'date_range': {
                                'type': 'halfday',           # halfday=按天
                                'new_begin': 开始时间戳,      # 首日00:00 UTC+8
                                'new_end': 结束时间戳,        # 末日后一日00:00 UTC+8
                                'new_duration': 总秒数        # 天数 * 86400
                            },
                            'type': 1  # 1=请假
                        }
                    }
                }
            },
            {
                # 请假事由（Textarea 控件）
                'control': 'Textarea',
                'id': 'item-1497581399901',
                'value': {
                    'text': '请假原因'
                }
            }
        ]
    },
    'summary_list': [{'summary_info': [{'text': f'请假申请 - {username} - {leave_type}', 'lang': 'zh_CN'}]}]
}, ensure_ascii=False)

result = json.loads(urllib.request.urlopen(
    urllib.request.Request(
        f'https://qyapi.weixin.qq.com/cgi-bin/oa/applyevent?access_token={TOKEN}',
        data=payload.encode('utf-8'),
        headers={'Content-Type': 'application/json'})
).read())
# 成功: {"errcode": 0, "errmsg": "ok", "sp_no": "202607200001"}
```

**⚠️ 格式踩坑记录（不可违反）：**
- **Vacation 控件**：请假模板不是普通的 Selector+Date 模板，而是内置的 Vacation 假勤模板。请假类型来自 `vacation_list`（`item[].id`，int 类型），不是 `controls[].config.selector.options`（string 类型）
- **请假类型 options[].value**：必须是对象数组 `[{"text": "年假", "lang": "zh_CN"}]`，不能是字符串数组 `["年假"]`
- **时间范围**：使用 `attendance.date_range`，`type="halfday"` 表示按天；`new_begin`/`new_end` 为 UTC+8 零点时间戳；`new_duration` 为总秒数（天数 × 86400）
- **审批人配置**：新版 API 使用 `process.node_list` 格式，不要用旧版 `approver` 数组
- **抄送人配置**：`notifyer`/`notify_type` 为旧版参数，**与 `process.node_list` 不兼容**（合并使用报 errcode 40058）。本 Skill 只配审批人，不设抄送人
- **`slice_info`（可选）**：非必填，如果提交报错 `day_items size not valid`，移除 `slice_info` 即可
- 审批人 userid 不是手机号/姓名，是通讯录唯一标识
- `use_template_approver=1` 和 `process.node_list` 不能同时存在
- 接口地址是 `/oa/applyevent`，频率限制 600次/分钟
- **摘要文字必须简洁清晰**，格式如「请假申请 - your_name_here - 年假」
- **IP 白名单（errcode 60020）**：自建应用需在企微管理后台「应用管理 → 自建 → 可信IP」加入调用机出口 IP，否则 `gettemplatedetail`/`applyevent` 报 "not allow to access from your ip"。注意 `gettoken` 不受此限制，会正常返回 token，容易误判为凭据问题。

### Step 4: 返回回执

审批提交成功后，向用户返回：

```
✅ 请假申请已提交！

| 项目 | 内容 |
|------|------|
| 审批单号 | 202607200001（sp_no） |
| 审批人 | your_name_here（自审） |
| 状态 | 待审批 |

请留意企业微信「审批」应用的通知，your_name_here 处理后你会收到结果推送。
```

- 如提交失败，把 `errcode` / `errmsg` 原样告知用户，并对照上方「格式踩坑记录」定位原因（常见：60020 IP 白名单、40058 抄送冲突、not in app visible range 可见范围）。

---

## 📅 工作日 vs 自然日计算模板

```python
from datetime import date, timedelta

def count_workdays(start_str, end_str):
    """计算两个日期之间的工作日天数（排除周六日）"""
    start = date.fromisoformat(start_str)
    end = date.fromisoformat(end_str)
    days = 0
    current = start
    while current <= end:
        if current.weekday() < 5:  # 周一=0, 周日=6
            days += 1
        current += timedelta(days=1)
    return days

def count_natural_days(start_str, end_str):
    """计算自然日天数"""
    start = date.fromisoformat(start_str)
    end = date.fromisoformat(end_str)
    return (end - start).days + 1
```

---

## 完整执行示例

用户说：
> "帮我请 3 天年假，下周一（2026-07-20）开始，回家探亲"

执行流程：
1. **Step 1 收集** → 默认发起人=your_name_here，确认请假类型（年假）、起止日期（07-20 ~ 07-22）、天数计算方式（工作日/自然日）、原因（回家探亲）
2. **Step 2 确认** → 展示确认表格，用户说「确认」（不可省略）
3. **Step 3a** → 调用 `gettemplatedetail` 实时获取请假类型 vacation_id 和控件 ID
4. **Step 3b** → 调用 `applyevent` 提交，`creator_userid=your_name_here`（默认），审批人=your_name_here（自审）
5. **Step 4 回执** → 返回 sp_no，提示留意企微审批通知

---

## 关键资源

- 不使用腾讯文档（`use_smart_table: false`）
- 企业微信请假审批模板 ID: `your_template_id_here`
- **发起人+审批人（固定）：** your_name_here（自审）
- Corpid: `your_corpid_here`
- API: `POST /cgi-bin/oa/applyevent`
- **⚠️ 模板选项必须实时获取，禁止硬编码！提交前必须调用：**
  ```
  POST /cgi-bin/oa/gettemplatedetail  # 传入 template_id
  ```
  - 请假类型 → 从 `vacation_list[].item[].id` 获取
  - 所有控件 ID → 从 `controls[].property.id` 获取
  - **每次请假都重新调用**，因为模板可能被管理员修改，缓存会过时。
