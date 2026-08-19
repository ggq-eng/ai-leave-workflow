# ai-leave-workflow

> 来源分类：**待确认** ｜ 导出批次：review

AI 智能请假流水线。用企业微信 API 实现端到端请假自动化：收集→确认→提交→回执。默认发起人+审批人均为 your_name_here（自审）。已配置可信IP，无需预配成员。

## 安装

把本文件夹整体复制到 WorkBuddy 技能目录：

```bash
cp -r . ~/.workbuddy/skills/ai-leave-workflow        # 用户级
# 或
cp -r . <项目>/.workbuddy/skills/ai-leave-workflow   # 项目级
```

重启/刷新 WorkBuddy 后即可在对话中触发。

## 说明

- 本技能从本地 WorkBuddy 环境导出，**所有真实密钥已脱敏为占位符**，使用前请配置你自己的 API Key。
- 若来自技能市场（文件夹名以 `__skillhub` 结尾），版权归原作者，请遵守其许可证。
