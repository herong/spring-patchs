������������
    ʹ��spring SimpleJdbcCall �ڵ��ô洢����ʱ������

�����������
     SimpleJdbcCall �ڵ��ô洢���̻���ʱ�������jdbc����������������������ȡ�˶���������Ϣ�������������͵�)��
          getProcedures
          getProcedureColumns
     ������������ѯoracle 10g ����ʱ�ر�������ͬ�����汾���Թ��ˡ���9i,11g�Ⱥõ㣬�����Ҫ����ߵĻ����ǿ����Ż���
    
������׷�١�

      ʹ��jdbc debug�汾��ͨ������_g��β����oracle :ojdbc5_g.jar


������־������
.level=FINER
oracle.jdbc.level=FINER
oracle.jdbc.handlers=java.util.logging.FileHandler
java.util.logging.FileHandler.level=FINER
java.util.logging.FileHandler.pattern=E:/jdbc.log
java.util.logging.FileHandler.count=1
java.util.logging.FileHandler.formatter=java.util.logging.SimpleFormatter


ָ��jvm������
-Djava.util.logging.config.file=E:/OracleLog.properties -Doracle.jdbc.Trace=true


�����־��
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

����������
      ���ַ�����
        һ���޸� jdbc ���������ڲ�ѯ���������Ӱ�ʾ��ָʾ��ѯSQLִ�мƻ�ʹ�ù����Ż�����/*+rule*/
        
        �����޸�spring Դ�룬�ڲ����ֵ���Ϣ֮�󻺴�����������ֱ�Ӵӻ���ȡ��
     �����޸ķ�����
     org.springframework.jdbc.core.metadata.CallMetaDataContext ��matchInParameterValuesWithCallParameters ������org.springframework.jdbc.core.metadata.GenericCallMetaDataProvider �� reconcileParameters ������

