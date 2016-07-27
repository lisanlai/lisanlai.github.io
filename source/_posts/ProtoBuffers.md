---
title: 使用ProtoBuffers做数据通信
date: 2016-04-20 14:29:01
categories: 
- Development
tags: 
- Protobuffers
- Java
---
# Proto Buffers 简介

## Proto Buffers 是什么？

{% blockquote @官网 https://developers.google.com/protocol-buffers %}
Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.
{% endblockquote %}

<!-- more -->

## Code example：
* Define proto file

		message Person {
  			required string name = 1;
  			required int32 id = 2;
  			optional string email = 3;
		}
		
* Write to outputstream

		Person john = Person.newBuilder()
    		.setId(1234)
    		.setName("John Doe")
    		.setEmail("jdoe@example.com")
    		.build();
		output = new FileOutputStream(args[0]);
		john.writeTo(output);
		
* Parse from inputstream

		Person john;
		fstream input(argv[1],
    		ios::in | ios::binary);
		john.ParseFromIstream(&input);
		id = john.id();
		name = john.name();
		email = john.email();
	
	
	

