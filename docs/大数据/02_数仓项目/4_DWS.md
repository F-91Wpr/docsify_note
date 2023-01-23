## 脚本

### 最近一日汇总表

1. 首日数据装载脚本`dwd_to_dws_1d_init.sh`

    ```shell
    #!/bin/bash
    APP=gmall

    if [ -n "$2" ] ;then
    do_date=$2
    else 
    echo "请传入日期参数"
    exit
    fi

    dws_trade_province_order_1d="
    insert overwrite table ${APP}.dws_trade_province_order_1d partition(dt)
    select
        province_id,
        province_name,
        area_code,
        iso_code,
        iso_3166_2,
        order_count_1d,
        order_original_amount_1d,
        activity_reduce_amount_1d,
        coupon_reduce_amount_1d,
        order_total_amount_1d,
        dt
    from
    (
        select
            province_id,
            count(distinct(order_id)) order_count_1d,
            sum(split_original_amount) order_original_amount_1d,
            sum(nvl(split_activity_amount,0)) activity_reduce_amount_1d,
            sum(nvl(split_coupon_amount,0)) coupon_reduce_amount_1d,
            sum(split_total_amount) order_total_amount_1d,
            dt
        from ${APP}.dwd_trade_order_detail_inc
        group by province_id,dt
    )o
    left join
    (
        select
            id,
            province_name,
            area_code,
            iso_code,
            iso_3166_2
        from ${APP}.dim_province_full
        where dt='$do_date'
    )p
    on o.province_id=p.id;
    "
    dws_trade_user_cart_add_1d="
    insert overwrite table ${APP}.dws_trade_user_cart_add_1d partition(dt)
    select
        user_id,
        count(*),
        sum(sku_num),
        dt
    from ${APP}.dwd_trade_cart_add_inc
    group by user_id,dt;
    "
    dws_trade_user_order_1d="
    insert overwrite table ${APP}.dws_trade_user_order_1d partition(dt)
    select
        user_id,
        count(distinct(order_id)),
        sum(sku_num),
        sum(split_original_amount),
        sum(nvl(split_activity_amount,0)),
        sum(nvl(split_coupon_amount,0)),
        sum(split_total_amount),
        dt
    from ${APP}.dwd_trade_order_detail_inc
    group by user_id,dt;
    "

    dws_trade_user_payment_1d="
    insert overwrite table ${APP}.dws_trade_user_payment_1d partition(dt)
    select
        user_id,
        count(distinct(order_id)),
        sum(sku_num),
        sum(split_payment_amount),
        dt
    from ${APP}.dwd_trade_pay_detail_suc_inc
    group by user_id,dt;
    "
    dws_trade_user_sku_order_1d="
    insert overwrite table ${APP}.dws_trade_user_sku_order_1d partition(dt)
    select
        user_id,
        id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name,
        order_count_1d,
        order_num_1d,
        order_original_amount_1d,
        activity_reduce_amount_1d,
        coupon_reduce_amount_1d,
        order_total_amount_1d,
        dt
    from
    (
        select
            dt,
            user_id,
            sku_id,
            count(*) order_count_1d,
            sum(sku_num) order_num_1d,
            sum(split_original_amount) order_original_amount_1d,
            sum(nvl(split_activity_amount,0.0)) activity_reduce_amount_1d,
            sum(nvl(split_coupon_amount,0.0)) coupon_reduce_amount_1d,
            sum(split_total_amount) order_total_amount_1d
        from ${APP}.dwd_trade_order_detail_inc
        group by dt,user_id,sku_id
    )od
    left join
    (
        select
            id,
            sku_name,
            category1_id,
            category1_name,
            category2_id,
            category2_name,
            category3_id,
            category3_name,
            tm_id,
            tm_name
        from ${APP}.dim_sku_full
        where dt='$do_date'
    )sku
    on od.sku_id=sku.id;
    "

    dws_tool_user_coupon_coupon_used_1d="
    insert overwrite table ${APP}.dws_tool_user_coupon_coupon_used_1d partition(dt)
    select
        user_id,
        coupon_id,
        coupon_name,
        coupon_type_code,
        coupon_type_name,
        benefit_rule,
        used_count,
        dt
    from
    (
        select
            dt,
            user_id,
            coupon_id,
            count(*) used_count
        from ${APP}.dwd_tool_coupon_used_inc
        group by dt,user_id,coupon_id
    )t1
    left join
    (
        select
            id,
            coupon_name,
            coupon_type_code,
            coupon_type_name,
            benefit_rule
        from ${APP}.dim_coupon_full
        where dt='$do_date'
    )t2
    on t1.coupon_id=t2.id;
    "
    dws_interaction_sku_favor_add_1d="
    insert overwrite table ${APP}.dws_interaction_sku_favor_add_1d partition(dt)
    select
        sku_id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name,
        favor_add_count,
        dt
    from
    (
        select
            dt,
            sku_id,
            count(*) favor_add_count
        from ${APP}.dwd_interaction_favor_add_inc
        group by dt,sku_id
    )favor
    left join
    (
        select
            id,
            sku_name,
            category1_id,
            category1_name,
            category2_id,
            category2_name,
            category3_id,
            category3_name,
            tm_id,
            tm_name
        from ${APP}.dim_sku_full
        where dt='$do_date'
    )sku
    on favor.sku_id=sku.id;
    "

    dws_traffic_page_visitor_page_view_1d="
    insert overwrite table ${APP}.dws_traffic_page_visitor_page_view_1d partition(dt='$do_date')
    select
        mid_id,
        brand,
        model,
        operate_system,
        page_id,
        sum(during_time),
        count(*)
    from ${APP}.dwd_traffic_page_view_inc
    where dt='$do_date'
    group by mid_id,brand,model,operate_system,page_id;
    "
    dws_traffic_session_page_view_1d="
    insert overwrite table ${APP}.dws_traffic_session_page_view_1d partition(dt='$do_date')
    select
        session_id,
        mid_id,
        brand,
        model,
        operate_system,
        version_code,
        channel,
        sum(during_time),
        count(*)
    from ${APP}.dwd_traffic_page_view_inc
    where dt='$do_date'
    group by session_id,mid_id,brand,model,operate_system,version_code,channel;
    "

    case $1 in
        "dws_trade_province_order_1d" )
            hive -e "$dws_trade_province_order_1d"
        ;;
        "dws_trade_user_cart_add_1d" )
            hive -e "$dws_trade_user_cart_add_1d"
        ;;
        "dws_trade_user_order_1d" )
            hive -e "$dws_trade_user_order_1d"
        ;;
        "dws_trade_user_payment_1d" )
            hive -e "$dws_trade_user_payment_1d"
        ;;
        "dws_trade_user_sku_order_1d" )
            hive -e "$dws_trade_user_sku_order_1d"
        ;;
        "dws_tool_user_coupon_coupon_used_1d" )
            hive -e "$dws_tool_user_coupon_coupon_used_1d"
        ;;
        "dws_interaction_sku_favor_add_1d" )
            hive -e "$dws_interaction_sku_favor_add_1d"
        ;;
        "dws_traffic_page_visitor_page_view_1d" )
            hive -e "$dws_traffic_page_visitor_page_view_1d"
        ;;
        "dws_traffic_session_page_view_1d" )
            hive -e "$dws_traffic_session_page_view_1d"
        ;;
        "all" )
            hive -e "$dws_trade_province_order_1d$dws_trade_user_cart_add_1d$dws_trade_user_order_1d$dws_trade_user_payment_1d$dws_trade_user_sku_order_1d$dws_tool_user_coupon_coupon_used_1d$dws_interaction_sku_favor_add_1d$dws_traffic_page_visitor_page_view_1d$dws_traffic_session_page_view_1d"
        ;;
    esac
    ```

    执行：

    ```shell
    dwd_to_dws_1d_init.sh all 2020-06-14
    ```

