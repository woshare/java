## springBoot开发中bug解决方案

### 问题一：org.springframework.web.bind.MissingServletRequestParameterException:Required String parameter 'cmd' is not present
>1，google结果基本是分析说是代码里什么地方多调用了一个HttpServletRequest，将http req参数消费了，所以等你想获取req中的数据的时候，说没有这个参数。因为流的数据放在内存中（部分或全部），read一次数据就移动一些mark_position，第二次读的时候mark_position不是从0开始的，就第二次读取为空，但是有的IO类是有reset方法，不是所有都可以reset              
>2，在代码里，查找全文HttpServletRequest，看什么地方有调用，但是结果是control，weblog，authrize 代码中有调用，但是从代码上来看，应该很正常，而且感觉google说的原因，可能不是我出现这个问题的原因，故而tcpdump+wireshark分析     
>3， tcpdump src host 10.3.xx.yy and dst port 8080 -n -s 0 -w 1300_3_manywords_uid_CET4.pcap           
>4，wireshark分析，追踪http流（如果是https，估计就难了），发现http post请求当数据量多的时候，有数据被截断的现象         
>5，我们的结构是 client--https-->nginx1--https-->nginx2--http-->tomcat-->server，怀疑nginx，tomcat，server（springboot框架）      
>6，在springboot application-test.properties配置中添加spring.http.multipart.max-file-size=1000Mb       
>                                                    spring.http.multipart.max-request-size=1000Mb 好像没有效果        
>7，修改tomcat server.xml中的配置Connector maxPostSize="-1"，重启，测试有效，备注：网上说maxPostSize="0"是没有数据大小限制，这个设置是无效的（tomcat7.0.63之前，<=0表示不限制，tomcat7.0.63包含及以后版本，<0表示不限制）            
>8，重复步骤3-7，发现，现在的post数据没有被截断（备注，对比试验，每次post数据量大小是一样的）       
>9，综上，应该是http post数据量大，超过了tomcat在内存中设定的最大值，当nginx2给tomcat的时候就被截断了，这也解释了为什么在nginx2到tomcat之前的数据是被截断了，不然就应该进一步分析是tomcat-->springboot server之间的数据是否被截断了                                                
