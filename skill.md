---
name: hr-query
description: 德创数字人事库查询。当用户询问德创数字公司的员工信息、部门信息、人事信息时触发，如「营销中心有几个人」「查一下智能装备部」「德创有多少员工」「某部门有哪些人」「某人是哪个部门的」「德创人事库」等。所有关于德创公司员工信息的查询都应该使用此 skill。Also triggers when user shares or references a CSV/Excel file about employee data and wants to search or analyze it.
---

# HR Query Skill

此 skill 用于查询德创数字人事库 CSV 文件，支持按部门、岗位、姓名等维度查询员工信息。

## 文件路径

**默认文件**：`D:\claude code\德创数字花名册20260522.csv`

此文件包含完整的 24 列信息（含合同到期日期）。如需使用其他版本，请用户明确指定路径。

用户可通过绝对路径指定其他文件。

## 文件编码

CSV 文件编码为 GB18030（简体中文 Windows 编码）。读取时必须先转换为 UTF-8：

```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null
```

## 基本查询方法

用 `iconv` 读取 CSV 内容后，使用 grep 过滤相关行：

```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "关键词"
```

**注意**：iconv 命令在 grep 之前完成转换，不要在 grep 中调用 iconv。

## 查询模式

### 1. 按一级部门查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "营销中心"
```

### 2. 按二级部门查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "智能装备部"
```

### 3. 按姓名查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "王"
```

### 4. 按岗位关键词查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "算法岗"
```

### 5. 按板块/备注查询（如智能装备板块）
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "智能装备板块"
```

### 6. 按入职年份查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep ",2023/"
```

### 7. 按合同到期年份查询
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep ",2026/5/"
```

### 8. 统计人数
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep -c "关键词"
```

### 9. 计算月度薪资总额
```bash
iconv -f GB18030 -t UTF-8 "<file_path>" 2>/dev/null | grep "关键词" | awk -F',' '{sum += $7} END {print sum}'
```
字段 $7 为月度薪资标准。

### 10. 计算年度薪资总额（13薪）
月度薪资总额 × 13 = 年度薪资总额

## 输出格式规范

### 人数统计
返回人数和名单表格：
```
**营销中心共 15 人：**

| 序号 | 姓名 | 二级部门 | 岗位 | 月薪 |
|------|------|----------|------|------|
| 8 | 席喜峰 | 质量交付部 | 副主任 | 16,760 |
...
```

### 薪资统计
返回人数、月度薪资总额、年度薪资总额（13薪）。

### 合同到期查询
返回合同到期人员名单，包含合同到期日期。

## 可视化报告生成

当用户需要图形化输出时，可生成 HTML 交互式报告或 PNG 静态图片。

### HTML 交互式报告
生成包含 ECharts 图表的 HTML 文件，保存在 CSV 文件同目录下：
- 文件名格式：`{查询主题}分析报告.html`
- 包含：统计数据卡片、饼图（部门分布）、柱状图（年份分布/薪资分布）、人员明细表
- 深色主题，适合汇报展示

### PNG 静态图片
使用 matplotlib 生成图表，保存在 CSV 文件同目录下：
- 文件名格式：`{查询主题}分析图.png`
- 包含：饼图 + 柱状图组合
- 适合直接发送/嵌入文档

### 生成条件
当用户明确要求"生成图表"、"可视化"、"图形化"时，或查询结果人数≥10人时，自动生成可视化报告。

## HTML 报告模板

