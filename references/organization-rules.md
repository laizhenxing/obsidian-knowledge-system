# 整理规则

## 指令映射

- “记录一下……”：使用快速记录结构写入 `00-收件箱`，`type: note`、`status: inbox`，不强迫用户先分类。
- “整理知识库”：处理全部可可靠归类的收件箱内容，并完成快照、链接、主题索引、日志和验证。
- “开始今天”：按核心插件配置创建或打开今日笔记，列出活跃项目及 `next_action`。
- “今日收尾”：整理当天记录并更新每日笔记，不把未完成任务写成完成。
- “更新 xxx 项目”：读取项目主页和相关记录，更新进展、资料链接与唯一下一步行动。
- “生成本周复盘”：根据本周每日记录、项目记录和新增知识生成周复盘，所有结论附回链。

## 内容映射

| 内容 | 目标目录 | `type` | 状态与专用属性 |
|---|---|---|---|
| 项目 | `10-项目` | `project` | `status: active`，保留一个明确的 `next_action` |
| 领域 | `20-领域` | `area` | 使用基础属性维护状态；首期无专用模板 |
| 知识卡片 | `30-知识/01-知识卡片` | `note` | 整理后为 `status: active`，回链原始资料 |
| 主题索引 | `30-知识/02-主题索引` | `moc` | 整理后为 `status: active` |
| 阅读资料 | `40-资源/阅读学习` | `resource` | 整理后为 `status: active`，补齐 `author`、`source` |
| 每日记录 | `50-日志/每日` | `daily` | 按每日模板维护状态 |
| 每周复盘 | `50-日志/每周` | `weekly` | 按每周模板维护状态 |
| 每月复盘 | `50-日志/每月` | `monthly` | 按每月模板维护状态 |
| 归档内容 | `99-归档` | 保留原类型 | `status: archived` |
| 公司研究 | `40-资源/公司研究报告` | 保留既有类型 | 保留既有状态，不统一改成 `active` |

按上表补齐适用的 `created`、`updated`、`topics`、`source` 等属性，不对所有内容统一设置类型或状态。项目完成时使用 `status: done`；只有归档时才使用 `status: archived` 并移入 `99-归档`。

## 每批整理流程

### 1. 建立批次、影响清单、before 快照和链接基线

1. 确认 `obsidian vaults` 可用，并检查 `git status --short`。生成批次 ID 和日志路径；若同名日志已存在，等待下一分钟重新生成，不覆盖旧日志。

   ```bash
   BATCH_ID="$(date '+%Y-%m-%d-%H-%M')"
   LOG="90-系统/整理日志/$BATCH_ID.md"
   test ! -e "$LOG"
   ```

2. 建立本批原路径、目标路径和计划主题清单。对每个计划移动或重命名的源笔记，在任何路径变更前用 Obsidian CLI 枚举既有反链；即使输出为空也要记录检查结果：

   ```bash
   obsidian backlinks path='<源笔记路径>' format=json
   ```

   将返回 JSON 中每个 `file` 路径合并为本批“预期反链”路径，纳入影响清单；不得只记录源路径和目标路径。
3. 保护 before 之前的内容和用户已有变更。工作区没有未提交知识变更时，直接执行 `before="$(git rev-parse HEAD)"`，以当前 HEAD 为整理前快照，不创建空提交。只有整理前已经存在但未提交的知识变更才单独暂存并提交为 `chore: snapshot before knowledge organization`，再将该提交哈希记为 `before`；提交必须逐个列出明确的知识路径，包括已有未提交变更的预期反链路径，不包含日志、`.obsidian` 个人状态或无关用户变更。其他已有变更保持原样并记录，不使用 `git commit -a`。
4. 从 `90-系统/模板/整理日志.md` 创建 `$LOG`，先记录批次 ID、before 和本批范围。逐项记录原路径、目标路径、预期反链、计划主题，以及原路径和预期反链路径在 `before` 中是否存在；只处理清单内内容。
5. 在任何正文、属性或路径变更前，保存整理前未解析链接基线。临时目录必须位于库外，成功或失败复验后都要清理。

   ```bash
   TMP_DIR="$(mktemp -d)"
   obsidian unresolved format=json > "$TMP_DIR/unresolved-before.json"
   ```

