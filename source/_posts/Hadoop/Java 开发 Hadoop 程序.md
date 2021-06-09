---
type: blog
title: Java 开发 Hadoop 程序
date: 2021-05-25
categories: 技术
tags: Java
cover: false
meta:
  date: false
---



<!-- more -->

```java
@RequestMapping(value = "/updatenmtest", method = RequestMethod.GET)
    public String updateNmTest() throws Exception {
        YarnConfiguration conf = new YarnConfiguration();
        YarnClient yarnClient = YarnClient.createYarnClient();
        String yarnSitePath = "/disk1/eadop/hadoop-2.8.5/etc/hadoop/yarn-site.xml";
        String coreSitePath = "/disk1/eadop/hadoop-2.8.5/etc/hadoop/core-site.xml";
        conf.addResource(new Path(yarnSitePath));
        conf.addResource(new Path(coreSitePath));

        // kerberos
        System.setProperty("java.security.krb5.conf", "/etc/krb5.conf" );
        conf.setBoolean("hadoop.security.authorization", true);
        conf.set("hadoop.security.authentication", "kerberos");

        ArrayList<String> nodeIdList = new ArrayList<>();

        UserGroupInformation.setConfiguration(conf);
        UserGroupInformation ugi = UserGroupInformation.loginUserFromKeytabAndReturnUGI("yarn/hd047.corp.yodao.com@YOUDAO.163.COM", "/disk1/eadhadoop/yarn.keytab");
        ugi.doAs(new PrivilegedAction<Object>() {

             @SneakyThrows
             @Override
             public Object run() {
                 RMAdminCLI rmAdminCLI = new RMAdminCLI(conf);
                 String cmd = "-updateNodeResource aliyun038.analysis.aliyun.corp.yodao.com:45454 512000 64";
                 String[] cmdList = cmd.split(" ");
                 rmAdminCLI.run(cmdList);

                 //
                 YarnClient yarnClient = YarnClient.createYarnClient();
                 yarnClient.init(conf);
                 yarnClient.start();
                 List<NodeReport> nodeReports = yarnClient.getNodeReports();

                 nodeReports.forEach(eachNode -> {
                     String s = eachNode.getNodeId().toString();
                     nodeIdList.add(s);
                 });
                 // ((NodeReportPBImpl) nodeReports.get(0)).getNodeId().toString()
                 yarnClient.stop();
                 return null;
             }
         });

        return nodeIdList.toString();
    }
```





参考：

> [Java访问kerberos认证的HDFS文件](https://blog.csdn.net/IBoyMan/article/details/103989570)
>
> [使用hdfsclient遇到的kerberos解决方法](https://blog.csdn.net/qq_35347459/article/details/73647225)