2. 每日数据装载脚本`dwd_to_dws_1d.sh`

    ```shell
    #!/bin/bash
    APP=gmall

    # 如果输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
    if [ -n "$2" ] ;then
        do_date=$2
    else 
        do_date=`date -d "-1 day" +%F`
    fi

    dws_trade_province_order_1d="
    insert overwrite table ${APP}.dws_trade_province_order_1d partition(dt='$do_date')
    select
        province_id,
        province_name,
        area_code,
        iso_code,
        iso_3166_2,
        order_count_1d,
        order_original_amount_1d,
        activity_reduce_amount_1d,
        coupon_reduce_amount_1d,
        order_total_amount_1d
    from
    (
        select
            province_id,
            count(distinct(order_id)) order_count_1d,
            sum(split_original_amount) order_original_amount_1d,
            sum(nvl(split_activity_amount,0)) activity_reduce_amount_1d,
            sum(nvl(split_coupon_amount,0)) coupon_reduce_amount_1d,
            sum(split_total_amount) order_total_amount_1d
        from ${APP}.dwd_trade_order_detail_inc
        where dt='$do_date'
        group by province_id
    )o
    left join
    (
        select
            id,
            province_name,
            area_code,
            iso_code,
            iso_3166_2
        from ${APP}.dim_province_full
        where dt='$do_date'
    )p
    on o.province_id=p.id;
    "
    dws_trade_user_cart_add_1d="
    insert overwrite table ${APP}.dws_trade_user_cart_add_1d partition(dt='$do_date')
    select
        user_id,
        count(*),
        sum(sku_num)
    from ${APP}.dwd_trade_cart_add_inc
    where dt='$do_date'
    group by user_id;
    "
    dws_trade_user_order_1d="
    insert overwrite table ${APP}.dws_trade_user_order_1d partition(dt='$do_date')
    select
        user_id,
        count(distinct(order_id)),
        sum(sku_num),
        sum(split_original_amount),
        sum(nvl(split_activity_amount,0)),
        sum(nvl(split_coupon_amount,0)),
        sum(split_total_amount)
    from ${APP}.dwd_trade_order_detail_inc
    where dt='$do_date'
    group by user_id;
    "

    dws_trade_user_payment_1d="
    insert overwrite table ${APP}.dws_trade_user_payment_1d partition(dt='$do_date')
    select
        user_id,
        count(distinct(order_id)),
        sum(sku_num),
        sum(split_payment_amount)
    from ${APP}.dwd_trade_pay_detail_suc_inc
    where dt='$do_date'
    group by user_id;
    "
    dws_trade_user_sku_order_1d="
    insert overwrite table ${APP}.dws_trade_user_sku_order_1d partition(dt='$do_date')
    select
        user_id,
        id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name,
        order_count,
        order_num,
        order_original_amount,
        activity_reduce_amount,
        coupon_reduce_amount,
        order_total_amount
    from
    (
        select
            user_id,
            sku_id,
            count(*) order_count,
            sum(sku_num) order_num,
            sum(split_original_amount) order_original_amount,
            sum(nvl(split_activity_amount,0)) activity_reduce_amount,
            sum(nvl(split_coupon_amount,0)) coupon_reduce_amount,
            sum(split_total_amount) order_total_amount
        from ${APP}.dwd_trade_order_detail_inc
        where dt='$do_date'
        group by user_id,sku_id
    )od
    left join
    (
        select
            id,
            sku_name,
            category1_id,
            category1_name,
            category2_id,
            category2_name,
            category3_id,
            category3_name,
            tm_id,
            tm_name
        from ${APP}.dim_sku_full
        where dt='$do_date'
    )sku
    on od.sku_id=sku.id;
    "

    dws_interaction_sku_favor_add_1d="
    insert overwrite table ${APP}.dws_interaction_sku_favor_add_1d partition(dt='$do_date')
    select
        sku_id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name,
        favor_add_count
    from
    (
        select
            sku_id,
            count(*) favor_add_count
        from ${APP}.dwd_interaction_favor_add_inc
        where dt='$do_date'
        group by sku_id
    )favor
    left join
    (
        select
            id,
            sku_name,
            category1_id,
            category1_name,
            category2_id,
            category2_name,
            category3_id,
            category3_name,
            tm_id,
            tm_name
        from ${APP}.dim_sku_full
        where dt='$do_date'
    )sku
    on favor.sku_id=sku.id;
    "

    dws_tool_user_coupon_coupon_used_1d="
    insert overwrite table ${APP}.dws_tool_user_coupon_coupon_used_1d partition(dt='$do_date')
    select
        user_id,
        coupon_id,
        coupon_name,
        coupon_type_code,
        coupon_type_name,
        benefit_rule,
        used_count
    from
    (
        select
            user_id,
            coupon_id,
            count(*) used_count
        from ${APP}.dwd_tool_coupon_used_inc
        where dt='$do_date'
        group by user_id,coupon_id
    )t1
    left join
    (
        select
            id,
            coupon_name,
            coupon_type_code,
            coupon_type_name,
            benefit_rule
        from ${APP}.dim_coupon_full
        where dt='$do_date'
    )t2
    on t1.coupon_id=t2.id;
    "

    dws_traffic_page_visitor_page_view_1d="
    insert overwrite table ${APP}.dws_traffic_page_visitor_page_view_1d partition(dt='$do_date')
    select
        mid_id,
        brand,
        model,
        operate_system,
        page_id,
        sum(during_time),
        count(*)
    from ${APP}.dwd_traffic_page_view_inc
    where dt='$do_date'
    group by mid_id,brand,model,operate_system,page_id;
    "
    dws_traffic_session_page_view_1d="
    insert overwrite table ${APP}.dws_traffic_session_page_view_1d partition(dt='$do_date')
    select
        session_id,
        mid_id,
        brand,
        model,
        operate_system,
        version_code,
        channel,
        sum(during_time),
        count(*)
    from ${APP}.dwd_traffic_page_view_inc
    where dt='$do_date'
    group by session_id,mid_id,brand,model,operate_system,version_code,channel;
    "

    case $1 in
        "dws_trade_province_order_1d" )
            hive -e "$dws_trade_province_order_1d"
        ;;
        "dws_trade_user_cart_add_1d" )
            hive -e "$dws_trade_user_cart_add_1d"
        ;;
        "dws_trade_user_order_1d" )
            hive -e "$dws_trade_user_order_1d"
        ;;
        "dws_trade_user_payment_1d" )
            hive -e "$dws_trade_user_payment_1d"
        ;;
        "dws_trade_user_sku_order_1d" )
            hive -e "$dws_trade_user_sku_order_1d"
        ;;
        "dws_tool_user_coupon_coupon_used_1d" )
            hive -e "$dws_tool_user_coupon_coupon_used_1d"
        ;;
        "dws_interaction_sku_favor_add_1d" )
            hive -e "$dws_interaction_sku_favor_add_1d"
        ;;
        "dws_traffic_page_visitor_page_view_1d" )
            hive -e "$dws_traffic_page_visitor_page_view_1d"
        ;;
        "dws_traffic_session_page_view_1d" )
            hive -e "$dws_traffic_session_page_view_1d"
        ;;
        "all" )
            hive -e "$dws_trade_province_order_1d$dws_trade_user_cart_add_1d$dws_trade_user_order_1d$dws_trade_user_payment_1d$dws_trade_user_sku_order_1d$dws_tool_user_coupon_coupon_used_1d$dws_interaction_sku_favor_add_1d$dws_traffic_page_visitor_page_view_1d$dws_traffic_session_page_view_1d"
        ;;
    esac
    ```

    执行：
    ```shell
    dwd_to_dws_1d.sh all 2020-06-14
    ```

### 最近 n 日汇总表

1. 每日数据装载脚本`dws_1d_to_dws_nd.sh`

    ```shell
    #!/bin/bash
    APP=gmall

    # 如果是输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
    if [ -n "$2" ] ;then
        do_date=$2
    else 
        do_date=`date -d "-1 day" +%F`
    fi

    dws_trade_province_order_nd="
    insert overwrite table ${APP}.dws_trade_province_order_nd partition(dt='$do_date')
    select
        province_id,
        province_name,
        area_code,
        iso_code,
        iso_3166_2,
        sum(if(dt>=date_add('$do_date',-6),order_count_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),order_original_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),activity_reduce_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),coupon_reduce_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),order_total_amount_1d,0)),
        sum(order_count_1d),
        sum(order_original_amount_1d),
        sum(activity_reduce_amount_1d),
        sum(coupon_reduce_amount_1d),
        sum(order_total_amount_1d)
    from ${APP}.dws_trade_province_order_1d
    where dt>=date_add('$do_date',-29)
    and dt<='$do_date'
    group by province_id,province_name,area_code,iso_code,iso_3166_2;
    "

    dws_trade_user_sku_order_nd="
    insert overwrite table ${APP}.dws_trade_user_sku_order_nd partition(dt='$do_date')
    select
        user_id,
        sku_id,
        sku_name,
        category1_id,
        category1_name,
        category2_id,
        category2_name,
        category3_id,
        category3_name,
        tm_id,
        tm_name,
        sum(if(dt>=date_add('$do_date',-6),order_count_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),order_num_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),order_original_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),activity_reduce_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),coupon_reduce_amount_1d,0)),
        sum(if(dt>=date_add('$do_date',-6),order_total_amount_1d,0)),
        sum(order_count_1d),
        sum(order_num_1d),
        sum(order_original_amount_1d),
        sum(activity_reduce_amount_1d),
        sum(coupon_reduce_amount_1d),
        sum(order_total_amount_1d)
    from ${APP}.dws_trade_user_sku_order_1d
    where dt>=date_add('$do_date',-30)
    group by  user_id,sku_id,sku_name,category1_id,category1_name,category2_id,category2_name,category3_id,category3_name,tm_id,tm_name;
    "

    case $1 in
        "dws_trade_province_order_nd" )
            hive -e "$dws_trade_province_order_nd"
        ;;
        "dws_trade_user_sku_order_nd" )
            hive -e "$dws_trade_user_sku_order_nd"
        ;;
        "all" )
            hive -e "$dws_trade_province_order_nd$dws_trade_user_sku_order_nd"
        ;;
    esac
    ```
    执行：
    ```shell
    dws_1d_to_dws_nd.sh all 2020-06-14
    ```