### 2. 整理内容

1. 逐篇读取收件箱内容，按“内容映射”判断项目、领域、知识卡片、主题索引、阅读资料或日志；证据不足时留在收件箱并提出具体问题。
2. 每次移动或重命名前，再次对该源笔记执行 `obsidian backlinks path='<源笔记路径>' format=json`。若反链集合变化，先更新影响清单和日志，再继续；新增反链同样纳入 before 存在性记录、提交和恢复范围。
3. 移动笔记并按映射设置类型、状态和专用属性。公司研究留在 `40-资源/公司研究报告`，遵守其归档与使用说明，不统一改成 `active`。
4. 可以压缩重复表述、重组正文和拆分独立观点；保留事实、来源和原意，新卡片回链原始资料。
5. 搜索相关笔记，在正文首次有语义价值的位置添加双链；只更新持续关注的主题索引。
6. 持续记录新增、移动、改写、归档、主动链接、CLI 被动更新的反链和待确认事项，日志不参与下一阶段的知识变更提交。

### 3. 查询四个视图并比较未解析链接

四个视图都位于 `90-系统/Bases/首页视图.base`，必须逐一执行以下确切命令，不能用打开首页代替查询：

```bash
obsidian base:query path='90-系统/Bases/首页视图.base' view='待整理' format=json
obsidian base:query path='90-系统/Bases/首页视图.base' view='进行中' format=json
obsidian base:query path='90-系统/Bases/首页视图.base' view='最近更新' format=json
obsidian base:query path='90-系统/Bases/首页视图.base' view='继续阅读' format=json
```

保存整理后的 JSON，并用以下内联 Python 对前后 JSON 做差集。输出必须包含整理前后数量和 `new` 列表；`new` 非空即验证失败，不得提交知识变更。

```bash
obsidian unresolved format=json > "$TMP_DIR/unresolved-after.json"
python3 - "$TMP_DIR/unresolved-before.json" "$TMP_DIR/unresolved-after.json" <<'PY'
import json
import sys
from pathlib import Path

before_items = json.loads(Path(sys.argv[1]).read_text())
after_items = json.loads(Path(sys.argv[2]).read_text())

def key(item):
    return json.dumps(item, ensure_ascii=False, sort_keys=True)

before = {key(item) for item in before_items}
new = [item for item in after_items if key(item) not in before]
print(json.dumps({
    "before": len(before_items),
    "after": len(after_items),
    "new": new,
}, ensure_ascii=False, indent=2))
if new:
    raise SystemExit("发现新增未解析链接，禁止提交知识变更")
PY
```

将四视图查询结果和 Python 输出写入日志。成功时清理临时文件；失败时先保留基线供恢复复验，复验后再清理：

```bash
rm -- "$TMP_DIR/unresolved-before.json" "$TMP_DIR/unresolved-after.json"
rmdir -- "$TMP_DIR"
```

### 4. 核对 Git 实际变更范围

1. 每次移动或重命名后，以及验证通过后、提交前，检查 Git 实际变更清单：

   ```bash
   git status --porcelain=v1 --untracked-files=all
   git diff --name-only
   git diff --cached --name-only
   ```

2. 将实际清单与“本批原路径、目标路径、预期反链、计划主题”逐项核对；`$LOG` 仅作为独立日志产物处理，批次开始前已记录的其他用户变更保持原样，不纳入本批提交。CLI 自动更新的反链变化必须逐项写入日志的“自动更新反链”。
3. 发现本批新增且超出允许范围的路径时，立即停止成功提交并进入失败恢复，不得通过扩大提交路径夹带额外变更。将额外路径、实际 Git 输出和处置写入失败日志。

### 5. 两阶段提交，消除日志自引用

1. 验证和 Git 范围核对通过后，只逐个暂存本批原路径、目标路径、预期反链和计划主题，包括原路径删除，明确排除 `$LOG`、其他用户变更和 `.obsidian`。用 `git diff --cached --quiet -- <上述明确路径>` 判断是否有内容变更。
2. 有知识变更时，用同一组明确路径提交并取得 after，不在日志提交前预填或猜测哈希：

   ```bash
   git commit -m 'chore: organize knowledge vault' -- <本批原路径、目标路径、预期反链、计划主题>
   after="$(git rev-parse HEAD)"
   ```

   没有知识变更时不创建空提交，日志的 after 字段写 `无内容变更，沿用 before`。
