### MySQL 建库建表

1. 创建数据库

    ```SQL
    CREATE DATABASE IF NOT EXISTS gmall_report DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
    ```

2. 建表

    ```SQL
    -- 1）各渠道流量统计

    DROP TABLE IF EXISTS `ads_traffic_stats_by_channel`;
    CREATE TABLE `ads_traffic_stats_by_channel`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `channel` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '渠道',
    `uv_count` bigint(20) NULL DEFAULT NULL COMMENT '访客人数',
    `avg_duration_sec` bigint(20) NULL DEFAULT NULL COMMENT '会话平均停留时长，单位为秒',
    `avg_page_count` bigint(20) NULL DEFAULT NULL COMMENT '会话平均浏览页面数',
    `sv_count` bigint(20) NULL DEFAULT NULL COMMENT '会话数',
    `bounce_rate` decimal(16, 2) NULL DEFAULT NULL COMMENT '跳出率',
    PRIMARY KEY (`dt`, `recent_days`, `channel`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各渠道流量统计' ROW_FORMAT = DYNAMIC;

    -- 2）路径分析

    DROP TABLE IF EXISTS `ads_page_path`;
    CREATE TABLE `ads_page_path`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `source` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '跳转起始页面ID',
    `target` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '跳转终到页面ID',
    `path_count` bigint(20) NULL DEFAULT NULL COMMENT '跳转次数',
    PRIMARY KEY (`dt`, `source`, `target`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '页面浏览路径分析' ROW_FORMAT = DYNAMIC;

    -- 3）用户变动统计

    DROP TABLE IF EXISTS `ads_user_change`;
    CREATE TABLE `ads_user_change`  (
    `dt` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '统计日期',
    `user_churn_count` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '流失用户数',
    `user_back_count` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '回流用户数',
    PRIMARY KEY (`dt`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户变动统计' ROW_FORMAT = DYNAMIC;

    -- 4）用户留存率

    DROP TABLE IF EXISTS `ads_user_retention`;
    CREATE TABLE `ads_user_retention`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `create_date` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户新增日期',
    `retention_day` int(20) NOT NULL COMMENT '截至当前日期留存天数',
    `retention_count` bigint(20) NULL DEFAULT NULL COMMENT '留存用户数量',
    `new_user_count` bigint(20) NULL DEFAULT NULL COMMENT '新增用户数量',
    `retention_rate` decimal(16, 2) NULL DEFAULT NULL COMMENT '留存率',
    PRIMARY KEY (`dt`, `create_date`, `retention_day`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '留存率' ROW_FORMAT = DYNAMIC;

    -- 5）用户新增活跃统计

    DROP TABLE IF EXISTS `ads_user_stats`;
    CREATE TABLE `ads_user_stats`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近n日,1:最近1日,7:最近7日,30:最近30日',
    `new_user_count` bigint(20) NULL DEFAULT NULL COMMENT '新增用户数',
    `active_user_count` bigint(20) NULL DEFAULT NULL COMMENT '活跃用户数',
    PRIMARY KEY (`dt`, `recent_days`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '用户新增活跃统计' ROW_FORMAT = DYNAMIC;

    -- 6）用户行为漏斗分析

    DROP TABLE IF EXISTS `ads_user_action`;
    CREATE TABLE `ads_user_action`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `home_count` bigint(20) NULL DEFAULT NULL COMMENT '浏览首页人数',
    `good_detail_count` bigint(20) NULL DEFAULT NULL COMMENT '浏览商品详情页人数',
    `cart_count` bigint(20) NULL DEFAULT NULL COMMENT '加入购物车人数',
    `order_count` bigint(20) NULL DEFAULT NULL COMMENT '下单人数',
    `payment_count` bigint(20) NULL DEFAULT NULL COMMENT '支付人数',
    PRIMARY KEY (`dt`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '漏斗分析' ROW_FORMAT = DYNAMIC;

    -- 7）新增下单用户统计

    DROP TABLE IF EXISTS `ads_new_order_user_stats`;
    CREATE TABLE `ads_new_order_user_stats`  (
    `dt` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `recent_days` bigint(20) NOT NULL,
    `new_order_user_count` bigint(20) NULL DEFAULT NULL,
    PRIMARY KEY (`recent_days`, `dt`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

    -- 8）最近7日内连续3日下单用户数

    DROP TABLE IF EXISTS `ads_order_continuously_user_count`;
    CREATE TABLE `ads_order_continuously_user_count`  (
    `dt` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `recent_days` bigint(20) NOT NULL,
    `order_continuously_user_count` bigint(20) NULL DEFAULT NULL,
    PRIMARY KEY (`dt`, `recent_days`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

    -- 9）最近30日各品牌复购率

    DROP TABLE IF EXISTS `ads_repeat_purchase_by_tm`;
    CREATE TABLE `ads_repeat_purchase_by_tm`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近天数,30:最近30天',
    `tm_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '品牌ID',
    `tm_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '品牌名称',
    `order_repeat_rate` decimal(16, 2) NULL DEFAULT NULL COMMENT '复购率',
    PRIMARY KEY (`dt`, `recent_days`, `tm_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各品牌复购率统计' ROW_FORMAT = DYNAMIC;

    -- 10）各品牌商品下单统计

    DROP TABLE IF EXISTS `ads_order_stats_by_tm`;
    CREATE TABLE `ads_order_stats_by_tm`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `tm_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '品牌ID',
    `tm_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '品牌名称',
    `order_count` bigint(20) NULL DEFAULT NULL COMMENT '订单数',
    `order_user_count` bigint(20) NULL DEFAULT NULL COMMENT '订单人数',
    PRIMARY KEY (`dt`, `recent_days`, `tm_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各品牌商品交易统计' ROW_FORMAT = DYNAMIC;

    -- 11）各分类商品下单统计

    DROP TABLE IF EXISTS `ads_order_stats_by_cate`;
    CREATE TABLE `ads_order_stats_by_cate`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `category1_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '一级分类id',
    `category1_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '一级分类名称',
    `category2_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '二级分类id',
    `category2_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '二级分类名称',
    `category3_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '三级分类id',
    `category3_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '三级分类名称',
    `order_count` bigint(20) NULL DEFAULT NULL COMMENT '订单数',
    `order_user_count` bigint(20) NULL DEFAULT NULL COMMENT '订单人数',
    PRIMARY KEY (`dt`, `recent_days`, `category1_id`, `category2_id`, `category3_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各分类商品交易统计' ROW_FORMAT = DYNAMIC;

    -- 12）各分类商品购物车存量Top3

    DROP TABLE IF EXISTS `ads_sku_cart_num_top3_by_cate`;
    CREATE TABLE `ads_sku_cart_num_top3_by_cate`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `category1_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '一级分类ID',
    `category1_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '一级分类名称',
    `category2_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '二级分类ID',
    `category2_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '二级分类名称',
    `category3_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '三级分类ID',
    `category3_name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '三级分类名称',
    `sku_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '商品id',
    `sku_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '商品名称',
    `cart_num` bigint(20) NULL DEFAULT NULL COMMENT '购物车中商品数量',
    `rk` bigint(20) NULL DEFAULT NULL COMMENT '排名',
    PRIMARY KEY (`dt`, `sku_id`, `category1_id`, `category2_id`, `category3_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各分类商品购物车存量Top10' ROW_FORMAT = DYNAMIC;

    -- 13）各品牌商品收藏次数Top3

    DROP TABLE IF EXISTS `ads_sku_favor_count_top3_by_tm`;
    CREATE TABLE `ads_sku_favor_count_top3_by_tm`  (
    `dt` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `tm_id` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `tm_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
    `sku_id` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `sku_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
    `favor_count` bigint(20) NULL DEFAULT NULL,
    `rk` bigint(20) NULL DEFAULT NULL,
    PRIMARY KEY (`dt`, `tm_id`, `sku_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

    -- 14）下单到支付时间间隔平均值

    DROP TABLE IF EXISTS `ads_order_to_pay_interval_avg`;
    CREATE TABLE `ads_order_to_pay_interval_avg`  (
    `dt` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `order_to_pay_interval_avg` bigint(20) NULL DEFAULT NULL,
    PRIMARY KEY (`dt`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

    -- 15）各省份交易统计

    DROP TABLE IF EXISTS `ads_order_by_province`;
    CREATE TABLE `ads_order_by_province`  (
    `dt` date NOT NULL COMMENT '统计日期',
    `recent_days` bigint(20) NOT NULL COMMENT '最近天数,1:最近1天,7:最近7天,30:最近30天',
    `province_id` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '省份ID',
    `province_name` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '省份名称',
    `area_code` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '地区编码',
    `iso_code` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '国际标准地区编码',
    `iso_code_3166_2` varchar(16) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '国际标准地区编码',
    `order_count` bigint(20) NULL DEFAULT NULL COMMENT '订单数',
    `order_total_amount` decimal(16, 2) NULL DEFAULT NULL COMMENT '订单金额',
    PRIMARY KEY (`dt`, `recent_days`, `province_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '各地区订单统计' ROW_FORMAT = DYNAMIC;

    -- 16）优惠券使用情况统计

    DROP TABLE IF EXISTS `ads_coupon_stats`;
    CREATE TABLE `ads_coupon_stats`  (
    `dt` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `coupon_id` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `coupon_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
    `used_count` bigint(20) NULL DEFAULT NULL,
    `userd_user_count` bigint(20) NULL DEFAULT NULL,
    PRIMARY KEY (`dt`, `coupon_id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
    ```


### DataX

1. DataX配置文件生成脚本`gen_export_config.py`

    ```python
    # coding=utf-8
    import json
    import getopt
    import os
    import sys
    import MySQLdb

    #MySQL相关配置，需根据实际情况作出修改
    mysql_host = "hadoop102"
    mysql_port = "3306"
    mysql_user = "root"
    mysql_passwd = "passw0rd"

    #HDFS NameNode相关配置，需根据实际情况作出修改
    hdfs_nn_host = "hadoop102"
    hdfs_nn_port = "8020"

    #生成配置文件的目标路径，可根据实际情况作出修改
    output_path = "/opt/module/datax/job/export"


    def get_connection():
        return MySQLdb.connect(host=mysql_host, port=int(mysql_port), user=mysql_user, passwd=mysql_passwd)


    def get_mysql_meta(database, table):
        connection = get_connection()
        cursor = connection.cursor()
        sql = "SELECT COLUMN_NAME,DATA_TYPE from information_schema.COLUMNS WHERE TABLE_SCHEMA=%s AND TABLE_NAME=%s ORDER BY ORDINAL_POSITION"
        cursor.execute(sql, [database, table])
        fetchall = cursor.fetchall()
        cursor.close()
        connection.close()
        return fetchall


    def get_mysql_columns(database, table):
        return map(lambda x: x[0], get_mysql_meta(database, table))


    def generate_json(target_database, target_table):
        job = {
            "job": {
                "setting": {
                    "speed": {
                        "channel": 3
                    },
                    "errorLimit": {
                        "record": 0,
                        "percentage": 0.02
                    }
                },
                "content": [{
                    "reader": {
                        "name": "hdfsreader",
                        "parameter": {
                            "path": "${exportdir}",
                            "defaultFS": "hdfs://" + hdfs_nn_host + ":" + hdfs_nn_port,
                            "column": ["*"],
                            "fileType": "text",
                            "encoding": "UTF-8",
                            "fieldDelimiter": "\t",
                            "nullFormat": "\\N"
                        }
                    },
                    "writer": {
                        "name": "mysqlwriter",
                        "parameter": {
                            "writeMode": "replace",
                            "username": mysql_user,
                            "password": mysql_passwd,
                            "column": get_mysql_columns(target_database, target_table),
                            "connection": [
                                {
                                    "jdbcUrl":
                                        "jdbc:mysql://" + mysql_host + ":" + mysql_port + "/" + target_database + "?useUnicode=true&characterEncoding=utf-8",
                                    "table": [target_table]
                                }
                            ]
                        }
                    }
                }]
            }
        }
        if not os.path.exists(output_path):
            os.makedirs(output_path)
        with open(os.path.join(output_path, ".".join([target_database, target_table, "json"])), "w") as f:
            json.dump(job, f)


    def main(args):
        target_database = ""
        target_table = ""

        options, arguments = getopt.getopt(args, '-d:-t:', ['targetdb=', 'targettbl='])
        for opt_name, opt_value in options:
            if opt_name in ('-d', '--targetdb'):
                target_database = opt_value
            if opt_name in ('-t', '--targettbl'):
                target_table = opt_value

        generate_json(target_database, target_table)

    if __name__ == '__main__':
        main(sys.argv[1:])
    ```

    注：
    1. 安装Python Mysql驱动
    由于需要使用Python访问Mysql数据库，故需安装驱动，命令如下：
    ```shell
    sudo yum install -y MySQL-python
    ```
    1. 脚本使用说明
    ```shell
    python gen_export_config.py -d database -t table
    ```
    通过-d传入MySQL数据库名，-t传入MySQL表名，执行上述命令即可生成该表的DataX同步配置文件。

2. `gen_export_config.sh`脚本

    ```shell
    #!/bin/bash

    python ~/bin/gen_export_config.py -d gmall_report -t ads_coupon_stats
    python ~/bin/gen_export_config.py -d gmall_report -t ads_new_order_user_stats
    python ~/bin/gen_export_config.py -d gmall_report -t ads_order_by_province
    python ~/bin/gen_export_config.py -d gmall_report -t ads_order_continuously_user_count
    python ~/bin/gen_export_config.py -d gmall_report -t ads_order_stats_by_cate
    python ~/bin/gen_export_config.py -d gmall_report -t ads_order_stats_by_tm
    python ~/bin/gen_export_config.py -d gmall_report -t ads_order_to_pay_interval_avg
    python ~/bin/gen_export_config.py -d gmall_report -t ads_page_path
    python ~/bin/gen_export_config.py -d gmall_report -t ads_repeat_purchase_by_tm
    python ~/bin/gen_export_config.py -d gmall_report -t ads_sku_cart_num_top3_by_cate
    python ~/bin/gen_export_config.py -d gmall_report -t ads_sku_favor_count_top3_by_tm
    python ~/bin/gen_export_config.py -d gmall_report -t ads_traffic_stats_by_channel
    python ~/bin/gen_export_config.py -d gmall_report -t ads_user_action
    python ~/bin/gen_export_config.py -d gmall_report -t ads_user_change
    python ~/bin/gen_export_config.py -d gmall_report -t ads_user_retention
    python ~/bin/gen_export_config.py -d gmall_report -t ads_user_stats
    ```

    执行：

    ```shell
    gen_export_config.sh 
    ```

3. 观察生成的配置文件
   
    [samantha@hadoop102 bin]$ ls /opt/module/datax/job/export/
    总用量 64
    gmall_report.ads_activity_stats.json                 gmall_report.ads_trade_stats_by_cate.json
    gmall_report.ads_coupon_stats.json                   gmall_report.ads_trade_stats_by_tm.json
    gmall_report.ads_new_buyer_stats.json                gmall_report.ads_trade_stats.json
    gmall_report.ads_order_by_province.json              gmall_report.ads_traffic_stats_by_channel.json
    gmall_report.ads_user_action.json
    gmall_report.ads_page_path.json                      gmall_report.ads_user_change.json
    gmall_report.ads_repeat_purchase_by_tm.json          gmall_report.ads_user_retention.json
    gmall_report.ads_sku_cart_num_top3_by_cate.json      gmall_report.ads_user_stats.json
    12.2.3 测试生成的DataX配置文件
    以ads_traffic_stats_by_channel为例，测试用脚本生成的配置文件是否可用。
    1）执行DataX同步命令
    [samantha@hadoop102 bin]$ python /opt/module/datax/bin/datax.py -p"-Dexportdir=/warehouse/gmall/ads/ads_traffic_stats_by_channel" /opt/module/datax/job/export/gmall_report.ads_traffic_stats_by_channel.json
    2）观察同步结果
    观察MySQL目标表是否出现数据。


### 编写每日导出脚本 

1. `hdfs_to_mysql.sh`

    ```shell
    #! /bin/bash

    DATAX_HOME=/opt/module/datax

    #DataX导出路径不允许存在空文件，该函数作用为清理空文件
    handle_export_path(){
    for i in `hadoop fs -ls -R $1 | awk '{print $8}'`; do
        hadoop fs -test -z $i
        if [[ $? -eq 0 ]]; then
        echo "$i文件大小为0，正在删除"
        hadoop fs -rm -r -f $i
        fi
    done
    }

    #数据导出
    export_data() {
    datax_config=$1
    export_dir=$2
    handle_export_path $export_dir
    $DATAX_HOME/bin/datax.py -p"-Dexportdir=$export_dir" $datax_config
    }

    case $1 in
    "ads_coupon_stats")
        export_data /opt/module/datax/job/export/gmall_report.ads_coupon_stats.json /warehouse/gmall/ads/ads_coupon_stats
    ;;
    "ads_new_order_user_stats")
        export_data /opt/module/datax/job/export/gmall_report.ads_new_order_user_stats.json /warehouse/gmall/ads/ads_new_order_user_stats
    ;;  
    "ads_order_by_province")
        export_data /opt/module/datax/job/export/gmall_report.ads_order_by_province.json /warehouse/gmall/ads/ads_order_by_province
    ;;
    "ads_order_continuously_user_count")
        export_data /opt/module/datax/job/export/gmall_report.ads_order_continuously_user_count.json /warehouse/gmall/ads/ads_order_continuously_user_count
    ;;
    "ads_order_stats_by_cate")
        export_data /opt/module/datax/job/export/gmall_report.ads_order_stats_by_cate.json /warehouse/gmall/ads/ads_order_stats_by_cate
    ;;
    "ads_order_stats_by_tm")
        export_data /opt/module/datax/job/export/gmall_report.ads_order_stats_by_tm.json /warehouse/gmall/ads/ads_order_stats_by_tm
    ;;  
    "ads_order_to_pay_interval_avg")
        export_data /opt/module/datax/job/export/gmall_report.ads_order_to_pay_interval_avg.json /warehouse/gmall/ads/ads_order_to_pay_interval_avg
    ;;
    "ads_page_path")
        export_data /opt/module/datax/job/export/gmall_report.ads_page_path.json /warehouse/gmall/ads/ads_page_path
    ;;
    "ads_repeat_purchase_by_tm")
        export_data /opt/module/datax/job/export/gmall_report.ads_repeat_purchase_by_tm.json /warehouse/gmall/ads/ads_repeat_purchase_by_tm
    ;;
    "ads_sku_cart_num_top3_by_cate")
        export_data /opt/module/datax/job/export/gmall_report.ads_sku_cart_num_top3_by_cate.json /warehouse/gmall/ads/ads_sku_cart_num_top3_by_cate
    ;;  
    "ads_sku_favor_count_top3_by_tm")
        export_data /opt/module/datax/job/export/gmall_report.ads_sku_favor_count_top3_by_tm.json /warehouse/gmall/ads/ads_sku_favor_count_top3_by_tm
    ;;
    "ads_traffic_stats_by_channel")
        export_data /opt/module/datax/job/export/gmall_report.ads_traffic_stats_by_channel.json /warehouse/gmall/ads/ads_traffic_stats_by_channel
    ;;
    "ads_user_action")
        export_data /opt/module/datax/job/export/gmall_report.ads_user_action.json /warehouse/gmall/ads/ads_user_action
    ;;
    "ads_user_change")
        export_data /opt/module/datax/job/export/gmall_report.ads_user_change.json /warehouse/gmall/ads/ads_user_change
    ;;  
    "ads_user_retention")
        export_data /opt/module/datax/job/export/gmall_report.ads_user_retention.json /warehouse/gmall/ads/ads_user_retention
    ;;
    "ads_user_stats")
        export_data /opt/module/datax/job/export/gmall_report.ads_user_stats.json /warehouse/gmall/ads/ads_user_stats
    ;;
    "all")
        export_data /opt/module/datax/job/export/gmall_report.ads_coupon_stats.json /warehouse/gmall/ads/ads_coupon_stats
        export_data /opt/module/datax/job/export/gmall_report.ads_new_order_user_stats.json /warehouse/gmall/ads/ads_new_order_user_stats
        export_data /opt/module/datax/job/export/gmall_report.ads_order_by_province.json /warehouse/gmall/ads/ads_order_by_province
        export_data /opt/module/datax/job/export/gmall_report.ads_order_continuously_user_count.json /warehouse/gmall/ads/ads_order_continuously_user_count
        export_data /opt/module/datax/job/export/gmall_report.ads_order_stats_by_cate.json /warehouse/gmall/ads/ads_order_stats_by_cate
        export_data /opt/module/datax/job/export/gmall_report.ads_order_stats_by_tm.json /warehouse/gmall/ads/ads_order_stats_by_tm
        export_data /opt/module/datax/job/export/gmall_report.ads_order_to_pay_interval_avg.json /warehouse/gmall/ads/ads_order_to_pay_interval_avg
        export_data /opt/module/datax/job/export/gmall_report.ads_page_path.json /warehouse/gmall/ads/ads_page_path
        export_data /opt/module/datax/job/export/gmall_report.ads_repeat_purchase_by_tm.json /warehouse/gmall/ads/ads_repeat_purchase_by_tm
        export_data /opt/module/datax/job/export/gmall_report.ads_sku_cart_num_top3_by_cate.json /warehouse/gmall/ads/ads_sku_cart_num_top3_by_cate
        export_data /opt/module/datax/job/export/gmall_report.ads_sku_favor_count_top3_by_tm.json /warehouse/gmall/ads/ads_sku_favor_count_top3_by_tm
        export_data /opt/module/datax/job/export/gmall_report.ads_traffic_stats_by_channel.json /warehouse/gmall/ads/ads_traffic_stats_by_channel
        export_data /opt/module/datax/job/export/gmall_report.ads_user_action.json /warehouse/gmall/ads/ads_user_action
        export_data /opt/module/datax/job/export/gmall_report.ads_user_change.json /warehouse/gmall/ads/ads_user_change
        export_data /opt/module/datax/job/export/gmall_report.ads_user_retention.json /warehouse/gmall/ads/ads_user_retention
        export_data /opt/module/datax/job/export/gmall_report.ads_user_stats.json /warehouse/gmall/ads/ads_user_stats
    ;;
    esac
    ```

    执行：

    ```shell
    hdfs_to_mysql.sh all
    ```
