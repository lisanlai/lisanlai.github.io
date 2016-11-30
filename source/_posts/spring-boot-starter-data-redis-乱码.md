---
title: spring-boot-starter-data-redis 乱码
date: 2016-11-30 14:17:22
categories:
- Development
tags:
- spring_boot 
- redis
---

1. RedisTemplate源码

   ```java
   public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {

   	private boolean enableTransactionSupport = false;
   	private boolean exposeConnection = false;
   	private boolean initialized = false;
   	private boolean enableDefaultSerializer = true;
   	private RedisSerializer<?> defaultSerializer;
   	private ClassLoader classLoader;
     
   	private RedisSerializer keySerializer = null;
   	private RedisSerializer valueSerializer = null;
   	private RedisSerializer hashKeySerializer = null;
   	private RedisSerializer hashValueSerializer = null;
   	private RedisSerializer<String> stringSerializer = new StringRedisSerializer();
      
     	....
       //spring-boot-starter-data-redis 默认情况下使用的是JdkSerializationRedisSerializer
       //这个地方是导致redis key乱码的根本原因
       //需要做的就是替换一下keySerializer 和 hashKeySerializer
       if (defaultSerializer == null) {
   		defaultSerializer = new JdkSerializationRedisSerializer(
   			classLoader != null ? classLoader : this.getClass().getClassLoader());
   	}
       
       if (enableDefaultSerializer) {

   			if (keySerializer == null) {
   				keySerializer = defaultSerializer;
   				defaultUsed = true;
   			}
   			if (valueSerializer == null) {
   				valueSerializer = defaultSerializer;
   				defaultUsed = true;
   			}
   			if (hashKeySerializer == null) {
   				hashKeySerializer = defaultSerializer;
   				defaultUsed = true;
   			}
   			if (hashValueSerializer == null) {
   				hashValueSerializer = defaultSerializer;
   				defaultUsed = true;
   			}
   		}
   	}
   }
   ```

2. 重新申明RedisTemplate并设置keySerializer 和 hashKeySerializer

   ```java
   @Configuration
   public class RedisConfiguration {
       @Bean
       public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
               throws UnknownHostException {
           RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
           template.setKeySerializer(new StringRedisSerializer());
           template.setHashKeySerializer(new StringRedisSerializer());
           template.setConnectionFactory(redisConnectionFactory);
           return template;
       }
   }
   ```

   ​