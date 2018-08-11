# 临时记录

没有想好分类到哪，临时记录

## 命令行创建数据库
```bash
CREATE DATABASE `表名` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

## 更新
```
UPDATE schedule_job
SET JOB_NAME = concat(JOB_NAME, '_', JOB_ID);
```
