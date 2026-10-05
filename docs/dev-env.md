# 开发环境与版本约定

本文件让**一台陌生机器**能在 5 分钟内判断自己是否具备运行本仓库的条件（对应验收判据 AC-STU-08）。

## 1. 最低要求

| 组件 | 最低版本 | 本仓库验证过的版本 | 检查命令 |
|---|---|---|---|
| JDK | 17 | 21.0.10（Oracle） | `java -version` |
| Maven | 3.9 | 3.9.4 | `mvn -v` |
| Node.js | 20 | 24.15.0 | `node -v` |
| pnpm | 9 | 11.x | `pnpm -v` |
| MySQL | 8.0 | 8.0.34 | `mysql --version` |
| Git | 2.40 | 2.53 | `git --version` |

Docker **不是必需**：它只是可选的部署交付物（`deploy/docker-compose.yml`），
CI 与本地验收都不依赖它。

## 2. JDK 版本策略（容易踩的坑）

技术基线声明 **Java 17**，但本机可能装的是 JDK 21。做法是：

```xml
<properties>
  <java.version>17</java.version>
  <maven.compiler.release>17</maven.compiler.release>
</properties>
```

`release` 同时约束源码级别、字节码目标与 API 可见性，比 `source/target` 更严格。
**验证产物确实兼容 17**（class 文件 major version 必须是 61）：

```powershell
javap -v projects\P01-student-info\backend\target\classes\com\...\XxxApplication.class |
  Select-String "major version"
```

看到 `major version: 61` = Java 17；`65` = 你实际编译成了 21，需要检查 `release` 是否生效。

## 3. 数据库约定

- 一个库：`java_fullstack_lab`（`utf8mb4` / `utf8mb4_general_ci`）
- 每项目独立表前缀：P01 → `p01_*`，P02 → `p02_*`，以此类推（同一个库里物理隔离，可共存）
- 连接信息只写在**已 gitignore** 的 `backend/src/main/resources/application-local.yml`
- 仓库内只保留 `application-local.yml.example`，默认从环境变量读取：

```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/java_fullstack_lab?useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true}
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:}
```

> `allowPublicKeyRetrieval=true` 是 MySQL 8 默认 `caching_sha2_password` 认证在非 SSL 连接下所需，
> 缺失时会报 `Public Key Retrieval is not allowed`。仅用于本地开发，生产环境应走 TLS。

建库：

```sql
CREATE DATABASE IF NOT EXISTS java_fullstack_lab
  DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

> 注意：`root` 通常带密码。若 `mysql -uroot` 报 `Access denied`，请用有权限的本地账号，
> 并把凭据写入 `application-local.yml`；**不要**把真实密码写进任何被跟踪的文件。

## 4. 前端约定

- 多个前端工程放在 pnpm workspace（`frontend-workspace/`）里，`node_modules` 共享，避免十份重复下载
- 包管理器锁定为 pnpm；仓库提交 `pnpm-lock.yaml`，CI 使用 `pnpm install --frozen-lockfile`
- 前端只允许通过 `VITE_*` 暴露**非秘密**配置；任何密钥都不得进入前端构建产物

## 5. Windows / PowerShell 注意事项

本仓库的脚本在 Windows + PowerShell 下可直接运行，但有两条硬约束：

1. **`tools/` 下的 `.ps1` 脚本内部只用 ASCII 字符**（注释与输出都用英文）。
   原因是 Windows PowerShell 5.1 在没有 BOM 时按系统 ANSI 代码页（简体中文 Windows 为 GBK）
   解析脚本文件，UTF-8 中文会变成乱码并导致脚本报错。
   中文只出现在 `.md`、`.sql`、`.java` 等按 UTF-8 读取的文件里。
2. **`.editorconfig` 规定 `.ps1` 使用 CRLF**，其余文件使用 LF，避免混合换行导致的 diff 噪音。

## 6. 一次性的环境自检

```powershell
.\tools\lab.ps1 doctor
```

`doctor` 会依次检查 JDK / Maven / Node / pnpm / MySQL / Git 版本，以及数据库能否连通，
并把结论打印成一张表。它是每个项目 `deploy/runbook.md` 的第一步。
