1.Hive 테이블 생성하기
---------------------------------------------------------------------------------------------------------------------------
<pre><code>CREATE EXTERNAL TABLE `weblogs.stat_access`(
   `host` string,                                                                       
   `count` int)
 PARTITIONED BY ( ymd string )
 STORED AS ORC
 LOCATION                                                                               
   'hdfs://sandbox-hdp.hortonworks.com:8020/stage-data/weblogs/stat_access';  
</code></pre>

2.Workflow File(workflow.xml) 
----------------------------------------------------------------------------------------------------------------------------
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<workflow-app name="lecture_06" xmlns="uri:oozie:workflow:0.5" xmlns:sla="uri:oozie:sla:0.2">
   <global/>
   <start to="hive_action_1"/>
   <kill name="Kill">
      <message>Action Failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
   </kill>
   <action name="hive_action_1">
     <hive2 xmlns="uri:oozie:hive2-action:0.2">
       <job-tracker>${jobTracker}</job-tracker>
       <name-node>${nameNode}</name-node>
       <prepare/>
       <configuration>
          <property>
              <name>mapred.job.queue.name</name>
              <value>${queueName}</value>
          </property>
       </configuration>
       <jdbc-url>jdbc:hive2://localhost:10000/weblog</jdbc-url>
       <password>hive</password>
       <script>lib/stat_accesslog.hql</script>
       <param>YMD=${YMD}</param>       
     </hive2>
     <ok to="end"/>
     <error to="Kill"/>
   </action>
   <end name="end"/>
</workflow-app>  
```

3.Library File(lib/stat_accesslog.hql)
---------------------------------------------------------------------------------------------------------------------------
<pre><code>INSERT OVERWRITE TABLE weblogs.stat_access PARTITION (ymd=${YMD})
SELECT remote_host, count(remote_host) AS count FROM weblogs.access_log_orc WHERE (ymd=${YMD}) GROUP BY remote_host ORDER BY count DESC;
</code></pre>

4.Coordinator File(coordinator.xml) 
----------------------------------------------------------------------------------------------------------------------------
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<coordinator-app xmlns:sla="uri:oozie:sla:0.2" xmlns="uri:oozie:coordinator:0.4" name="stat_weblog_coordinator" frequency="10 0 * * *" start="2018-04-01T00:00+0900" end="2020-12-31T02:00+0900" timezone="Asia/Seoul">
    <controls>
        <timeout>1440</timeout>
    </controls>
   <datasets>
        <dataset name="data" frequency="1440" initial-instance="2018-04-01T00:00+0900" timezone="Asia/Seoul">
            <uri-template>hdfs://sandbox-hdp.hortonworks.com/stage-data/weblog/access/${YEAR}${MONTH}${DAY}</uri-template>
            <done-flag>_DONE</done-flag>
        </dataset>
   <datasets>
      
   
      
</coordinator-app>
       
```   

5.Coordinator job properties File(coord-job.properties) 
----------------------------------------------------------------------------------------------------------------------------
<pre><code>user.name=mapred
oozie.use.system.libpath=true
oozie.coord.application.path=${nameNode}/user/oozie/workflow/practice_06/coordinator.xml
queueName=default
nameNode=hdfs://sandbox-hdp.hortonworks.com:8020
oozie.libpath=
jobTracker=sandbox-hdp.hortonworks.com\:8032

</code></pre>

X.Job Propreties File(workflow-job.properties)
--
<pre><code>
user.name=mapred
oozie.use.system.libpath=true
oozie.wf.application.path=${nameNode}/user/oozie/workflow/practice_06
queueName=default
nameNode=hdfs://sandbox-hdp.hortonworks.com:8020
oozie.libpath=/user/oozie/share/lib/lib_20180201102929/sqoop
jobTracker=sandbox-hdp.hortonworks.com\:8032
YMD=20180401
</code></pre>
