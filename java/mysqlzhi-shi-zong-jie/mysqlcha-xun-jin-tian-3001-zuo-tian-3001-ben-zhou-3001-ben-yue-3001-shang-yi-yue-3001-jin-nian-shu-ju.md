# Mysql查询今天、昨天、本周、本月、上一月 、今年数据

    --今天

    select * from 表名 where to_days(时间字段名) = to_days(now());

    --昨天

    SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1

    --本周

    SELECT * FROM  表名 WHERE YEARWEEK( date_format(  时间字段名,'%Y-%m-%d' ) ) = YEARWEEK( now() ) ;

    --本月

    SELECT * FROM  表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) ,'%Y%m' ) 

    --上一个月

    SELECT * FROM  表名 WHERE PERIOD_DIFF(date_format(now(),'%Y%m'),date_format(时间字段名,'%Y%m') =1

    --本年

    SELECT * FROM 表名 WHERE YEAR(  时间字段名 ) = YEAR( NOW( ) ) 


    --上一月

    SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1



    --查询本季度数据
    select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());
    --查询上季度数据
    select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));
    --查询本年数据
    select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());
    --查询上年数据
    select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));




    --查询当前这周的数据 
    SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());

    --查询上周的数据
    SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;

    --查询当前月份的数据
    select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m')

    --查询距离当前现在6个月的数据
    select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now();

    --查询上个月的数据
    select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m')

    select * from ` user ` where DATE_FORMAT(pudate, ' %Y%m ' ) = DATE_FORMAT(CURDATE(), ' %Y%m ' ) ;

    select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now())

    select * 
    from user 
    where MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now())

    select * 
    from [ user ] 
    where YEAR (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = YEAR (now())
    and MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now())

    select * 
    from [ user ] 
    where pudate between 上月最后一天
    and 下月第一天

    where   date(regdate)   =   curdate();

    select   *   from   test   where   year(regdate)=year(now())   and   month(regdate)=month(now())   and   day(regdate)=day(now())

    SELECT date( c_instime ) ,curdate( )
    FROM `t_score`
    WHERE 1
    LIMIT 0 , 30



