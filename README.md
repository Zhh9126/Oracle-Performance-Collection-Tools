
# Oracle 性能诊断工具箱 (AWR/ASH/SQL Trace)

一个用于 Oracle 数据库的交互式性能诊断 Python 脚本，旨在简化 AWR、ASH 报告生成以及 SQL Trace 的开启与收集流程。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Python](https://img.shields.io/badge/Python-3.6%2B-blue)
![Oracle](https://img.shields.io/badge/Oracle-11g%20%7C%2012c%20%7C%2019c-red)

## 📖 项目简介

在 Oracle 数据库日常运维中，生成 AWR/ASH 报告和开启 SQL Trace 是排查性能问题的核心手段。然而，频繁连接数据库、查找快照、处理不同版本的函数签名差异往往耗时且容易出错。

本工具箱提供统一的命令行交互界面，**自动处理 Oracle 11g 至 19c 的版本兼容性问题**，让你只需几步简单的输入即可获取标准的 HTML/文本格式诊断报告，或快速开启会话级跟踪。

## ✨ 主要功能

- **📊 AWR 报告生成**
  - 交互式列出可用快照，支持按时间过滤。
  - 自动校验快照 ID 合法性（非空、数字、结束大于起始）。
  - **智能检测数据库重启**：若所选区间内发生过重启，将主动警告并拒绝生成无效报告。
  - 支持生成 HTML 或纯文本格式报告。

- **⚡ ASH 报告生成**
  - 支持绝对时间（`YYYY-MM-DD HH24:MI:SS`）和相对时间（如 `15m`, `2h`）输入。
  - 兼容 Oracle 11g/12c/19c 不同版本的 `DBMS_WORKLOAD_REPOSITORY.ASH_REPORT_*` 函数签名。
  - 自动获取 DBID 和实例编号，无需手动干预。

- **🔍 SQL Trace 辅助工具**
  - 支持三种跟踪级别：
    - **当前会话跟踪**（`ALTER SESSION` / `DBMS_SESSION`）。
    - **指定会话跟踪**（通过 SID 和 SERIAL# 使用 `DBMS_SYSTEM`）。
    - **数据库实例级跟踪**（使用 `DBMS_MONITOR`，带有危险操作警告）。
  - 自动查询并显示 Trace 文件生成路径。
  - 内置 `tkprof` 格式化工具使用说明。

- **🛡️ 高兼容性 & 智能容错**
  - 完全兼容 **Oracle 11g、12c、19c**。
  - 修复了 `PLS-00302`、`ORA-20019`、`DBMS_OUTPUT` 缓冲区溢出等常见版本兼容性问题。
  - 支持 `oracledb` 和 `cx_Oracle` 两种 Python 驱动。

## 📦 环境要求

- **Python**: 3.6 或更高版本
- **Oracle 客户端**: 需安装 Oracle Instant Client 或已配置 `ORACLE_HOME` 环境变量。
- **Python 依赖**: `oracledb` (推荐) 或 `cx_Oracle`

## 🚀 快速开始

### 1. 克隆仓库
```bash
git clone https://github.com/Zhh9126/Oracle-Performance-Collection-Tools.git
cd Oracle-Performance-Collection-Tools
```

### 2. 安装依赖
```bash
pip install oracledb
```

### 3. 运行脚本
```bash
python Oracle-Performance-Collection-Tools.py
```

### 4. 根据菜单选择功能
按照终端提示输入数据库连接信息（用户名、密码、主机、端口、服务名），然后选择你需要的功能：

```
请选择功能：
1. 生成 AWR 报告（基于快照）
2. 生成 ASH 报告（基于时间范围）
3. SQL Trace 辅助工具
q. 退出
```

## 📝 使用示例

### 生成 AWR 报告
```
--- AWR 报告生成 ---
显示最近多少小时的快照？(直接回车显示全部): 24

DBID            INST  SNAP_ID    BEGIN_TIME             END_TIME
--------------------------------------------------------------------------------
1323206914      1     57         2026-04-17 16:46:52    2026-04-17 16:57:52
1323206914      1     58         2026-04-17 16:57:52    2026-04-17 18:00:03

请输入起始快照 ID: 57
请输入结束快照 ID: 58
报告格式 (html/text，默认 html): 

正在生成 AWR 报告...
✓ 报告已保存至: ./awr_57_58_20260421_142355.html
```

### 生成 ASH 报告（使用相对时间）
```
--- ASH 报告生成 ---
选择输入方式 (1=绝对时间, 2=相对时间，默认1): 2
相对开始时间 (例如 1h): 1h
相对结束时间 (例如 0m 表示当前，直接回车=当前): 
解析时间范围: 2026-04-21 13:30:00 至 2026-04-21 14:30:00
报告格式 (html/text，默认 html): 

正在生成 ASH 报告...
✓ 报告已保存至: ./ash_20260421_133000_20260421_143000_20260421_142530.html
```

### 使用 SQL Trace 跟踪指定会话
```
【SQL Trace 诊断工具】
请选择操作：
1. 跟踪当前会话
2. 跟踪指定会话（通过 SID 和 SERIAL#）
...
请输入选项: 2

请输入目标会话的 SID: 42
请输入目标会话的 SERIAL#: 9876
开启跟踪还是关闭跟踪？(start/stop): start
✓ 已开启会话 (SID=42, SERIAL#=9876) 的 SQL Trace。
预计 trace 文件: /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_12345.trc
```

## ⚙️ 环境变量配置（可选）

你可以通过设置环境变量来预设连接参数，避免每次输入：

| 变量名 | 描述 | 示例 |
| :--- | :--- | :--- |
| `ORACLE_USER` | 默认用户名 | `system` |
| `ORACLE_PASS` | 默认密码（不推荐明文存储） | `manager` |
| `ORACLE_HOST` | 默认主机地址 | `192.168.1.100` |
| `ORACLE_PORT` | 默认端口 | `1521` |
| `ORACLE_SERVICE` | 默认服务名 | `orcl` |
| `ORACLE_REPORT_DIR` | 报告输出目录 | `/home/oracle/reports` |

## 📂 项目结构
```
.
├── ora_perf_toolkit.py    # 主程序脚本
├── README.md              # 项目说明文档
└── LICENSE                # 开源许可证
```

## ❗ 注意事项

1. **权限要求**：生成 AWR/ASH 报告通常需要 `SELECT_CATALOG_ROLE` 角色或拥有 `DBA_HIST_*` 视图的访问权限。SQL Trace 的部分功能（如跟踪其他会话）可能需要 `SYSDBA` 权限。
2. **ASH 数据保留**：ASH 数据在内存中保留的时间有限，如果查询过老的时间范围可能返回空报告。
3. **SQL Trace 开销**：请仅在诊断时开启跟踪，完成后及时关闭，以避免产生大量 Trace 文件影响磁盘空间和系统性能。

## 🤝 贡献

欢迎提交 Issue 报告 Bug 或提出新功能建议。如果你想贡献代码，请遵循以下步骤：

1. Fork 本项目
2. 创建你的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交你的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

## 📄 许可证

本项目基于 MIT 许可证开源。详见 [LICENSE](LICENSE) 文件。

---

**如果觉得这个工具有用，请给个 Star ⭐ 支持一下！**
```

这份 `README.md` 结构清晰、信息完整，既适合首次接触的用户快速上手，也涵盖了高级用户关心的版本兼容性和权限细节。你可以根据实际的 GitHub 仓库链接替换 `your-username` 部分。
