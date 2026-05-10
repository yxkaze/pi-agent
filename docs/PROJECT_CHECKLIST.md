# GitHub Project 创建 Checklist

**仓库地址：** https://github.com/yxkaze/pi-agent

---

## ✅ 第一步：创建Project

- [ ] 访问 https://github.com/yxkaze/pi-agent
- [ ] 点击顶部 "Projects" 标签
- [ ] 点击 "New project" 按钮
- [ ] 选择 "Team planning" 模板
- [ ] 名称：`Pi Agent Development`
- [ ] 描述：`Pi Agent Core开发跟踪 - 基于eino框架的Go语言Agent系统`
- [ ] 点击 "Create project"

---

## ✅ 第二步：添加自定义字段

点击右上角 "+" → "New field"

### 字段1：Phase
- [ ] Field name: `Phase`
- [ ] Type: `Single select`
- [ ] Options: `Phase 1`, `Phase 2`, `Phase 3`, `Phase 4`

### 字段2：Priority
- [ ] Field name: `Priority`
- [ ] Type: `Single select`
- [ ] Options: `P0`, `P1`, `P2`, `P3`

### 字段3：Size
- [ ] Field name: `Size`
- [ ] Type: `Single select`
- [ ] Options: `XS`, `S`, `M`, `L`, `XL`

---

## ✅ 第三步：创建Issue

### Issue #1
- [ ] 标题：`[FEATURE] 实现Provider注册中心`
- [ ] 复制内容从：`docs/QUICK_START_PROJECT.md`
- [ ] 标签：`feature`, `phase-1`, `provider`
- [ ] Phase: Phase 1
- [ ] Priority: P0
- [ ] Size: M

### Issue #2
- [ ] 标题：`[PROVIDER] 添加OpenAI Provider支持`
- [ ] 复制内容从：`docs/QUICK_START_PROJECT.md`
- [ ] 标签：`provider`, `openai`, `phase-1`
- [ ] Phase: Phase 1
- [ ] Priority: P0
- [ ] Size: M

### Issue #3
- [ ] 标题：`[PROVIDER] 添加百度千帆Provider支持`
- [ ] 复制内容从：`docs/QUICK_START_PROJECT.md`
- [ ] 标签：`provider`, `qianfan`, `phase-1`
- [ ] Phase: Phase 1
- [ ] Priority: P1
- [ ] Size: M

### Issue #4
- [ ] 标题：`[FEATURE] 实现Message Builder`
- [ ] 复制内容从：`docs/QUICK_START_PROJECT.md`
- [ ] 标签：`feature`, `message`, `phase-1`
- [ ] Phase: Phase 1
- [ ] Priority: P1
- [ ] Size: S

---

## ✅ 第四步：创建视图

- [ ] 视图1：`By Phase` (Group by Phase)
- [ ] 视图2：`By Priority` (Group by Priority)
- [ ] 视图3：`Roadmap` (Layout: Roadmap)

---

## ✅ 第五步：添加到看板

对每个Issue：
- [ ] Status: `Todo`
- [ ] 设置Phase、Priority、Size字段
- [ ] 拖动到对应的列

---

## 📋 所有内容准备就绪

**详细步骤和Issue内容请查看：**
```
docs/QUICK_START_PROJECT.md
```

**准备好了吗？开始操作！** 🚀
