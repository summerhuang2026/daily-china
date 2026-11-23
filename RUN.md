# 每日运行流程（定时任务读取本文件并照做）

## 0. 确定参数
- 运行环境可能是 UTC 时区：先用 `TZ=Asia/Shanghai date` 取得北京时间，运行日期 D = 北京时间当天日期。
- 默认窗口：D-1 07:00 至 D 07:00（北京时间）；默认事件："无特定事件，只要近期新闻"。
- 读 overrides.yaml：若有 run_date == D 的条目，用它的 window / companies 事件 / only 覆盖默认值。
- 读 data/companies.yaml、data/portfolio.yaml。

## 1. 逐家公司（可并行，每家一个子任务）
中间结果写到 reports/_work/<id>.md（不上传 GitHub）。
对 companies.yaml 中的每家公司（或 only 指定的公司）：
1. 用该公司资料和窗口填好 prompts/01_research.md，执行（使用网页搜索），得到新闻条目。
   - media 字段里的 media_mainland / media_hk 展开为 defaults 中的媒体列表。
   - confusable 为空时，删掉"注意"那一行。
2. 把 01 的结果和 portfolio.yaml 交给 prompts/02_impact.md，得到影响分析（不再搜索）。
3. 把 01、02 交给 prompts/03_summary_en.md，得到英文摘要（不再搜索）。
4. 若发现某回购新闻而 shares_outstanding 为空：在报告中注明，不要自行编造。

## 2. 合成一个报告文件：reports/D.md

```
# Greater China Daily — D
窗口：<start> ~ <end>（北京时间）｜覆盖：<n> 家｜特殊设置：<无 / 说明>

## 今日中等以上影响一览
| 公司 | 新闻 | 综合等级 | 方向 |
（没有则写"今日无中等以上影响的新闻"）

---

## <中文名> <English name>（<tickers>）

### 新闻 1：<标题>
**中文摘要**：<事实摘要>
来源：<原发来源>，<发布时间>，<链接>；另见：<…>

**影响分析**
<02 的输出>

**English summary**：<03 的段落；综合等级低于中则写 "— (below medium impact)">

### 新闻 2 …

### 其他
- 影响低于中等：<中文一句话>
- 未找到：<中文列表>
- English: <03 的合并一句话>

---
## <下一家公司> …
```

公司顺序与 companies.yaml 一致。所有公司都写在同一个文件里。

## 3. 保存
- git add reports/D.md，commit 信息 "Daily report D"，直接推送到 main 分支。
- 若某家公司失败：在报告该公司位置写明失败原因，其他公司照常完成，不要整份放弃。
