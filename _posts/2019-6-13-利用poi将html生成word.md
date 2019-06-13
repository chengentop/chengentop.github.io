---
layout:     post
title:      html生成word
subtitle:   html生成word
date:       2019-06-13
author:     Chen
catalog:      true
tags:
    - poi
    - html生成word
---
下载poi的jar包 此处用的3.15

核心代码
```
 try {
       //word内容
       String content = "<html><head><meta charset='UTF-8'/></head> <body>" + article.getContent() + "</body></html>";

       //这里是必须要设置编码的，不然导出中文就会乱码。
       byte b[] = content.getBytes("utf-8");
       //将字节数组包装到流中
       ByteArrayInputStream bais = new ByteArrayInputStream(b);
       /*
        * 关键地方
        * 生成word格式 
        */
       POIFSFileSystem poifs = new POIFSFileSystem();
       DirectoryEntry directory = poifs.getRoot();
       DocumentEntry documentEntry = directory.createDocument("WordDocument", bais);
       //输出文件
       request.setCharacterEncoding("utf-8");
       //导出word格式
       response.setContentType("application/msword");
       response.addHeader("Content-Disposition", "attachment;filename=" + new String(("name" + ".doc").getBytes(), "iso-8859-1"));
       OutputStream ostream = response.getOutputStream();
       poifs.writeFilesystem(ostream);
       bais.close();
       ostream.close();
     } catch (Exception e) {
         collect(e, model);
     }
```