### 历史至今汇总表

1. 首日数据装载脚本`dws_1d_to_dws_td_init.sh`

    ```shell
    #!/bin/bash
    APP=gmall

    if [ -n "$2" ] ;then
    do_date=$2
    else 
    echo "请传入日期参数"
    exit
    fi

    dws_trade_user_order_td="
    insert overwrite table ${APP}.dws_trade_user_order_td partition(dt='$do_date')
    select
        user_id,
        min(dt) login_date_first,
        max(dt) login_date_last,
        sum(order_count_1d) order_count,
        sum(order_num_1d) order_num,
        sum(order_original_amount_1d) original_amount,
        sum(activity_reduce_amount_1d) activity_reduce_amount,
        sum(coupon_reduce_amount_1d) coupon_reduce_amount,
        sum(order_total_amount_1d) total_amount
    from ${APP}.dws_trade_user_order_1d
    group by user_id;
    "


    dws_user_user_login_td="
    insert overwrite table ${APP}.dws_user_user_login_td partition(dt='$do_date')
    select
        u.id,
        nvl(login_date_last,date_format(create_time,'yyyy-MM-dd')),
        nvl(login_count_td,1)
    from
    (
        select
            id,
            create_time
        from ${APP}.dim_user_zip
        where dt='9999-12-31'
    )u
    left join
    (
        select
            user_id,
            max(dt) login_date_last,
            count(*) login_count_td
        from ${APP}.dwd_user_login_inc
        group by user_id
    )l
    on u.id=l.user_id;
    "

    case $1 in
        "dws_trade_user_order_td" )
            hive -e "$dws_trade_user_order_td"
        ;;
        "dws_user_user_login_td" )
            hive -e "$dws_user_user_login_td"
        ;;
        "all" )
            hive -e "$dws_trade_user_order_td$dws_user_user_login_td"
        ;;
    esac
    ```

    执行：
    ```shell
    dws_1d_to_dws_td_init.sh all 2020-06-14
    ```

