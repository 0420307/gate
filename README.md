# hezhanleiok/gate 同步 jerylihub/gate 最新版修改说明

## 一、结论

`hezhanleiok/gate` 是你那份旧代码的独立仓库，落后于 `jerylihub/gate` 两个版本，而且它比旧版还缺一个「30 分钟定时」。

你「URL 粘贴可用」是在 `jerylihub/gate` 那边，这份 `hezhanleiok/gate` 目前并不具备这个能力。

---

## 二、诊断：hezhanleiok/gate 缺什么

| 缺口 | 位置 | 影响 |
|---|---|---|
| 没有 `nodes.txt` 生成 | `vpngate.py` `write_outputs()` | 没有纯节点版，谈不上 URL 自动轮换 |
| 没有 `NODES_URL` 常量 + 提示日志 | `vpngate.py` | 同上 |
| **没有 30 分钟定时** | `check.yml` `on:` 只有 `workflow_dispatch` | 就算生成了也不会每 30 分钟自动更新 |
| URL 写死指向 `jerylihub` | `vpngate.py` 3 处 + `check.yml` 1 处 | 生成的链接/注释指向别人的站 |
| 说明文档是旧的（只有手动粘贴） | `README.md` | 没有 URL 轮换教程 |

---

## 三、修改总览

共修改 **3 个文件，13 处**：

| 文件 | 处数 | 说明 |
|---|---:|---|
| `vpngate.py` | 6 | 增加 `NODES_URL`、生成 `nodes.txt`、日志、替换 URL |
| `.github/workflows/check.yml` | 2 | 增加 30 分钟定时、替换站点 URL |
| `README.md` | 5 | 核心价值、架构图、配置表、使用教程、速查表 |

另外，全局替换 `jerylihub` → `hezhanleiok` 共 **5 处**：

- `vpngate.py`：`CHAIN_URL`、`HOSTS_URL`、`NODES_URL`、`SUB_URL`
- `check.yml`：`Show site URL` 那行

---

## 四、方案 A（推荐，最省事）：同步 jerylihub 最新版 + 全局替换

因为所有这些改动在 `jerylihub/gate` 已经做完，你直接做两件事即可：

1. 从 `jerylihub/gate` 下载最新：
   - `vpngate.py`
   - `.github/workflows/check.yml`
   - `README.md`

   覆盖到 `hezhanleiok/gate`。  
   可以用 GitHub 网页编辑器删旧贴新，或本地替换后 push。

2. 全局把 `jerylihub` 换成 `hezhanleiok`。**共 5 处**：
   - `vpngate.py`：`CHAIN_URL`(410)、`HOSTS_URL`(469)、`NODES_URL`(470)、`SUB_URL`(529) 4 处
   - `check.yml`：`Show site URL` 那行 1 处

其余：

- `EDT_DOMAIN`：`ed.xiaolei.qzz.io`
- `EDT_UUID`
- `EDGE_HOSTS`
- `CHECK_WORKER`：`check.helei.kdns.fr`

都是你自己的值，**不用动**。

---

## 五、方案 B（手工改）：3 个文件，共 13 处

### 5.1 `vpngate.py`（706 行，6 处）

#### 第 410 行：`CHAIN_URL`

`jerylihub` → `hezhanleiok`：

```python
CHAIN_URL = os.environ.get("CHAIN_URL", "https://hezhanleiok.github.io/gate/chains.txt")
```

#### 第 469 行：`HOSTS_URL` 改域名，并在其后新增一行 `NODES_URL`

```python
HOSTS_URL = os.environ.get("HOSTS_URL", "https://hezhanleiok.github.io/gate/hosts.txt")
NODES_URL = os.environ.get("NODES_URL", "https://hezhanleiok.github.io/gate/nodes.txt")
```

#### 第 528 行：`SUB_URL`

`jerylihub` → `hezhanleiok`：

```python
SUB_URL = os.environ.get("SUB_URL", "https://hezhanleiok.github.io/gate/sub.txt")
```

#### `write_outputs()`：插入生成 `nodes.txt` 的 6 行，并修改 return

在第 633 行之后、第 635 行 sub 注释之前插入：

```python
    # 纯节点版(无注释): 把 URL 填进 edgetunnel「自定义优选IP」框, 客户端刷新订阅即自动轮换
    nodes_path = os.path.join(PUBLIC_DIR, "nodes.txt")
    nodes_lines = [ln for ln in build_hosts_text(data).split("\n") if ln and not ln.startswith("#")]
    with open(nodes_path, "w", encoding="utf-8") as f:
        f.write("\n".join(nodes_lines) + ("\n" if nodes_lines else ""))
```

