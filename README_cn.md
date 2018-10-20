# Fortuna OJ 说明文档（非官方）

[English Version](README.md)

1. 页面中出现 "Failed to write session data" 的提示，如何解决？

   Fortuna OJ 使用 [redis](https://github.com/antirez/redis) 来存储用户交互凭据（以及代码提交的队列）。而这个问题的出现，通常是因为 **redis 的 AOF 文件在服务器意外断电重启时，遭受损坏**。

   如果要确认是否是这个原因，可以查看 redis 的日志（通常在 /var/log/redis/redis-server.log）。

   解决步骤：

   1. 在终端中打开 /var/lib/redis，找到 appendonly.aof

   2. 执行 sudo redis-check-aof --fix appendonly.aof，修复 AOF 文件

   3. 执行 sudo service redis-server restart，用以重启 redis 的后台服务进程

   4. 杀掉 php 并重新运行 yaujpushd 推送队列服务，以重启代码提交队列（当然重启一次服务器是更好的选择）

      - sudo killall php
      - sudo service yaujpushd start
      - 在管理页面中，对所有待评测任务提交重测请求

2. 某些提交一直处于 “运行中” 的状态，评测非常缓慢甚至卡住

   Fortuna OJ 使用 [vfk_uoj_sandbox](https://github.com/roastduck/vfk_uoj_sandbox) 作为沙盒来运行提交的代码。因为一个未知的问题（其实是因为我太菜），沙盒在运行一些**读入有缺陷的 Pascal 程序**时会变得非常缓慢（一个读入有缺陷的典型例子是，程序要求读入的数据量超出了题目规定）。

   然而，目前暂时没有办法能彻底解决这个问题。当然，最好别让用户重复提交有缺陷的 Pascal 程序。

