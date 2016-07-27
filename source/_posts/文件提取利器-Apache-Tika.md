---
title: 文件提取利器-Apache Tika
date: 2016-04-29 21:13:18
categories:
- Development
tags:
- Tika
- 文件提取
---

{% blockquote @官网 https://tika.apache.org/ %}
Description: The Apache Tika™ toolkit detects and extracts metadata and text from over a thousand different file types (such as PPT, XLS, and PDF).
功能描述：Apache Tika 可以发现和提取各种各样类型文件的元数据（meatadata）和文件内容，例如:PPT，XLS 和PDF等等。
{% endblockquote %}

## Using Tika as a Maven dependency

	<dependency>
    	<groupId>org.apache.tika</groupId>
    	<artifactId>tika-core</artifactId>
    	<version>1.13</version>
  	</dependency>

> If you want to use Tika to parse documents (instead of simply detecting document types, etc.), you'll want to depend on tika-parsers instead:

	<dependency>
    	<groupId>org.apache.tika</groupId>
    	<artifactId>tika-parsers</artifactId>
    	<version>1.13</version>
  	</dependency>

<!-- more -->
## 使用案例：利用Tika获取文件的Content-Type
		String[] files = new String[]{
                "/Users/lisanlai/Desktop/file.docx",
                "/Users/lisanlai/Desktop/file.xlsx",
                "/Users/lisanlai/Desktop/file.pptx",
                "/Users/lisanlai/Desktop/file.md",
                "/Users/lisanlai/Desktop/file.txt",
                "/Users/lisanlai/Desktop/Test.java"
        };

        for (int i = 0; i < files.length; i++) {
            TikaConfig tikaConfig = TikaConfig.getDefaultConfig();;
            Metadata metadata = new Metadata();

            MimeTypes mimeRegistry = tikaConfig.getMimeRepository();
            String filename = files[i];
            metadata.set(Metadata.RESOURCE_NAME_KEY, filename);
            System.out.println(i+ " = ["
                    + mimeRegistry.detect(null, metadata) + "]");
        }

        System.out.println("=========================================");


        for (int i = 0; i < files.length; i++) {
            TikaConfig tikaConfig = TikaConfig.getDefaultConfig();;
            Metadata metadata = new Metadata();
            String filename = files[i];
            Path path = FileSystems.getDefault().getPath(filename);
            InputStream stream = TikaInputStream.get(path);
            Detector detector = tikaConfig.getDetector();
            MediaType mediaType = detector.detect(stream, metadata);
            System.out.println(i+ " = ["
                    + detector.detect(stream, metadata) + "]" + " ==== " + mediaType.getSubtype());
        }
        
        打印结果如下：
        
        0 = [application/vnd.openxmlformats-officedocument.wordprocessingml.document]
		1 = [application/vnd.openxmlformats-officedocument.spreadsheetml.sheet]
		2 = [application/vnd.openxmlformats-officedocument.presentationml.presentation]
		3 = [text/x-web-markdown]
		4 = [text/plain]
		5 = [text/x-java-source]
		=========================================
		0 = [application/vnd.openxmlformats-officedocument.wordprocessingml.document] ==== vnd.openxmlformats-officedocument.wordprocessingml.document
		1 = [application/vnd.openxmlformats-officedocument.spreadsheetml.sheet] ==== vnd.openxmlformats-officedocument.spreadsheetml.sheet
		2 = [application/vnd.openxmlformats-officedocument.presentationml.presentation] ==== vnd.openxmlformats-officedocument.presentationml.presentation
		3 = [text/plain] ==== plain
		4 = [text/plain] ==== plain
		5 = [text/plain] ==== plain
        
                        


		

