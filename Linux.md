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

