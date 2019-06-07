---
layout:     post
title:      kubectl常用命令使用
date:       2019-06-03
author:     cyf
header-img: img/beautiful.jpg
catalog: true
tags:
    - Java
---

# 一、报错信息
```
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2019-06-07 15:45:19.461 ERROR 1 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'weChatLoginCheckServiceImpl': Unsatisfied dependency expressed through field 'addLoginLogService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'addLoginLogService': Unsatisfied dependency expressed through field 'loginTimeHistoryESRepository'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ILoginTimeHistoryES': Cannot resolve reference to bean 'elasticsearchTemplate' while setting bean property 'elasticsearchOperations'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'elasticsearchTemplate' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchDataAutoConfiguration.class]: Unsatisfied dependency expressed through method 'elasticsearchTemplate' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:846) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:863) ~[spring-context-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:546) ~[spring-context-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140) ~[spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:775) [spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397) [spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:316) [spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1260) [spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1248) [spring-boot-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at com.allqj.authentication_java.AuthenticationJavaApplication.main(AuthenticationJavaApplication.java:22) [classes!/:0.0.1-SNAPSHOT]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_192]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_192]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_192]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_192]
	at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48) [app.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:87) [app.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:50) [app.jar:0.0.1-SNAPSHOT]
	at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51) [app.jar:0.0.1-SNAPSHOT]
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'addLoginLogService': Unsatisfied dependency expressed through field 'loginTimeHistoryESRepository'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ILoginTimeHistoryES': Cannot resolve reference to bean 'elasticsearchTemplate' while setting bean property 'elasticsearchOperations'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'elasticsearchTemplate' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchDataAutoConfiguration.class]: Unsatisfied dependency expressed through method 'elasticsearchTemplate' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:273) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1239) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1166) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:593) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 27 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ILoginTimeHistoryES': Cannot resolve reference to bean 'elasticsearchTemplate' while setting bean property 'elasticsearchOperations'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'elasticsearchTemplate' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchDataAutoConfiguration.class]: Unsatisfied dependency expressed through method 'elasticsearchTemplate' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:378) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:110) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1648) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1400) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:273) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1239) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1166) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:593) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 40 common frames omitted
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'elasticsearchTemplate' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchDataAutoConfiguration.class]: Unsatisfied dependency expressed through method 'elasticsearchTemplate' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:767) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:508) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1288) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1127) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:538) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:367) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 53 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'elasticsearchClient' defined in class path resource [org/springframework/boot/autoconfigure/data/elasticsearch/ElasticsearchAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:625) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:455) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1288) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1127) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:538) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:273) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1239) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1166) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:855) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:758) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 63 common frames omitted
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.elasticsearch.client.transport.TransportClient]: Factory method 'elasticsearchClient' threw exception; nested exception is java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:620) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 77 common frames omitted
Caused by: java.lang.IllegalStateException: availableProcessors is already set to [4], rejecting [4]
	at io.netty.util.NettyRuntime$AvailableProcessorsHolder.setAvailableProcessors(NettyRuntime.java:51) ~[netty-common-4.1.29.Final.jar!/:4.1.29.Final]
	at io.netty.util.NettyRuntime.setAvailableProcessors(NettyRuntime.java:87) ~[netty-common-4.1.29.Final.jar!/:4.1.29.Final]
	at org.elasticsearch.transport.netty4.Netty4Utils.setAvailableProcessors(Netty4Utils.java:83) ~[transport-netty4-client-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.transport.netty4.Netty4Transport.<init>(Netty4Transport.java:112) ~[transport-netty4-client-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.transport.Netty4Plugin.lambda$getTransports$0(Netty4Plugin.java:86) ~[transport-netty4-client-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.client.transport.TransportClient.buildTemplate(TransportClient.java:189) ~[elasticsearch-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.client.transport.TransportClient.<init>(TransportClient.java:283) ~[elasticsearch-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.transport.client.PreBuiltTransportClient.<init>(PreBuiltTransportClient.java:128) ~[transport-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.transport.client.PreBuiltTransportClient.<init>(PreBuiltTransportClient.java:114) ~[transport-6.4.2.jar!/:6.4.2]
	at org.elasticsearch.transport.client.PreBuiltTransportClient.<init>(PreBuiltTransportClient.java:104) ~[transport-6.4.2.jar!/:6.4.2]
	at org.springframework.data.elasticsearch.client.TransportClientFactoryBean.buildClient(TransportClientFactoryBean.java:85) ~[spring-data-elasticsearch-3.1.2.RELEASE.jar!/:3.1.2.RELEASE]
	at org.springframework.data.elasticsearch.client.TransportClientFactoryBean.afterPropertiesSet(TransportClientFactoryBean.java:80) ~[spring-data-elasticsearch-3.1.2.RELEASE.jar!/:3.1.2.RELEASE]
	at org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration.elasticsearchClient(ElasticsearchAutoConfiguration.java:59) ~[spring-boot-autoconfigure-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration$$EnhancerBySpringCGLIB$$3605591f.CGLIB$elasticsearchClient$0(<generated>) ~[spring-boot-autoconfigure-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration$$EnhancerBySpringCGLIB$$3605591f$$FastClassBySpringCGLIB$$57b93fc7.invoke(<generated>) ~[spring-boot-autoconfigure-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:244) ~[spring-core-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:363) ~[spring-context-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	at org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration$$EnhancerBySpringCGLIB$$3605591f.elasticsearchClient(<generated>) ~[spring-boot-autoconfigure-2.1.0.RELEASE.jar!/:2.1.0.RELEASE]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_192]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_192]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_192]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_192]
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154) ~[spring-beans-5.1.2.RELEASE.jar!/:5.1.2.RELEASE]
	... 78 common frames omitted
```
# 二、问题原因
springboot版本2.1.0集成redis和elasticsearch初始化冲突
```
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```
# 三、问题解决
添加如下参数即可
```
-Des.set.netty.runtime.available.processors=false
```