2. 每日数据装载脚本`dws_1d_to_dws_td.sh`

    ```shell
    #!/bin/bash
    APP=gmall

    # 如果输入的日期按照取输入日期；如果没输入日期取当前时间的前一天
    if [ -n "$2" ] ;then
        do_date=$2
    else 
        do_date=`date -d "-1 day" +%F`
    fi

    dws_trade_user_order_td="
    insert overwrite table ${APP}.dws_trade_user_order_td partition(dt='$do_date')
    select
        nvl(old.user_id,new.user_id),
        if(new.user_id is not null and old.user_id is null,'$do_date',old.order_date_first),
        if(new.user_id is not null,'$do_date',old.order_date_last),
        nvl(old.order_count_td,0)+nvl(new.order_count_1d,0),
        nvl(old.order_num_td,0)+nvl(new.order_num_1d,0),
        nvl(old.original_amount_td,0)+nvl(new.order_original_amount_1d,0),
        nvl(old.activity_reduce_amount_td,0)+nvl(new.activity_reduce_amount_1d,0),
        nvl(old.coupon_reduce_amount_td,0)+nvl(new.coupon_reduce_amount_1d,0),
        nvl(old.total_amount_td,0)+nvl(new.order_total_amount_1d,0)
    from
    (
        select
            user_id,
            order_date_first,
            order_date_last,
            order_count_td,
            order_num_td,
            original_amount_td,
            activity_reduce_amount_td,
            coupon_reduce_amount_td,
            total_amount_td
        from ${APP}.dws_trade_user_order_td
        where dt=date_add('$do_date',-1)
    )old
    full outer join
    (
        select
            user_id,
            order_count_1d,
            order_num_1d,
            order_original_amount_1d,
            activity_reduce_amount_1d,
            coupon_reduce_amount_1d,
            order_total_amount_1d
        from ${APP}.dws_trade_user_order_1d
        where dt='$do_date'
    )new
    on old.user_id=new.user_id;
    "

    dws_user_user_login_td="
    insert overwrite table ${APP}.dws_user_user_login_td partition(dt='$do_date')
    select
        nvl(old.user_id,new.user_id),
        if(new.user_id is null,old.login_date_last,'$do_date'),
        nvl(old.login_count_td,0)+nvl(new.login_count_1d,0)
    from
    (
        select
            user_id,
            login_date_last,
            login_count_td
        from ${APP}.dws_user_user_login_td
        where dt=date_add('$do_date',-1)
    )old
    full outer join
    (
        select
            user_id,
            count(*) login_count_1d
        from ${APP}.dwd_user_login_inc
        where dt='$do_date'
        group by user_id
    )new
    on old.user_id=new.user_id;
    "

    case $1 in
        "dws_trade_user_order_td" )
            hive -e "$dws_trade_user_order_td"
        ;;
        "dws_user_user_login_td" )
            hive -e "$dws_user_user_login_td"
        ;;
        "all" )
            hive -e "$dws_trade_user_order_td$dws_user_user_login_td"
        ;;
    esac
    ```

    执行：
    ```shell
    dws_1d_to_dws_td.sh all 2020-06-14
    ```
