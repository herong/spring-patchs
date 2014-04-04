【问题描述】
    使用spring SimpleJdbcCall 在调用存储过程时很慢。

【问题分析】
     SimpleJdbcCall 在调用存储过程或函数时，会调用jdbc驱动程序以下两个方法获取此对象的相关信息（参数名和类型等)。
          getProcedures
          getProcedureColumns
     这两个方法查询oracle 10g 对象时特别慢，不同驱动版本都试过了。在9i,11g稳好点，如果在要求更高的话还是可以优化。
    
【问题追踪】

      使用jdbc debug版本，通常是以_g经尾。如oracle :ojdbc5_g.jar


配置日志参数：
.level=FINER
oracle.jdbc.level=FINER
oracle.jdbc.handlers=java.util.logging.FileHandler
java.util.logging.FileHandler.level=FINER
java.util.logging.FileHandler.pattern=E:/jdbc.log
java.util.logging.FileHandler.count=1
java.util.logging.FileHandler.formatter=java.util.logging.SimpleFormatter


指定jvm参数：
-Djava.util.logging.config.file=E:/OracleLog.properties -Doracle.jdbc.Trace=true


输出日志：
 SELECT package_name AS procedure_cat,
       owner AS procedure_schem,
       object_name AS procedure_name,
       argument_name AS column_name,
       DECODE(position, 0, 5,
                        DECODE(in_out, 'IN', 1,
                                       'OUT', 4,
                                       'IN/OUT', 2,
                                       0)) AS column_type,
       DECODE (data_type, 'CHAR', 1,
                          'VARCHAR2', 12,
                          'NUMBER', 3,
                          'LONG', -1,
                          'DATE', 93,
                          'RAW', -3,
                          'LONG RAW', -4,
                          'TIMESTAMP', 93, 
                          'TIMESTAMP WITH TIME ZONE', -101, 
               'TIMESTAMP WITH LOCAL TIME ZONE', -102, 
               'INTERVAL YEAR TO MONTH', -103, 
               'INTERVAL DAY TO SECOND', -104, 
               'BINARY_FLOAT', 100, 'BINAvRY_DOUBLE', 101,               1111) AS data_type,
       DECODE(data_type, 'OBJECT', type_owner || '.' || type_name,               data_type) AS type_name,
       DECODE (data_precision, NULL, data_length,
                               data_precision) AS precision,
       data_length AS length,
       data_scale AS scale,
       10 AS radix,
       1 AS nullable,
       NULL AS remarks,
       sequence,
       overload,
       default_value
 FROM all_arguments
WHERE owner LIKE 'LD_DEV_STHS' ESCAPE '/'
  AND object_name LIKE 'P_TEST2' ESCAPE '/' AND data_level = 0
  AND package_name LIKE 'PKG_FW' ESCAPE '/'
  AND (argument_name LIKE null ESCAPE '/'
       OR (argument_name IS NULL
           AND data_type IS NOT NULL))
ORDER BY procedure_schem, procedure_name, overload, sequence;

【问题解决】
      两种方法：
        一、修改 jdbc 驱动程序，在查询程序中增加暗示，指示查询SQL执行计划使用规则优化器。/*+rule*/
        
        二、修改spring 源码，在查找字典信息之后缓存起来，后续直接从缓存取。
     具体修改方法：
     org.springframework.jdbc.core.metadata.CallMetaDataContext 类matchInParameterValuesWithCallParameters 方法。org.springframework.jdbc.core.metadata.GenericCallMetaDataProvider 类 reconcileParameters 方法。

