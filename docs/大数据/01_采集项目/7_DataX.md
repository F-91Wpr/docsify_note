0. 准备
   
    安装驱动：

    ```shell
    sudo yum install -y MySQL-python
    ```

1. 自动生成 DataX job 配置文件的脚本`gen_import_config.py`：

    ```shell
    vim ~/bin/gen_import_config.py
    ```

    ```python
    # coding=utf-8
    import json
    import getopt
    import os
    import sys
    import MySQLdb

    #MySQL相关配置，需根据实际情况作出修改
    mysql_host = "bd102"
    mysql_port = "3306"
    mysql_user = "root"
    mysql_passwd = "000000"

    #HDFS NameNode相关配置，需根据实际情况作出修改
    hdfs_nn_host = "bd102"
    hdfs_nn_port = "8020"

    #生成配置文件的目标路径，可根据实际情况作出修改
    output_path = "/opt/module/datax/job/import"


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


    def get_hive_columns(database, table):
        def type_mapping(mysql_type):
            mappings = {
                "bigint": "bigint",
                "int": "bigint",
                "smallint": "bigint",
                "tinyint": "bigint",
                "decimal": "string",
                "double": "double",
                "float": "float",
                "binary": "string",
                "char": "string",
                "varchar": "string",
                "datetime": "string",
                "time": "string",
                "timestamp": "string",
                "date": "string",
                "text": "string"
            }
            return mappings[mysql_type]

        meta = get_mysql_meta(database, table)
        return map(lambda x: {"name": x[0], "type": type_mapping(x[1].lower())}, meta)


    def generate_json(source_database, source_table):
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
                        "name": "mysqlreader",
                        "parameter": {
                            "username": mysql_user,
                            "password": mysql_passwd,
                            "column": get_mysql_columns(source_database, source_table),
                            "splitPk": "",
                            "connection": [{
                                "table": [source_table],
                                "jdbcUrl": ["jdbc:mysql://" + mysql_host + ":" + mysql_port + "/" + source_database]
                            }]
                        }
                    },
                    "writer": {
                        "name": "hdfswriter",
                        "parameter": {
                            "defaultFS": "hdfs://" + hdfs_nn_host + ":" + hdfs_nn_port,
                            "fileType": "text",
                            "path": "${targetdir}",
                            "fileName": source_table,
                            "column": get_hive_columns(source_database, source_table),
                            "writeMode": "append",
                            "fieldDelimiter": "\t",
                            "compress": "gzip"
                        }
                    }
                }]
            }
        }
        if not os.path.exists(output_path):
            os.makedirs(output_path)
        with open(os.path.join(output_path, ".".join([source_database, source_table, "json"])), "w") as f:
            json.dump(job, f)


    def main(args):
        source_database = ""
        source_table = ""

        options, arguments = getopt.getopt(args, '-d:-t:', ['sourcedb=', 'sourcetbl='])
        for opt_name, opt_value in options:
            if opt_name in ('-d', '--sourcedb'):
                source_database = opt_value
            if opt_name in ('-t', '--sourcetbl'):
                source_table = opt_value

        generate_json(source_database, source_table)


    if __name__ == '__main__':
        main(sys.argv[1:])
    ```
   
    脚本使用说明：

    ```shell
    python gen_import_config.py -d database -t table
    ```
    通过`-d`传入数据库名，`-t`传入表名，执行上述命令即可生成该表的 DataX 同步配置文件。

2. 批量生成配置文件脚本`gen_import_config.sh`

    ```shell
    vim ~/bin/gen_import_config.sh
    ```

    ```shell
    #!/bin/bash

    python ~/bin/gen_import_config.py -d gmall -t activity_info
    python ~/bin/gen_import_config.py -d gmall -t activity_rule
    python ~/bin/gen_import_config.py -d gmall -t base_category1
    python ~/bin/gen_import_config.py -d gmall -t base_category2
    python ~/bin/gen_import_config.py -d gmall -t base_category3
    python ~/bin/gen_import_config.py -d gmall -t base_dic
    python ~/bin/gen_import_config.py -d gmall -t base_province
    python ~/bin/gen_import_config.py -d gmall -t base_region
    python ~/bin/gen_import_config.py -d gmall -t base_trademark
    python ~/bin/gen_import_config.py -d gmall -t cart_info
    python ~/bin/gen_import_config.py -d gmall -t coupon_info
    python ~/bin/gen_import_config.py -d gmall -t sku_attr_value
    python ~/bin/gen_import_config.py -d gmall -t sku_info
    python ~/bin/gen_import_config.py -d gmall -t sku_sale_attr_value
    python ~/bin/gen_import_config.py -d gmall -t spu_info
    ```

    执行`gen_import_config.sh`，生成15张全量表同步配置文件。

