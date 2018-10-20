# Documents for Fortuna Online Judge (Unofficial)

[中文文档](README_cn.md)

1. How to solve "Failed to write session data" problem?

   fortuna-oj uses [redis](https://github.com/antirez/redis) to handle with session data (and submissions). Usually problem occurs because the **AOF file of redis is corrupted**, due to accidental reboots of the web server.

   To identify this problem, please check redis log (usually in /var/log/redis/redis-server.log).

   To solve this:

   1. Go to /var/lib/redis in terminal, find "appendonly.aof"

   2. Execute "sudo redis-check-aof --fix appendonly.aof" to fix the AOF file

   3. Execute "sudo service redis-server restart" to restart redis background service

   4. Kill php and restart yaujpushd service to reset the submission queue if necessary

      - sudo killall php
      - sudo service yaujpushd start
      - Rejudge all submissions that are pending to be judged in the webpage

2. Some of the submissions are being judged, but way slower than usual.

   fortuna-oj uses [vfk_uoj_sandbox](https://github.com/roastduck/vfk_uoj_sandbox) for running submissions' programs. Due to an unknown issue (most probably because I'm too garbage :( ), the sandbox will become **much slower** when **judging a specific kind of Pascal programs which have I/O problems (e.g. reading too much data from input)**. 

   Unfortunately there is no way to completely solve this problem. But still, stopping users submitting wrong Pascal programs (especially submitting them repeatedly) might be the best solution.