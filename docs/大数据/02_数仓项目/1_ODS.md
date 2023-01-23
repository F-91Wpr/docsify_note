- HDFS File (采集) -> SQL (hive) -> Table Data (ODS)

### ODS 层设计：

1. 表结构设计
    - 日志表 (Flume)：`JSON`格式
    - 全量表 (DataX)：`TSV`格式
    - 增量表 (Maxwell)：`JSON`格式

2. 压缩格式：gzip - 压缩比高，Hadoop 可以直接使用。

3. 命名规范为：ods_表名_增量全量标识（inc/full） 

### SQL TIP

1. 建表：

    - 内部表：默认

    - 外部表：`external`

    - 建表路径：`location ''`

    - 分区：

        - 定义分区：`partitioned by (key string)` - 在路径中追加分区目录，在表中添加分区属性

        - 静态分区操作：`partition ( key=value )`

        - 动态分区操作：`set hive.exec.dynamic.partition.mode=nonstrict;`

    - 特殊表：JSON 表

        ```SQL
        CREATE TABLE my_table(a string, b bigint, ...)
        ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
        STORED AS TEXTFILE;
        ```

        - 定义特殊字段

          - array_type
            : ARRAY < data_type >
            
          - map_type
            : MAP < primitive_type, data_type >
            
          - struct_type
            : STRUCT < col_name : data_type [COMMENT col_comment], ...>
            
          - union_type
            : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
            
          - row_format
            : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
                    [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
                    [NULL DEFINED AS char]   

2. file --> table
    Hive does not do any transformation while loading data into tables. Load operations are currently pure copy/move operations that move datafiles into locations corresponding to Hive tables.

    ```SQL
    LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)];
    ```

### 脚本

1. 日志数据装载脚本`hdfs_to_ods_log.sh`

    ```shell
    #!/bin/bash

    # 定义变量方便修改
    APP=gmall
    APP_DB=gmall

    # 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
    if [ -n "$1" ] ;then
    DO_DATE=$1
    else
    DO_DATE=`date -d "-1 day" +%F`
    fi

    echo ================== 日志日期为 $DO_DATE ==================
    SQL="
    load data inpath '/origin_data/$APP/log/topic_log/$DO_DATE' into table ${APP_DB}.ods_log_inc partition(dt='$DO_DATE');
    "
    hive -e "$SQL"
    ```

    执行：
    ```
    hdfs_to_ods_log.sh 2020-06-14
    ```

2. 业务数据装载脚本`hdfs_to_ods_db.sh`

    ```shell
    #!/bin/bash

    APP=gmall
    APP_DB=gmall

    if [ -n "$2" ] ;then
    DO_DATE=$2
    else 
    DO_DATE=`date -d '-1 day' +%F`
    fi

    load_data(){
        SQL=""
        for i in $*; do
            #判断路径是否存在
            hadoop fs -test -e /origin_data/$APP/db/${i:4}/$DO_DATE
            #路径存在方可装载数据
            if [[ $? = 0 ]]; then
                SQL=$SQL"load data inpath '/origin_data/$APP/db/${i:4}/$DO_DATE' OVERWRITE into table ${APP_DB}.$i partition(dt='$DO_DATE');"
            fi
        done
        hive -e "$SQL"
    }

    case $1 in
        "ods_activity_info_full")
            load_data "ods_activity_info_full"
        ;;
        "ods_activity_rule_full")
            load_data "ods_activity_rule_full"
        ;;
        "ods_base_category1_full")
            load_data "ods_base_category1_full"
        ;;
        "ods_base_category2_full")
            load_data "ods_base_category2_full"
        ;;
        "ods_base_category3_full")
            load_data "ods_base_category3_full"
        ;;
        "ods_base_dic_full")
            load_data "ods_base_dic_full"
        ;;
        "ods_base_province_full")
            load_data "ods_base_province_full"
        ;;
        "ods_base_region_full")
            load_data "ods_base_region_full"
        ;;
        "ods_base_trademark_full")
            load_data "ods_base_trademark_full"
        ;;
        "ods_cart_info_full")
            load_data "ods_cart_info_full"
        ;;
        "ods_coupon_info_full")
            load_data "ods_coupon_info_full"
        ;;
        "ods_sku_attr_value_full")
            load_data "ods_sku_attr_value_full"
        ;;
        "ods_sku_info_full")
            load_data "ods_sku_info_full"
        ;;
        "ods_sku_sale_attr_value_full")
            load_data "ods_sku_sale_attr_value_full"
        ;;
        "ods_spu_info_full")
            load_data "ods_spu_info_full"
        ;;

        "ods_cart_info_inc")
            load_data "ods_cart_info_inc"
        ;;
        "ods_comment_info_inc")
            load_data "ods_comment_info_inc"
        ;;
        "ods_coupon_use_inc")
            load_data "ods_coupon_use_inc"
        ;;
        "ods_favor_info_inc")
            load_data "ods_favor_info_inc"
        ;;
        "ods_order_detail_inc")
            load_data "ods_order_detail_inc"
        ;;
        "ods_order_detail_activity_inc")
            load_data "ods_order_detail_activity_inc"
        ;;
        "ods_order_detail_coupon_inc")
            load_data "ods_order_detail_coupon_inc"
        ;;
        "ods_order_info_inc")
            load_data "ods_order_info_inc"
        ;;
        "ods_order_refund_info_inc")
            load_data "ods_order_refund_info_inc"
        ;;
        "ods_order_status_log_inc")
            load_data "ods_order_status_log_inc"
        ;;
        "ods_payment_info_inc")
            load_data "ods_payment_info_inc"
        ;;
        "ods_refund_payment_inc")
            load_data "ods_refund_payment_inc"
        ;;
        "ods_user_info_inc")
            load_data "ods_user_info_inc"
        ;;
        "all")
            load_data "ods_activity_info_full" "ods_activity_rule_full" "ods_base_category1_full" "ods_base_category2_full" "ods_base_category3_full" "ods_base_dic_full" "ods_base_province_full" "ods_base_region_full" "ods_base_trademark_full" "ods_cart_info_full" "ods_coupon_info_full" "ods_sku_attr_value_full" "ods_sku_info_full" "ods_sku_sale_attr_value_full" "ods_spu_info_full" "ods_cart_info_inc" "ods_comment_info_inc" "ods_coupon_use_inc" "ods_favor_info_inc" "ods_order_detail_inc" "ods_order_detail_activity_inc" "ods_order_detail_coupon_inc" "ods_order_info_inc" "ods_order_refund_info_inc" "ods_order_status_log_inc" "ods_payment_info_inc" "ods_refund_payment_inc" "ods_user_info_inc"
        ;;
    esac
    ```

    执行：
    ```shell
    hdfs_to_ods_db.sh all 2020-06-14
    ```