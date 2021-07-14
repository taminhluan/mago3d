# MAGO3D Project

## 1. Project version
* openjdk 16
* postgres 13.3
* gradle7
* spring boot 2.5

***

## 2. Setting for Eclipse
### 1. openjdk16 does not recognized
openjdk16 must be installed

* [Eclipse Marketplace Issue Link](https://marketplace.eclipse.org/content/java-16-support-eclipse-2021-03-419)
* [Eclipse plugin installation link](https://download.eclipse.org/eclipse/updates/4.19-P-builds/)

### 2. gradle jdk
gradle task  jdk wrapper gradle jdk must be specified explicitly.

![Eclipse project environment settings](./doc/setting-images/1.png)

![gradle jdk settings](./doc/setting-images/2.png)

### 3. geotools library
* geotools  repository can't be downloaded, so I put it in manually
* Library location : mago3d-admin/libs

TODO : You should test it in the manager layer function. Currently, the build works, but there is an error in the function. All layer-related functions should be tested.

***

## 3. Setting for IntelliJ

Spring Boot Run Configuration in settings Working Directory : `$MODULE_WORKING_DIR$` must be set to

![IntelliJ Preferences](./doc/setting-images/3.png)

***

## 4. Service Run Resolving errors that occurred after

### 1. BeanDefinitionOverrideException Error

#### Error message
```
org.springframework.beans.factory.support.BeanDefinitionOverrideException: Invalid bean definition with name 'localeResolver' defined in class path resource 
[gaia3d/config/ServletConfig.class]: Cannot register bean definition
The bean 'localeResolver', defined in class path resource [gaia3d/config/ServletConfig.class], could not be registered. 
A bean with that name has already been defined in class path resource [org/springframework/web/servlet/config/annotation/DelegatingWebMvcConfiguration.class] 
and overriding is disabled.Action:Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

#### Cause Analysis
DelegatingWebMvcConfiguration.class -> WebMvcConfigurationSupport.class (1181 ~ 1184 lines)
WebMvcConfigurationSupport 클래스에 이미 LocaleResolver 빈이 localeResolver으로 설정되어서 발생한 오류.
spring boot 2.5 judged to have been added.

```
// WebMvcConfigurationSupport.class
@Bean
public LocaleResolver localeResolver() {
   return new AcceptHeaderLocaleResolver();
}
```
```
// ServletConfig.class
@Bean
public LocaleResolver localeResolver() {
   return new SessionLocaleResolver();
}
```
#### Resolution
ServletConfig에서 LocaleResolver 빈 설정을 주석처리하는 것으로 해결.

### 2. IllegalStateException 에러

#### Error message
```
Unable to locate the default servlet for serving static content. 
Please set the 'defaultServletName' property explicitly.
```
#### Cause Analysis
* [stackoverflow 참고링크](https://stackoverflow.com/questions/64822250/illegalstateexception-after-upgrading-web-app-to-spring-boot-2-4)

스프링 부트 2.4 릴리즈 노트 설정에 의하면, 내장된 서블릿 컨네이너에 의해 제공되던 `DefaultServlet`이 더 이상 기본으로 등록되지 않는다.
만일, 어플리케이션에 필요하다면 server.servlet.register-default-servlet을 true로 설정하면 된다.

>As described in the Spring Boot 2.4 release notes, the DefaultServlet provided by the embedded Servlet container is no longer registered by default.
If your application needs it, as yours appears to do, you can enable it by setting server.servlet.register-default-servlet to true.

또는, `WebServerFactoryCustomizer` 빈을 프로그래밍 방식으로 설정할 수 있다.
>Alternatively, you can configure it programatically using a `WebServerFactoryCustomizer` bean:

```
@Bean
WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> enableDefaultServlet() {
    return (factory) -> factory.setRegisterDefaultServlet(true);
}
```
#### 해결 방법
`application.properties`에 `server.servlet.register-default-servlet=true` 설정을 추가하는 것으로 해결.