3. 全量表同步脚本`mysql_to_hdfs_full.sh`

    ```shell
    vim ~/bin/mysql_to_hdfs_full.sh
    ```

    ```shell
    #!/bin/bash

    DATAX_HOME=/opt/module/datax

    # 如果传入日期则do_date等于传入的日期，否则等于前一天日期
    if [ -n "$2" ] ;then
        do_date=$2
    else
        do_date=`date -d "-1 day" +%F`
    fi

    #处理目标路径，此处的处理逻辑是，如果目标路径不存在，则创建；若存在，则清空，目的是保证同步任务可重复执行
    handle_targetdir() {
    hadoop fs -test -e $1
    if [[ $? -eq 1 ]]; then
        echo "路径$1不存在，正在创建......"
        hadoop fs -mkdir -p $1
    else
        echo "路径$1已经存在"
        fs_count=$(hadoop fs -count $1)
        content_size=$(echo $fs_count | awk '{print $3}')
        if [[ $content_size -eq 0 ]]; then
        echo "路径$1为空"
        else
        echo "路径$1不为空，正在清空......"
        hadoop fs -rm -r -f $1/*
        fi
    fi
    }

    #数据同步
    import_data() {
    datax_config=$1
    target_dir=$2

    handle_targetdir $target_dir
    python $DATAX_HOME/bin/datax.py -p"-Dtargetdir=$target_dir" $datax_config
    }

    case $1 in
    "activity_info")
    import_data /opt/module/datax/job/import/gmall.activity_info.json /origin_data/gmall/db/activity_info_full/$do_date
    ;;
    "activity_rule")
    import_data /opt/module/datax/job/import/gmall.activity_rule.json /origin_data/gmall/db/activity_rule_full/$do_date
    ;;
    "base_category1")
    import_data /opt/module/datax/job/import/gmall.base_category1.json /origin_data/gmall/db/base_category1_full/$do_date
    ;;
    "base_category2")
    import_data /opt/module/datax/job/import/gmall.base_category2.json /origin_data/gmall/db/base_category2_full/$do_date
    ;;
    "base_category3")
    import_data /opt/module/datax/job/import/gmall.base_category3.json /origin_data/gmall/db/base_category3_full/$do_date
    ;;
    "base_dic")
    import_data /opt/module/datax/job/import/gmall.base_dic.json /origin_data/gmall/db/base_dic_full/$do_date
    ;;
    "base_province")
    import_data /opt/module/datax/job/import/gmall.base_province.json /origin_data/gmall/db/base_province_full/$do_date
    ;;
    "base_region")
    import_data /opt/module/datax/job/import/gmall.base_region.json /origin_data/gmall/db/base_region_full/$do_date
    ;;
    "base_trademark")
    import_data /opt/module/datax/job/import/gmall.base_trademark.json /origin_data/gmall/db/base_trademark_full/$do_date
    ;;
    "cart_info")
    import_data /opt/module/datax/job/import/gmall.cart_info.json /origin_data/gmall/db/cart_info_full/$do_date
    ;;
    "coupon_info")
    import_data /opt/module/datax/job/import/gmall.coupon_info.json /origin_data/gmall/db/coupon_info_full/$do_date
    ;;
    "sku_attr_value")
    import_data /opt/module/datax/job/import/gmall.sku_attr_value.json /origin_data/gmall/db/sku_attr_value_full/$do_date
    ;;
    "sku_info")
    import_data /opt/module/datax/job/import/gmall.sku_info.json /origin_data/gmall/db/sku_info_full/$do_date
    ;;
    "sku_sale_attr_value")
    import_data /opt/module/datax/job/import/gmall.sku_sale_attr_value.json /origin_data/gmall/db/sku_sale_attr_value_full/$do_date
    ;;
    "spu_info")
    import_data /opt/module/datax/job/import/gmall.spu_info.json /origin_data/gmall/db/spu_info_full/$do_date
    ;;
    "all")
    import_data /opt/module/datax/job/import/gmall.activity_info.json /origin_data/gmall/db/activity_info_full/$do_date
    import_data /opt/module/datax/job/import/gmall.activity_rule.json /origin_data/gmall/db/activity_rule_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_category1.json /origin_data/gmall/db/base_category1_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_category2.json /origin_data/gmall/db/base_category2_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_category3.json /origin_data/gmall/db/base_category3_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_dic.json /origin_data/gmall/db/base_dic_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_province.json /origin_data/gmall/db/base_province_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_region.json /origin_data/gmall/db/base_region_full/$do_date
    import_data /opt/module/datax/job/import/gmall.base_trademark.json /origin_data/gmall/db/base_trademark_full/$do_date
    import_data /opt/module/datax/job/import/gmall.cart_info.json /origin_data/gmall/db/cart_info_full/$do_date
    import_data /opt/module/datax/job/import/gmall.coupon_info.json /origin_data/gmall/db/coupon_info_full/$do_date
    import_data /opt/module/datax/job/import/gmall.sku_attr_value.json /origin_data/gmall/db/sku_attr_value_full/$do_date
    import_data /opt/module/datax/job/import/gmall.sku_info.json /origin_data/gmall/db/sku_info_full/$do_date
    import_data /opt/module/datax/job/import/gmall.sku_sale_attr_value.json /origin_data/gmall/db/sku_sale_attr_value_full/$do_date
    import_data /opt/module/datax/job/import/gmall.spu_info.json /origin_data/gmall/db/spu_info_full/$do_date
    ;;
    esac
    ```

    执行：

    ```shell
    mysql_to_hdfs_full.sh all 2020-06-14
    ```

4. 观察 hdfs 目录


