Facebook_SQL_CodeSample
=======================
with elig as
(
Select group_id,
       create_dt_tm as Start_tm, 
       runtime_fh, 
       to_number(to_char(CREATE_DT_TM, 'SS')) as create_second,
      create_dt_tm + (runtime_fh/100000) as End_tm
      
                                      from services
                                     where SERVICE_METHOD_ID = XXXXXX 
                                     and to_number(to_char(CREATE_DT_TM, 'YYYY')) = '2014'
                                     and to_number(to_char(CREATE_DT_TM, 'MM')) = '11'
                                     and to_number(to_char(CREATE_DT_TM, 'DD')) = '17'
                                     and runtime_fh < 120
                    order by Start_tm
),

datetable as
(
    select to_date('17-11-2014 00:00:00','dd-mm-yy hh24:mi:ss') + rownum/86400  as dat from dual connect by rownum<=86400
)


select datetable.dat ,
       ( Case when datetable.dat <= to_date('17-11-2014 07:00:00','dd-mm-yy hh24:mi:ss') then 'Before 7am'
        when datetable.dat between to_date('17-11-2014 07:00:00','dd-mm-yy hh24:mi:ss ') and to_date('17-11-2014 21:00:00','dd-mm-yy hh24:mi:ss') then 'Between 7am and 9pm'
        when datetable.dat > to_date('17-11-2014 21:00:0','dd-mm-yy hh24:mi:ss') then 'After 9pm' end) as Time_of_day,
              (select count(distinct e.group_id) 
              from datetable d, elig e        
              where dat between e.Start_tm and 
              (e.Start_tm + e.runtime_fh/100000)
              and d.dat=datetable.dat
               ) Trxn_count
        
                  ,(select round(avg(e.runtime_fh),2) 
                  from datetable d, elig e        
                  where dat between e.Start_tm and 
                  (e.Start_tm + e.runtime_fh/100000)
                  and d.dat=datetable.dat
                  ) ARTime
                  
