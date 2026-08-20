---
id: checklist.before-finish
type: checklist
status: active
when: always
when_to_use: 声称完成一轮开发或调研前
related: [rule.common.quality]
---

# 结束前

- [ ] 用户任务已满足，没有用额外工作替换原目标
- [ ] 已按当前类型包做验证（Android 已接真机则含安装/截图/Logcat；桌面能启动则含打开应用；否则已写明缺口）
- [ ] 没有提交密钥、无关格式化、范围外重构
- [ ] 用户未要求则未 commit / push
- [ ] 若写了知识库文档：路径正确、frontmatter 完整、索引（Hub）已更新
- [ ] 若做了开源深研：源码仍只在 Temp，总结已按能力层归档
- [ ] 若是架构/分层改动：已完成 [architecture-review.md](architecture-review.md)