3. 知识变更 after 已存在或确定无内容变更后，补全 `$LOG` 的 before、知识变更 after、影响清单、反链变化、Git 范围核对、四视图结果和未解析链接差集。日志不记录也不依赖自己的提交哈希。
4. 最后只提交日志，与知识变更提交分开：

   ```bash
   git add -- "$LOG"
   git commit -m 'chore: record knowledge organization log' -- "$LOG"
   ```

## 失败恢复

1. 验证、Git 范围核对或整理失败时停止后续操作，设置 `FAILED_LOG="90-系统/整理日志/$BATCH_ID-失败.md"` 和 `ZIP="90-系统/迁移备份/失败整理-$BATCH_ID.zip"`，两者都不得覆盖已有文件。将已创建的 `$LOG` 重命名为 `$FAILED_LOG`；若尚未创建日志，则从模板直接创建 `$FAILED_LOG`。在失败日志中记录失败命令、输出、实际变更清单和待恢复路径，后续始终保留该日志。
2. 先只对本批原路径和预期反链清单中在 `before` 已存在的路径执行定向恢复，再处理新目标；不恢复整个目录或仓库，不改变暂存区：

   ```bash
   git restore --source="$before" --worktree -- <明确本批原路径和预期反链路径>
   ```

   这样可同时还原源笔记和 CLI 被动更新的反链内容，并保留 before 快照中的既有内容和用户已有知识变更。不得触碰 before 之前的提交、无关路径或未纳入本批的用户变更。
3. 仅对“本批新建或移动产生、且在 before 不存在”的未跟踪目标执行 ZIP 隔离。按目标清单逐项核对，不使用全库未跟踪文件列表，也不包含 `$FAILED_LOG`。有这类目标时，从库根目录将每个目标的原相对路径显式传给 `zip`，不得打包整个目录：

   ```bash
   test ! -e "$ZIP"
   zip -r "$ZIP" <明确本批未跟踪目标相对路径>
   unzip -t "$ZIP"
   ```

   ZIP 必须位于 `90-系统/迁移备份`，其中文件保持原相对路径。`unzip -t` 成功后才可继续；验证失败时保留库内原文件并记录错误，不得移除副本。
4. 只有 ZIP 完整性验证通过后，才能移除已打包目标的库内未跟踪副本：

   ```bash
   rm -- <已打包且通过unzip -t的本批未跟踪目标副本>
   ```

   此处的移除是“移入压缩隔离备份”：内容仍保存在已验证 ZIP 中，不是永久删除。不得移除未打包、未验证或不在本批目标清单中的文件。
5. 不得使用 `git reset --hard`、`git clean`、`obsidian delete permanent` 或任何永久删除来恢复。临时 JSON 只能按上一节的明确路径清理。
6. Obsidian 对外部文件变更的索引刷新可能是异步的。隔离后每秒重新生成 `unresolved-after.json` 并运行同一内联 Python 差集，最多轮询 30 秒，直到 `new` 为空；首次查询仍显示已删除文件的旧链接时不得将其当作最终结果。超时仍非空则记录复验失败并保留现场，不得清理基线或提交成功结论。链接差集通过后再重新执行四个 `base:query` 命令，确保隔离文件不再污染视图与链接结果。将原路径与反链恢复结果、ZIP 路径、`unzip -t` 结果、索引轮询和复验输出写入 `$FAILED_LOG`，再按明确路径清理临时文件。
7. 恢复和复验完成后单独提交 `$FAILED_LOG`。失败日志的知识变更 after 写 `整理失败，无 after；已恢复到 before`，不得把失败日志提交哈希写成知识变更 after。

## 边界

- 不安装常驻服务，不自动永久删除知识内容。
- 疑似重复但语义不同的笔记保留并互相链接，不强行合并。
- 冲突文件和无法识别的附件保持原样并记录。
- 公司研究结论、券商预测和外部来源不得由整理流程补写或改写。
