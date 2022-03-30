---
title: mysql触发器
date: 2022-03-24 16:43:25
tags: mysql
---
## 1.问题
今天在进行业务操作的时候发现一个问题，sql语句是没有问题的，但是执行的时候发现会报错，无法保存进去
![sql错误](/../../static/mysql/mysql-error.png)

在数据库里面执行的时候也是出现了这个问题，当时一脸懵逼，感觉也没啥问题,确实没啥问题。

## 2 如何解决
将数据库的相关表结构导出来发现sql语句存在 触发器的语句
```xml
  -- Table structure for pv_visit_record_statistics_daily
-- ----------------------------
DROP TABLE IF EXISTS `pv_visit_record_statistics_daily`;
CREATE TABLE `pv_visit_record_statistics_daily`  (
  `id` char(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '' COMMENT '走访记录每天的统计表id',
  `total_amount` bigint(10) NULL DEFAULT NULL COMMENT '巡查/走访总量',
  `transfer_event_amount` bigint(10) NULL DEFAULT NULL COMMENT '巡查/走访总量异常转事件量',
  `abnormal_amount` bigint(10) NULL DEFAULT NULL COMMENT '异常总量(异常可以不转事件)',
  `issue_amount` bigint(10) NULL DEFAULT NULL COMMENT '转事件总量',
  `org_id` bigint(19) NULL DEFAULT NULL COMMENT '组织机构id',
  `org_code` varchar(45) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '组织机构编码',
  `type` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据类型(三级类型)',
  `statistics_date` date NULL DEFAULT NULL COMMENT '统计日期',
  `create_date` datetime(0) NOT NULL COMMENT '创建时间',
  `create_user` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '新建者',
  `update_date` datetime(0) NULL DEFAULT NULL COMMENT '更新时间',
  `update_user` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '更新者',
  `visit_mode` tinyint(1) NULL DEFAULT NULL COMMENT '走访方式1 任务，2自主',
  `domain_type` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '一级类型',
  `parent_type` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '二级类型',
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `idx_org_id`(`org_id`) USING BTREE,
  INDEX `idx_type_visit_mode`(`type`, `visit_mode`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '走访记录每天的统计' ROW_FORMAT = Dynamic;

-- ----------------------------
-- Triggers structure for table pv_visit_record_statistics_daily
-- ----------------------------
DROP TRIGGER IF EXISTS `SIS_I68011E5815AC7B52`;
delimiter ;;
CREATE TRIGGER `SIS_I68011E5815AC7B52` AFTER INSERT ON `pv_visit_record_statistics_daily` FOR EACH ROW begin    declare flag int;   select count(1) into flag from CBD_GAP_EVENT as G where G.CBD_DB_HOST=@legendsec_con_str;   if flag = 0 then      insert into CBD_DB_EVENT(CBD_DB_TIME,CBD_DB_NAME,CBD_DB_OPERATE,CBD_DB_STATUS,CBD_DB_KEY) values (now(),'pv_visit_record_statistics_daily','INSERT',1,CONCAT('','^',NEW.id));   end if; end
;;
delimiter ;

-- ----------------------------
-- Triggers structure for table pv_visit_record_statistics_daily
-- ----------------------------
DROP TRIGGER IF EXISTS `SIS_U68011E5815AC7B52`;
delimiter ;;
CREATE TRIGGER `SIS_U68011E5815AC7B52` AFTER UPDATE ON `pv_visit_record_statistics_daily` FOR EACH ROW begin   declare flag int;  declare myfields varchar(2000);  select count(1) into flag from CBD_GAP_EVENT as G where G.CBD_DB_HOST=@legendsec_con_str;  if flag = 0 then        if !(NEW.id<=>OLD.id) then     select 1 into flag from dual;   end if;      if flag = 1 then    insert into CBD_DB_EVENT(CBD_DB_TIME,CBD_DB_NAME,CBD_DB_OPERATE,CBD_DB_STATUS,CBD_DB_KEY) values (now(),'pv_visit_record_statistics_daily','DELETE',1,CONCAT('','^',OLD.id));    insert into CBD_DB_EVENT(CBD_DB_TIME,CBD_DB_NAME,CBD_DB_OPERATE,CBD_DB_STATUS,CBD_DB_KEY) values (now(),'pv_visit_record_statistics_daily','INSERT',1,CONCAT('','^',NEW.id));   else    set @myfields='';        if !(NEW.org_id<=>OLD.org_id) then     set @myfields=CONCAT(@myfields,'^','org_id');    end if;        if !(NEW.total_amount<=>OLD.total_amount) then     set @myfields=CONCAT(@myfields,'^','total_amount');    end if;        if !(NEW.update_user<=>OLD.update_user) then     set @myfields=CONCAT(@myfields,'^','update_user');    end if;        if !(NEW.create_date<=>OLD.create_date) then     set @myfields=CONCAT(@myfields,'^','create_date');    end if;        if !(NEW.parent_type<=>OLD.parent_type) then     set @myfields=CONCAT(@myfields,'^','parent_type');    end if;        if !(NEW.transfer_event_amount<=>OLD.transfer_event_amount) then     set @myfields=CONCAT(@myfields,'^','transfer_event_amount');    end if;        if !(NEW.update_date<=>OLD.update_date) then     set @myfields=CONCAT(@myfields,'^','update_date');    end if;        if !(NEW.org_code<=>OLD.org_code) then     set @myfields=CONCAT(@myfields,'^','org_code');    end if;        if !(NEW.create_user<=>OLD.create_user) then     set @myfields=CONCAT(@myfields,'^','create_user');    end if;        if !(NEW.domain_type<=>OLD.domain_type) then     set @myfields=CONCAT(@myfields,'^','domain_type');    end if;        if !(NEW.statistics_date<=>OLD.statistics_date) then     set @myfields=CONCAT(@myfields,'^','statistics_date');    end if;        if !(NEW.type<=>OLD.type) then     set @myfields=CONCAT(@myfields,'^','type');    end if;        if !(NEW.visit_mode<=>OLD.visit_mode) then     set @myfields=CONCAT(@myfields,'^','visit_mode');    end if;        if !(NEW.issue_amount<=>OLD.issue_amount) then     set @myfields=CONCAT(@myfields,'^','issue_amount');    end if;        if !(NEW.abnormal_amount<=>OLD.abnormal_amount) then     set @myfields=CONCAT(@myfields,'^','abnormal_amount');    end if;            insert into CBD_DB_EVENT(CBD_DB_TIME,CBD_DB_NAME,CBD_DB_OPERATE,CBD_DB_STATUS,CBD_DB_KEY,CBD_DB_FIELDS) values (now(),'pv_visit_record_statistics_daily','UPDATE',1,CONCAT('','^',OLD.id),@myfields);   end if;  end if; end
;;
delimiter ;

-- ----------------------------
-- Triggers structure for table pv_visit_record_statistics_daily
-- ----------------------------
DROP TRIGGER IF EXISTS `SIS_D68011E5815AC7B52`;
delimiter ;;
CREATE TRIGGER `SIS_D68011E5815AC7B52` AFTER DELETE ON `pv_visit_record_statistics_daily` FOR EACH ROW begin  declare flag int;  select count(1) into flag from CBD_GAP_EVENT as G where G.CBD_DB_HOST=@legendsec_con_str;  if flag = 0 then   insert into CBD_DB_EVENT(CBD_DB_TIME,CBD_DB_NAME,CBD_DB_OPERATE,CBD_DB_STATUS,CBD_DB_KEY) values (now(),'pv_visit_record_statistics_daily','DELETE',1,CONCAT('','^',OLD.id));  end if; end
;;
delimiter ;

SET FOREIGN_KEY_CHECKS = 1;
```
sql语句多出了三句执行器相关的语句.
>当执行新增，删除，修改的时候，会执行相关的触发器的执行语句
