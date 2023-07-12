# Synjq常见问题 </br>
>常见问题解决方法，仅供参考，具体报错请根据实际情况分析！
<table>
<tr>
<th>PROBLEM</th>
<th>SOLVE</th>
</tr>
<tr>
<td>dm.jdbc.driver.DMEXception: ERROR in line: 1 flashback function is disable(达梦闪回功能禁用)</td>
<td>select name,type,value from v$parameter where name='ENABLE_FLASHBACK';</br>alter system set 'ENABLE_FLASHBACK'=1 both; ##打开 </td>
</tr>
<tr>
<td>caused by：ora.apache.kafka.commect.error.ConnectException:An exception occurred in the change event producer. this connector will be stopped(并发开的太高)</td>
<td>减少并发数</td>
</tr>
<tr>
<td>
ERROR org.tikv.common.PDClient [] - Could not get leader member with: [http://9.16.98.81:2379](中间机--tidb源端端口不通)</td>
<td>开启中间机到tidb所有pd的2379以及所有tikv 20161等等端口</td>
</tr>
<tr>
<td>ERROR io.debezium.pipeline.ErrorHandler [] - Producer failure oracle.streams.StreamsException: ORA-26701: Streams process DEBUG0 does not exist(oracle配置debug错误)</td>
<td>web配置只填写debug，开几个并发在后端创建几个debug0 1 2排序</td>
</tr>
<tr>
<td>oragent方式增量不通(大多数情况是因为8303端口没有映射)</td>
<td>请联系九桥工程师调试</td>
</tr>
<tr>
<td>oragent一直重启(fzs 的 tablelist 是个数组，大小是固定的 255 个，table 数量过多就会越界)具体情况请按照实际报错分析解决</td>
<td>修改export.cong文件，改为用户级同步，重启</td>
</tr>
</table>