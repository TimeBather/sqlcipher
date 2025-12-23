# SQLCipher Windows 构建工作流

## 概述

此 GitHub Action 工作流会自动在 Windows 平台下编译 SQLCipher 命令行工具。

## 工作流说明

### 触发条件

- 推送到 `main`, `master`, 或 `prerelease` 分支
- 针对上述分支的 Pull Request
- 手动触发 (workflow_dispatch)

### 构建矩阵

工作流会为以下平台构建 SQLCipher:
- x64 (64位)
- x86 (32位)

### 主要步骤

1. **检出代码**: 使用 `actions/checkout@v4` 检出仓库代码

2. **设置 MSBuild**: 配置 Microsoft Build Tools

3. **安装 OpenSSL**: 通过 vcpkg 安装静态链接的 OpenSSL 库
   - 使用 GitHub Actions runner 内置的 vcpkg (最新版本)
   - x64 平台: `openssl:x64-windows-static`
   - x86 平台: `openssl:x86-windows-static`

4. **设置 Visual Studio 开发者命令提示符**: 配置正确的编译环境

5. **编译 SQLCipher**: 使用 `nmake` 和 `Makefile.msc` 编译
   - 必需的编译选项:
     - `SQLITE_HAS_CODEC`: 启用加密支持
     - `SQLITE_TEMP_STORE=2`: 设置临时存储模式
     - `SQLITE_EXTRA_INIT=sqlcipher_extra_init`: 额外初始化函数
     - `SQLITE_EXTRA_SHUTDOWN=sqlcipher_extra_shutdown`: 额外关闭函数
     - `SQLCIPHER_CRYPTO_OPENSSL`: 使用 OpenSSL 作为加密提供者
     - `HAVE_STDINT_H=1`: 启用 stdint.h 支持（MSVC 需要）
   - 链接 OpenSSL 的 `libcrypto.lib` 以及必要的 Windows 系统库:
     - `ws2_32.lib`: Windows Sockets API
     - `advapi32.lib`: 高级 Windows API（加密服务）
     - `user32.lib`: Windows 用户界面 API
     - `crypt32.lib`: Windows 加密 API

6. **验证构建**: 检查 `sqlite3.exe` 是否成功生成

7. **上传构建产物**: 将编译好的 `sqlite3.exe` 作为构建产物上传

## 编译原理

SQLCipher 是 SQLite 的加密版本，需要以下组件:

1. **OpenSSL**: 提供加密功能 (使用 libcrypto)
2. **特殊编译标志**: 
   - `SQLITE_HAS_CODEC`: 启用编解码器
   - `SQLITE_TEMP_STORE=2`: 使用内存存储临时数据
   - `SQLITE_EXTRA_INIT` 和 `SQLITE_EXTRA_SHUTDOWN`: SQLCipher 初始化/关闭钩子
   - `SQLCIPHER_CRYPTO_OPENSSL`: 指定使用 OpenSSL 作为加密提供者
   - `HAVE_STDINT_H=1`: 启用标准整数类型头文件支持（MSVC 编译器需要）

## 使用构建产物

构建完成后，可以从 GitHub Actions 的 Artifacts 部分下载编译好的可执行文件:
- `sqlcipher-windows-x64`: 64位版本
- `sqlcipher-windows-x86`: 32位版本

## 参考文档

- [SQLCipher 官方文档](https://www.zetetic.net/sqlcipher/documentation/)
- [从源码编译 SQLCipher](https://github.com/sqlcipher/sqlcipher#compiling)
- [Windows 编译说明](../../doc/compile-for-windows.md)

## 注意事项

- 此工作流使用静态链接的 OpenSSL，生成的可执行文件不需要额外的 DLL 依赖
- 编译过程不需要 TCL (通过 `NO_TCL=1` 禁用)
- 使用合并文件模式 (`USE_AMALGAMATION=1`) 以提高编译速度和性能
