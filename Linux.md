# Linux

## 你平时是怎么查看日志的？

一、实时日志监控（动态跟踪）

1、基础命令

```bash
# 实时追踪最新日志
tail -f app.log
# 实时过滤错误日志
tail -f app.log | grep "ERROR"
```

2、多文件追踪

```bash
# 同时监控多个日志文件
multitail app.log -I api.log  # -I 表示合并到一个窗口显示
# 通配符匹配多个服务日志
tail -f service_*.log
```

二、历史日志检索（（静态分析）

1、关键词搜索

```bash
# 基础搜索
grep "NullPointer" app.log
# 显示匹配行前后 10 行(Context)
grep -C 10 "OrderlD:123" app.log
#递归搜索目录
grep -r "fonnectionTimeout" /var/log//
```

## 软连接和硬链接

### 如何在Linux中查看进程号？

1. 使用 `ps` 命令

   `ps` 命令用于显示当前系统中运行的进程信息。通过不同的选项，可以查看进程号。

   ```bash
   # 列出所有进程，包括进程号（PID）
   ps -e
   # 显示更详细的信息，包括用户、进程号、父进程号、CPU 使用率等。
   ps -ef
   # 根据特定条件过滤
   ps -ef | grep [进程名]
   ```

2. 使用 `top` 命令

   `top` 命令提供了一个动态更新的进程列表，显示当前系统中占用资源最多的进程。

3. 使用 `pgrep` 命令

   `pgrep` 命令可以根据进程名或其他属性查找进程号。

   ```bash
   # 按进程名查找
   pgrep [进程名]
   # 按用户查找
   pgrep -u [用户名] [进程名]
   ```

4. 使用 `pidof` 命令

   `pidof` 命令可以根据进程名查找进程号。

   