生成 HTML 文件使用 ECharts 5.x（CDN 引入），深色主题：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    <style>
        body { font-family: "Microsoft YaHei"; background: linear-gradient(135deg, #1a1a2e, #16213e); color: #fff; padding: 40px; }
        .container { max-width: 1200px; margin: 0 auto; }
        h1 { color: #00d4ff; text-align: center; }
        .stats-row { display: flex; gap: 20px; margin: 30px 0; }
        .stat-card { flex: 1; background: rgba(255,255,255,0.1); border-radius: 16px; padding: 24px; text-align: center; }
        .stat-card .number { font-size: 36px; color: #00d4ff; font-weight: bold; }
        .chart-row { display: flex; gap: 20px; margin: 30px 0; }
        .chart-card { flex: 1; background: rgba(255,255,255,0.08); border-radius: 16px; padding: 20px; }
        .chart { height: 300px; }
        table { width: 100%; border-collapse: collapse; background: rgba(255,255,255,0.05); border-radius: 12px; }
        th, td { padding: 12px 16px; text-align: left; border-bottom: 1px solid rgba(255,255,255,0.1); }
        th { background: rgba(0,212,255,0.2); }
    </style>
</head>
<body>
    <div class="container">
        <h1>{报告标题}</h1>
        <div class="stats-row">
            <div class="stat-card"><div class="number">{总人数}</div><div>总人数</div></div>
            <div class="stat-card"><div class="number">{月度薪资总额}</div><div>月度薪资总额 (元)</div></div>
            <div class="stat-card"><div class="number">{年度薪资总额}</div><div>年度薪资总额 (元)</div></div>
        </div>
        <div class="chart-row">
            <div class="chart-card"><h3>部门分布</h3><div id="deptChart" class="chart"></div></div>
            <div class="chart-card"><h3>年份分布</h3><div id="entryChart" class="chart"></div></div>
        </div>
        <table><thead><tr><th>序号</th><th>姓名</th><th>部门</th><th>岗位</th><th>月薪</th></tr></thead>
        <tbody>{人员明细}</tbody></table>
    </div>
    <script>
        // ECharts 初始化和配置
    </script>
</body>
</html>
```

## PNG 图片生成脚本

使用 Python matplotlib 生成：
```python
import matplotlib.pyplot as plt
plt.rcParams['font.sans-serif'] = ['Microsoft YaHei']
plt.rcParams['axes.unicode_minus'] = False
fig, axes = plt.subplots(1, 2, figsize=(14, 5))
fig.patch.set_facecolor('#1a1a2e')
# 饼图 + 柱状图组合，保存为 PNG
plt.savefig('{output_path}', dpi=150, facecolor='#1a1a2e', bbox_inches='tight')
```

## 工作流程

1. 从用户输入中识别查询意图
2. 确定搜索关键词并执行查询
3. 收集数据（人数、薪资、部门分布、入职年份等）
4. 根据查询类型返回格式化结果
5. 若需可视化，生成 HTML/PNG 文件到 CSV 同目录

## CSV 字段说明（完整版24列）

| 字段位置 | 字段名 | 说明 |
|----------|--------|------|
| 1 | 序号 | 员工编号 |
| 2 | 姓名 | 员工姓名 |
| 3 | 一级部门 | 如：经营层、营销中心、创研中心等 |
| 4 | 二级部门 | 如：质量交付部、智能制造部等 |
| 5 | 岗位名称2 | 具体岗位 |
| 6 | 入职时间 | 格式：YYYY/M/D |
| 7 | 月度薪资标准 | 单位：元 |
| 8 | 年度薪资总额 | 单位：元 |
| 9 | 年度薪资（含公司成本） | 单位：元 |
| 10 | 备注 | 如：智能装备板块、借调说明等 |
| 11 | 合同签署 | 合同甲方公司 |
| 12 | 合同开始时间 | 格式：YYYY/M/D |
| 13 | 合同结束时间 | 格式：YYYY/M/D（合同到期查询用此字段） |
| 14 | 员工状态 | 如：已转正 |
| 15 | 性别 | 男/女 |
| 16 | 民族 | 如：汉 |
| 17 | 政治面貌 | 如：中共党员、群众 |
| 18 | 年龄 | 整数 |
| 19 | 籍贯 | 省市 |
| 20 | 最高学历 | 如：大学本科、硕士研究生 |
| 21 | 最高学历毕业院校 | 学校名称 |
| 22 | 最高学历毕业专业 | 专业名称 |
| 23 | 最高学历毕业时间 | 格式：YYYY/M/D |
| 24 | 职称/职业资格证书名称 | 证书名称 |

## 常见一级部门

- 经营层
- 营销中心 / 营销业务部
- 智能制造中心
- 创研中心 / 智慧运力部 / 智能装备部
- 财务中心
- 综合运营中心
- 产业金融中心
- 德创数银
- 智联远行
- 智联绿能

## 工作流程

1. 从用户输入中识别查询意图（查人数、查人员、查部门、查薪资、查合同等）
2. 确定要搜索的关键词（部门名、姓名、岗位、入职年份、合同到期月份等）
3. 用 iconv 读取 CSV 文件（GB18030 → UTF-8）
4. 用 grep 过滤相关行
5. 必要时用 awk 计算薪资总额
6. 根据查询类型返回格式化结果（人数统计表、薪资统计、合同到期列表）