# 0基础下载并安装SQLite并新建数据库

//本文章由chatgpt-3.5协助生成。//

## 步骤 1: 下载并安装 SQLite

（此部分有参考）

1. 访问 SQLite 官方Download Page: [https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)

2. 选择适用的文件：**Precompiled Binaries for <你的操作系统>**

   例如Windows就在**Precompiled Binaries for Windows**中下载

   | [sqlite-dll-win-x64-3440200.zip](https://www.sqlite.org/2023/sqlite-dll-win-x64-3440200.zip) (1.24 MiB) | 64-bit DLL (x64) for SQLite version 3.44.2.<br/>(SHA3-256: bf2b78a7f610cabd1046ee2587640b0ecc01bf8916381e7e6cdaa0be70eeee70) |
   | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | [sqlite-tools-win-x64-3440200.zip](https://www.sqlite.org/2023/sqlite-tools-win-x64-3440200.zip) (4.71 MiB) | A bundle of command-line tools for managing SQLite database files, including the [command-line shell](https://www.sqlite.org/cli.html) program, the [sqldiff.exe](https://www.sqlite.org/sqldiff.html) program, and the [sqlite3_analyzer.exe](https://www.sqlite.org/sqlanalyze.html) program. 64-bit.<br/>(SHA3-256: debfc40f706b389efbfe90ccdcb9f209ab0bb7306115a868c192ce40fd03bb71) |

   这两个文件（版本可能会有变化）。

3. 新建文件夹 `C:\sqlite3` 并把这两个文件解压进此位置。

## 步骤 2: 在终端或命令提示符中打开 SQLite

（此部分有参考）

### Windows

在执行sql之前需要给系统环境变量添加 `C:\sqlite3` 。

- 点击此电脑”右键-属性-高级系统设置-环境变量。或者点击右下角Windows图标并搜索“编辑系统环境变量”，进入“系统属性页面”，然后点击 `N-键` （或者右下角“环境变量”）

- 在跳出“环境变量”界面的“用户环境变量”中选择“Path”, 然后点击“编辑”
- 在“编辑环境变量”界面右侧点击“新建”，输入`C:\sqlite3`，然后上移到最上方。

打开命令提示符 (Command Prompt)，可以右下角搜索。然后输入

```shell
cd C:\sqlite3
```

导航到 SQLite 安装的文件夹。可以通过输入 `sqlite3` 指令运行刚才解压好的 `sqlite3.exe`，然后出现以下界面：

```shell
C:\sqlite3>sqlite3
SQLite version 3.44.2 2023-11-24 11:41:44 (UTF-16 console I/O)
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite>
```

则成功运行。

### macOS 或 Linux

打开终端，然后直接运行 `sqlite3`。

## 步骤 3: 创建一个新数据库

在 SQLite 命令行中，可以使用以下命令创建一个新的数据库文件：

```shell
sqlCopy code
sqlite> .open mydatabase.db
```

这会在当前目录下创建一个名为 `mydatabase.db` 的数据库文件。你可以将文件名替换为你想要的名字。

## 步骤 4: 创建表和插入数据

创建表格的 SQL 语句类似于以下示例：

```shell
sqlCopy codesqlite> CREATE TABLE users (
   ...>    id INTEGER PRIMARY KEY,
   ...>    name TEXT,
   ...>    age INTEGER
   ...> );
```

这个示例创建了一个名为 `users` 的表，包含 `id`、`name` 和 `age` 列。

然后，你可以插入数据：

```shell
sqlCopy codesqlite> INSERT INTO users (name, age) VALUES ('Alice', 25);
sqlite> INSERT INTO users (name, age) VALUES ('Bob', 30);
```

## 步骤 5: 查询数据

使用 `SELECT` 语句来查询数据：

```shell
sqlCopy code
sqlite> SELECT * FROM users;
```

这将返回 `users` 表中的所有数据。

## 步骤 6: 退出 SQLite

在 SQLite 命令行中，你可以使用 `.exit` 命令或者按 `Ctrl + D`（在 Windows 中是 `Ctrl + Z`）退出 SQLite。

这是一个简单的起点，你可以从这里开始学习更多关于 SQLite 的内容。随着你的学习深入，你可以探索更多高级的 SQL 概念和 SQLite 特有的功能。



## Reference

- [Windows上安装sqlite3（超详细）](https://blog.csdn.net/lj19990824/article/details/120966250)