第 639 行 `return` 改为：

```python
    return data_path, html_path, chains_path, hosts_path, nodes_path, sub_path
```

#### `main()`：解包加 `nodes_path`，日志加一行

第 691 行解包改为：

```python
    data_path, html_path, chains_path, hosts_path, nodes_path, sub_path = write_outputs(data)
```

第 695 行后加一行日志：

```python
    log("WEBSITE", f"生成 {os.path.relpath(nodes_path, REPO_DIR)}")
```

---

### 5.2 `.github/workflows/check.yml`（71 行，2 处）

#### 第 3–4 行：`on:` 增加 30 分钟定时

现在只有手动触发，改为：

```yaml
on:
  # 定时检测: 默认每 30 分钟一次
  schedule:
    - cron: "*/30 * * * *"
  workflow_dispatch:
```

#### 第 71 行：`Show site URL`

`jerylihub` → `hezhanleiok`：

```yaml
        run: 'echo "site: https://hezhanleiok.github.io/gate/ (deploy outcome: ${{ steps.deployment.outcome }})"'
```

---

### 5.3 `README.md`（约 220 行，5 处整体替换）

| 处 | 位置 | 改什么 |
|---|---|---|
| 1 | 第 5 行「核心价值」 | 「定期打开 URL 复制粘贴」→「**填一次 URL 即自动轮换**」 |
| 2 | 第 22–45 行「架构图」 | 加 `nodes.txt` 及 4 产物数据流（照 `jerylihub/gate` 最新版抄） |
| 3 | 第 3 步配置表 | URL 那行加 `NODES_URL` |
| 4 | 「二、使用教程」 | 单方式 → 双方式（方式一 URL 轮换 / 方式二手动粘贴） |
| 5 | 「四、配置速查表」 | 加 `NODES_URL / HOSTS_URL / CHAIN_URL / SUB_URL` 一行 |

> README 这 5 处直接照 `jerylihub/gate` 的最新 README 抄最快，内容一字不差地搬过去即可。

---

## 六、全局替换清单：`jerylihub` → `hezhanleiok`

| 文件 | 常量 / 位置 | 约行号 | 修改后 |
|---|---|---|---|
| `vpngate.py` | `CHAIN_URL` | 410 | `https://hezhanleiok.github.io/gate/chains.txt` |
| `vpngate.py` | `HOSTS_URL` | 469 | `https://hezhanleiok.github.io/gate/hosts.txt` |
| `vpngate.py` | `NODES_URL` | 470 | `https://hezhanleiok.github.io/gate/nodes.txt` |
| `vpngate.py` | `SUB_URL` | 529 | `https://hezhanleiok.github.io/gate/sub.txt` |
| `check.yml` | `Show site URL` | 71 | `https://hezhanleiok.github.io/gate/` |

---

## 七、计数汇总

- `vpngate.py`：6 处
- `check.yml`：2 处
- `README.md`：5 处

合计：**13 处**，分布在 **3 个文件**。

另外全局替换：`jerylihub` → `hezhanleiok`，共 **5 处**。

---

## 八、注意事项

1. `CHECK_WORKER` 通过 workflow 环境变量传给脚本，会覆盖 `vpngate.py` 里的默认值，所以检测 Worker 域名只需在 workflow 里改一处。
2. `EDT_DOMAIN` / `EDT_UUID` / `EDGE_HOSTS` 是 `vpngate.py` 里的默认值，直接改源码。
3. 当前只修改说明文件，其他文件先不修改。
4. 如果采用方案 A，替换完 5 处 URL 后，直接手动触发一次 Action，确认 `nodes.txt` 能生成、`check.yml` 的 cron 存在即可。

---

## 九、待办 Checklist

- [ ] 确认采用方案 A 还是方案 B
- [ ] 同步 `vpngate.py`
- [ ] 同步 `.github/workflows/check.yml`
- [ ] 同步 `README.md`
- [ ] 全局替换 `jerylihub` → `hezhanleiok` 共 5 处
- [ ] 确认 `NODES_URL` 已加入 `vpngate.py`
- [ ] 确认 `write_outputs()` 会生成 `nodes.txt`
- [ ] 确认 `main()` 日志包含 `nodes.txt`
- [ ] 确认 `check.yml` 有 `*/30 * * * *`
- [ ] 手动触发 Action，验证成功
- [ ] 打开 `hosts.txt` 和 `nodes.txt` 检查产物
