
Chapter 4.5. AWS 기본 설정 및 클라우드 서비스 환경 구축,
AWS 활용 스프링부트 프로젝트 배포 -> 4


**AWS를 이해하고 어떻게 클라우드 서비스 환경을 구축하는지 실습을 통해 학습합니다.**

*Chapter 4-4. AWS S3 설정*



```
/* 변경한 파일 리스트 */
+36-0Areview/src/main/java/com/chicken/review/ReviewApplication.java
+22-0Areview/src/main/java/com/chicken/review/config/ErrorConfiguration.java
+44-0Areview/src/main/java/com/chicken/review/config/StaticResourceConfiguration.java
+30-0Areview/src/main/java/com/chicken/review/config/TilesConfig.java
+78-0Areview/src/main/java/com/chicken/review/config/WebSecurityConfig.java
+64-0Areview/src/main/java/com/chicken/review/login/controller/UserController.java
+15-0Areview/src/main/java/com/chicken/review/login/mapper/ReviewMapper.java
+25-0Areview/src/main/java/com/chicken/review/login/service/ReviewService.java
+36-0Areview/src/main/java/com/chicken/review/login/vo/ReviewVO.java
+45-0Areview/src/main/resources/application.properties
+2-0Areview/src/main/resources/log4jdbc.log4j2.properties
+48-0Areview/src/main/resources/logback-spring.xml
+17-0Areview/src/main/resources/mapper/review.xml
+0-0Areview/src/main/resources/static/img/img1.jpg
+0-0Areview/src/main/resources/static/img/img2.jpg
+0-0Areview/src/main/resources/static/img/img3.jpg
+30-0Areview/src/main/webapp/WEB-INF/layout/footer.jsp
+37-0Areview/src/main/webapp/WEB-INF/layout/header.jsp
+3-0Areview/src/main/webapp/WEB-INF/layout/modal.jsp
+290-0Areview/src/main/webapp/WEB-INF/login/login.jsp
+36-0Areview/src/main/webapp/WEB-INF/template/base.jsp
+25-0Areview/src/main/webapp/WEB-INF/template/errorbase.jsp
+21-0Areview/src/main/webapp/WEB-INF/template/nobase.jsp
+35-0Areview/src/main/webapp/WEB-INF/tiles/tiles.xml
+16-0Areview/src/test/java/com/chicken/review/ReviewApplicationTests.java
```

스프링 부트를 웹서버에 올리기 위한 세팅
File review/src/main/java/com/chicken/review/ReviewApplication.java ADDED
```
+package com.chicken.review;
+
+import javax.servlet.Filter;
+
+import org.springframework.boot.SpringApplication;
+import org.springframework.boot.autoconfigure.SpringBootApplication;
+import org.springframework.boot.builder.SpringApplicationBuilder;
+import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
+import org.springframework.context.annotation.Bean;
+import org.springframework.http.converter.HttpMessageConverter;
+import org.springframework.web.filter.CharacterEncodingFilter;
+
+@SpringBootApplication
+public class ReviewApplication extends SpringBootServletInitializer{
+
+	
+	public static void main(String[] args) {
+		SpringApplication.run(ReviewApplication.class, args);
+	}
+	
+	
+    @Override
+    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
+        return builder.sources(ReviewApplication.class);
+    }
+    
+
+    
+    @Bean
+    public Filter characterEncodingFilter() {
+        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
+        characterEncodingFilter.setEncoding("UTF-8");
+        characterEncodingFilter.setForceEncoding(true);
+        return characterEncodingFilter;
+    }
+}
```

에러 났을때 처리 하는 경로
File review/src/main/java/com/chicken/review/config/ErrorConfiguration.java ADDED
```
+package com.chicken.review.config;
+
+import org.springframework.http.HttpStatus;
+import org.springframework.boot.web.server.ErrorPage;
+import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
+import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
+import org.springframework.context.annotation.Bean;
+import org.springframework.context.annotation.Configuration;
+
+@Configuration
+public class ErrorConfiguration {
+	
+	@Bean
+	public ConfigurableServletWebServerFactory webServerFactory() {
+		
+		TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
+        factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/error/404"));
+    
+        return factory;
+	}
+	
+}
File review/src/main/java/com/chicken/review/config/StaticResourceConfiguration.java ADDED
 Side-by-side diff View file Comment More
+package com.chicken.review.config;
+
+
+import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.beans.factory.annotation.Qualifier;
+import org.springframework.context.annotation.Bean;
+import org.springframework.context.annotation.Configuration;
+import org.springframework.web.servlet.HandlerInterceptor;
+import org.springframework.web.servlet.ViewResolver;
+import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
+import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
+import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
+
+import org.springframework.web.servlet.resource.PathResourceResolver;
+import org.springframework.web.servlet.view.InternalResourceViewResolver;
+import org.springframework.web.servlet.view.JstlView;
+
+@Configuration
+public class StaticResourceConfiguration  implements WebMvcConfigurer {
+
+  private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
+	        "classpath:/META-INF/resources/", 
+	        "classpath:/resources/",
+	        "classpath:/resources/static/",
+	        "classpath:/static/", 
+	        "classpath:/public/" };
+
+
+    
+    @Override
+    public void addResourceHandlers(ResourceHandlerRegistry registry) {
+    	if (!registry.hasMappingForPattern("/**")) {
+
+	        registry.addResourceHandler("/**")
+	                .addResourceLocations("CLASSPATH_RESOURCE_LOCATIONS")
+	                .setCachePeriod(3600)
+	                .resourceChain(true)
+	                .addResolver(new PathResourceResolver());
+
+    	}
+      
+    }
+  
+}
File review/src/main/java/com/chicken/review/config/TilesConfig.java ADDED
 Side-by-side diff View file Comment More
+package com.chicken.review.config;
+
+import org.springframework.context.annotation.Bean;
+import org.springframework.context.annotation.Configuration;
+import org.springframework.web.servlet.view.UrlBasedViewResolver;
+import org.springframework.web.servlet.view.tiles3.TilesConfigurer;
+import org.springframework.web.servlet.view.tiles3.TilesView;
+
+
+@Configuration
+public class TilesConfig {
+	
+	@Bean
+	public TilesConfigurer tilesConfigurer(){
+		final TilesConfigurer configurer = new TilesConfigurer();
+		configurer.setDefinitions(new String[] {"/WEB-INF/tiles/tiles.xml"});
+		configurer.setCheckRefresh(true);
+		
+		return configurer;
+	}
+	
+	@Bean
+	public UrlBasedViewResolver tilesViewResolver(){
+		UrlBasedViewResolver resolver = new UrlBasedViewResolver();
+		resolver.setOrder(0);
+		resolver.setViewClass(TilesView.class);
+		return resolver;
+	}
+	
+}

```

스프링 부트에서 웹 관련 보안 설정을 하는 곳 (이미지와 css 파일 같은 것들은 접근 가능하도록 열어준다)
로그인 로그아웃 같은 패스들을 열어준다.
File review/src/main/java/com/chicken/review/config/WebSecurityConfig.java ADDED
 ```
+package com.chicken.review.config;
+
+import java.io.IOException;
+import java.util.Properties;
+
+import javax.servlet.ServletException;
+
+import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.beans.factory.annotation.Value;
+import org.springframework.context.annotation.Bean;
+import org.springframework.context.annotation.Configuration;
+import org.springframework.core.env.Environment;
+import org.springframework.security.authentication.AuthenticationManager;
+import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
+import org.springframework.security.config.annotation.web.builders.HttpSecurity;
+import org.springframework.security.config.annotation.web.builders.WebSecurity;
+import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
+import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
+import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
+
+
+
+
+@Configuration
+@EnableWebSecurity
+public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
+ 
+	String[] staticResources  =  {
+            "/css/**",
+            "/js/**",
+            "/img/**",
+            "/fonts/**",
+            "/resources/**"
+        };
+ 
+	
+	@Autowired
+	private Environment env;
+
+    
+	
+    @Override
+    protected void configure(HttpSecurity http) throws Exception {
+       
+    	http
+    	.csrf().disable()
+		.headers()
+			.and()
+		.authorizeRequests()
+		 	.antMatchers(staticResources).permitAll()
+		 	.anyRequest().authenticated()
+			.and()
+
+		.exceptionHandling()
+			.and()
+		.formLogin() 
+			.loginPage("/login")
+			.defaultSuccessUrl("/")
+			.permitAll()
+			.and()
+		.logout()
+			.logoutUrl("/logout")
+        	.permitAll();
+    }
+
+	 
+
+
+
+	@Override
+    public void configure(WebSecurity web) throws Exception {
+      web
+        .ignoring()
+           .antMatchers(staticResources); 
+    }
+
+   
+}
```

로그인 처리 부분
File review/src/main/java/com/chicken/review/login/controller/UserController.java ADDED
```
+package com.chicken.review.login.controller;
+
+import java.util.Map;
+import java.util.Optional;
+
+import javax.servlet.http.HttpServletRequest;
+import javax.servlet.http.HttpServletResponse;
+
+import org.slf4j.Logger;
+import org.slf4j.LoggerFactory;
+import org.springframework.beans.factory.annotation.Autowired;
+//import org.springframework.security.authentication.AuthenticationManager;
+import org.springframework.security.core.Authentication;
+import org.springframework.security.core.context.SecurityContextHolder;
+import org.springframework.stereotype.Controller;
+import org.springframework.ui.Model;
+import org.springframework.web.bind.annotation.RequestMapping;
+import org.springframework.web.context.request.RequestContextHolder;
+import org.springframework.web.context.request.ServletRequestAttributes;
+
+import com.chicken.review.login.service.ReviewService;
+
+
+
+
+
+
+@Controller
+public class UserController {
+	
+	//private final Logger logger = LoggerFactory.getLogger(UserController.class);
+	
+	
+	@Autowired
+    private ReviewService reviewService;
+	
+
+    @RequestMapping(value="/login")
+    public String login(Model model, String error, String logout) {
+    	
+    	ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
+    	HttpServletRequest request = attr.getRequest();
+    	HttpServletResponse response = attr.getResponse();
+
+    	Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
+    	
+    	
+    	model.addAttribute("reviewList", reviewService.getReviewList());
+    	
+    	System.out.println(reviewService.getReviewList());
+    	if(authentication != null) {
+    		
+	    	if(!authentication.getPrincipal().equals("anonymousUser"))
+	    		return "redirect:/";
+    	}
+    	
+    	
+
+        return "login/login";
+    }
+
+    
+   
+}
```

리뷰를 작성하는 리뷰 관련 매퍼 
File review/src/main/java/com/chicken/review/login/mapper/ReviewMapper.java ADDED
```
+package com.chicken.review.login.mapper;
+
+import java.util.List;
+
+import org.apache.ibatis.annotations.Mapper;
+
+import com.chicken.review.login.vo.ReviewVO;
+
+
+@Mapper
+public interface ReviewMapper {
+
+	List<ReviewVO> getReviewList();
+	
+}
File review/src/main/java/com/chicken/review/login/service/ReviewService.java ADDED
 Side-by-side diff View file Comment More
+package com.chicken.review.login.service;
+
+import java.util.List;
+
+import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.stereotype.Service;
+
+import com.chicken.review.login.mapper.ReviewMapper;
+import com.chicken.review.login.vo.ReviewVO;
+
+@Service
+public class ReviewService {
+	
+	
+	@Autowired
+	private ReviewMapper reviewMapper;
+	
+
+	public List<ReviewVO> getReviewList() {
+		
+		return reviewMapper.getReviewList();
+	}
+	
+	
+}

```

리뷰 관련 VO (Value Object)
File review/src/main/java/com/chicken/review/login/vo/ReviewVO.java ADDED
```
+package com.chicken.review.login.vo;
+import lombok.Data;
+
+@Data
+public class ReviewVO {
+
+	int seq;
+	String title;
+	String content;
+	String s3ImageUrl;
+	public int getSeq() {
+		return seq;
+	}
+	public void setSeq(int seq) {
+		this.seq = seq;
+	}
+	public String getTitle() {
+		return title;
+	}
+	public void setTitle(String title) {
+		this.title = title;
+	}
+	public String getContent() {
+		return content;
+	}
+	public void setContent(String content) {
+		this.content = content;
+	}
+	public String getS3ImageUrl() {
+		return s3ImageUrl;
+	}
+	public void setS3ImageUrl(String s3ImageUrl) {
+		this.s3ImageUrl = s3ImageUrl;
+	}
+	
+}
```

File review/src/main/resources/application.properties ADDED
```
+#SERVER PORT
+server.port=7000                #포트 세팅
+server.servlet-path=/*          #서블릿 패스 설정
+
+cloud.aws.region.static=ap-northeast-2     #AWS 의 리젼을 세팅하고 입력
+
+
+#VIEW 
+spring.mvc.view.prefix=/WEB-INF/       #jsp 파일의 위치를 설정
+spring.mvc.view.suffix=.jsp            #웹 상에서 파일 path를 축약한다 실제로 login.jsp 의 경우 login 으로 줄여주는 역할
+
+spring.datasource.driverClassName=com.mysql.jdbc.Driver  #드라이버 세팅 mysql 아니라 마리아 DB 혹은 다른 디비로 연결시에 이 부분을 설정
+spring.datasource.url=jdbc:mysql://review.cm03m1qy5nzi.ap-northeast-2.rds.amazonaws.com:3306/innodb #RDS endpoint 경로
+spring.datasource.username=admin       #DB admin 명 
+spring.datasource.password=abcd1234    #DB admin 패스워드
+
+cloud.aws.stack.auto=false
+
+mybatis.mapper-locations=/mapper/*.xml #mybatis 의 매퍼의 쿼리가 들어가 있는 xml 경로를 일괄 세팅해준다.
+
+
+#IMG SETTING  #이미지 세팅
+server.compression.enabled=true    
+spring.resources.chain.cache=true          
+server.compression.min-response-size=2048
+spring.resources.chain.enabled=true
+spring.resources.cache.period=3600
+spring.resources.add-mappings=true  
+
+#ENCODING UTF8 #스프링의 인코딩 세팅 부분
+spring.http.encoding.charset=UTF-8
+spring.http.encoding.enabled=true
+spring.http.encoding.force=true
+
+spring.messages.basename=validation 
+
+spring.main.allow-bean-definition-overriding=true
+
+mybatis.configuration.map-underscore-to-camel-case=true
+
+spring.mvc.contentnegotiation.favor-parameter=true
+spring.mvc.contentnegotiation.favor-path-extension=true
+spring.mvc.contentnegotiation.media-types.xls=application/vnd.ms-excel
+
+spring.aop.proxy-target-class=true
```


File review/src/main/resources/log4jdbc.log4j2.properties ADDED
```
+log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
+log4jdbc.dump.sql.maxlinelength=0
```

웹서버 로그 관련 제어부
File review/src/main/resources/logback-spring.xml ADDED
```
+<?xml version="1.0" encoding="UTF-8"?>
+<configuration>
+    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
+    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
+        <encoder>
+            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
+        </encoder>
+    </appender>
+    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
+        <file>${catalina.base}/logs/logs.log</file>
+        <!-- 파일이 하루에 한개씩 생성된다 -->
+        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
+	          	<fileNamePattern>${catalina.base}/logs/log-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
+	            <maxHistory>90</maxHistory>
+	            <timeBasedFileNamingAndTriggeringPolicy                  class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
+	                <!-- or whenever the file size reaches 100MB -->
+	                <maxFileSize>100MB</maxFileSize>
+	            </timeBasedFileNamingAndTriggeringPolicy>
+	    </rollingPolicy>
+	   
+        <encoder>
+            <!-- <pattern>%-5relative %-5level %logger{35} - %msg%n</pattern> -->
+        	<pattern>${CONSOLE_LOG_PATTERN}</pattern>
+        </encoder>
+    </appender>
+    
+
+
+
+
+
+    <logger name="jdbc" level="OFF"/>
+
+    <logger name="jdbc.sqlonly" level="OFF"/>
+    <logger name="jdbc.sqltiming" level="INFO, ERROR"/>
+    <logger name="jdbc.audit" level="OFF"/>
+    <logger name="jdbc.resultset" level="OFF"/>
+    <logger name="jdbc.resultsettable" level="ERROR"/>
+    <logger name="jdbc.connection" level="OFF"/>
+    
+    <logger name="com.bitsm" level="INFO, ERROR"/>
+
+    <root level="INFO, ERROR">
+        <appender-ref ref="STDOUT" />
+        <appender-ref ref="FILE" />
+    </root>
+
+</configuration>
```


리뷰관련 쿼리 (리뷰의 내용 전체를 설정)
File review/src/main/resources/mapper/review.xml ADDED
```
+<?xml version="1.0" encoding="UTF-8"?>
+<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
+<mapper namespace="com.chicken.review.login.mapper.ReviewMapper">
+
+	
+
+	<select id="getReviewList"  parameterType="String"  resultType="com.chicken.review.login.vo.ReviewVO">
+		
+		select 
+			*
+		from
+			review
+			
+	</select>
+	
+	
+</mapper>
```

이미지 추가
File review/src/main/resources/static/img/img1.jpg ADDED
File review/src/main/resources/static/img/img2.jpg ADDED
File review/src/main/resources/static/img/img3.jpg ADDED

하단 푸터 부분 설정
File review/src/main/webapp/WEB-INF/layout/footer.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
+<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
+
+
+<style>
+	
+
+	.footer{
+		height: 100px;
+		width: 100%;
+		background: #C02425;  /* fallback for old browsers */
+		background: -webkit-linear-gradient(to right, #F0CB35, #C02425);  /* Chrome 10-25, Safari 5.1-6 */
+		background: linear-gradient(to right, #F0CB35, #C02425); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
+		
+	}
+	
+	.footerWrapper{
+		width: 1024px;
+		margin:30px auto;
+		padding-top: 40px;
+		color: #FFF;
+	}
+</style>
+
+<div class="footer">
+	<div class="footerWrapper">
+		 강사 : 배진호 , Copyright. All Rights Reserved.
+	</div>
+</div>
```

상단 헤더 부분 설정
File review/src/main/webapp/WEB-INF/layout/header.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
+<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
+<html>
+<style>
+	html, body{
+		padding:0;
+		margin:0;
+		height:100%;
+	}
+
+	.header{
+		height: 50px;
+		width: 100%;
+		background: #C02425;  /* fallback for old browsers */
+		background: -webkit-linear-gradient(to right, #F0CB35, #C02425);  /* Chrome 10-25, Safari 5.1-6 */
+		background: linear-gradient(to right, #F0CB35, #C02425); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
+		
+	}
+	
+	.headerWrapper{
+		width: 1024px;
+		margin:2px auto;
+	}
+	.logo{
+		padding-top:15px;
+		font-weight:bold;
+    	color: #010101;
+	}
+</style>
+<div class="header">
+	<div class="headerWrapper">
+	<div class="logo">OffREV</div>
+	</div>
+</div>
+
+</html>
```

모달 팝업 세팅 부분
File review/src/main/webapp/WEB-INF/layout/modal.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
+<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
File review/src/main/webapp/WEB-INF/login/login.jsp ADDED
 Side-by-side diff View file Comment More
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
+<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
+
+<script src="//developers.kakao.com/sdk/js/kakao.min.js"></script>
+
+<style>
+	
+	
+	.loginWrapper{
+		margin-top: 50px;
+		margin-left:auto;
+		margin-right:auto;
+		max-width: 400px;
+		height: 250px;
+		border-radius:5px;
+		text-align: center;
+		line-height:1.8;
+	}
+	
+	#kakao-login-btn{
+		margin-top:20px;
+    	border-top: 1px solid #DDDDDD99;
+	}
+	.itemListWrapper{
+		width: 960px;
+		height: 480px;
+		margin: 5px auto;
+		text-align: center;
+	}
+	.itemList{
+		float:left;
+		width: 306px;
+		height: 400px;
+		margin: 5px;
+		border-radius: 5px;
+		border : 2px solid #DDDDDD;
+	}
+	.imageArea{
+		width: 100%;
+		height: 200px;
+		background: #EEEEEE;
+		overflow:hidden;
+	}
+	
+	.imageArea > img{
+		width: 100%;
+		height: 100%;
+	}
+	.reviewArea{
+		width: 100%;
+		height: 180px;
+		text-align: left;
+		padding : 4px;
+	}
+	.reviewTitle{
+		width: 100%;
+		height: 20px;
+		text-align: left;
+		padding : 4px;
+	}
+	
+	.reviewArea > textarea {
+		resize: none;
+		width: 98%;
+		height: 168px;
+		background:#AAAA0011;
+		overflow:hidden;
+    	border: 0px !important;
+	}
+	.slide-child{
+		transform: translateY(50px);
+        opacity: 0;
+        transition: all 1s;
+    }
+    .is-visible{
+		transform: translateY(0px);
+        opacity: 1;
+    }
+    
+   
+</style>
+
+<script type="text/javascript">
+
+$(document).ready(function(){
+	Kakao.init('b30526376249afa40b9d4f5c977a841f');
+    // 카카오 로그인 버튼을 생성합니다.
+    Kakao.Auth.createLoginButton({
+      container: '#kakao-login-btn',
+      success: function(authObj) {
+        alert(JSON.stringify(authObj));
+      },
+      fail: function(err) {
+         alert(JSON.stringify(err));
+      }
+    });
+    
+    var slideAelements = $('.slide-child')
+    
+    
+    function animateSlideA() {
+      slideAelements.each(function (i) {
+          setTimeout(function () {
+              slideAelements.eq(i).addClass('is-visible');
+          }, 300 * (i + 1));
+      });
+    }
+    animateSlideA() ;
+    
+});
+</script>
+
+<html>
+<title>오프라인 리뷰 웹테스트</title>
+
+
+
+<body>
+
+<div class="loginWrapper">
+	<div>
+		<div class="slide-child">OffREV 는 Offline</div>
+		<div class="slide-child">Review Flatform 의 약자로써</div>
+		<div class="slide-child">오프라인 후기 정보들을</div>
+		<div class="slide-child">모아모아 제공합니다.</div>
+	</div>
+	<div id="kakao-login-btn">
+		
+	</div>
+	
+	
+</div>
+
+<div class="itemListWrapper">
+<div class="itemList slide-child">
+   <div class="imageArea"><img src="<c:out value="${ reviewList[0].s3ImageUrl }" />"/></div>
+   <div class="reviewArea">
+   	  <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
+      <textarea readonly><c:out value="${ reviewList[0].content }" /></textarea>
+   </div>
+</div>
+<div class="itemList slide-child">
+   <div class="imageArea"><img src="img/img2.jpg"/></div>
+   <div class="reviewArea">
+  	  <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
+      <textarea readonly>일시 : 2019년 9월 28일
+장소 : 공덕 IOT COC (1차), 바른치킨(2차)
+시간 : 3시, 6시
+인원 1차 60명 2차 50명
+
+이번 모임은 공덕에서 진행되었습니다.
+장소를 빌려주신 IOT COC 담당분들께는 감사말씀 드립니다.
+
+이번 모임은 1차에서는 주로 세미나 위주
+2차에서는 치킨을 위주로 진행되었습니다.
+
+폰이 말썽이라 배터리가 다 닳아버린 이후로는
+개인적으로는 고통의 시간들이었는데요.
+
+연락들도 많이오고 하는 과정에서 차가막히고 늦게되는 바람에...
+이기정 군이 고생을 많이해주었고요~!
+
+그리고 다른 분들도 도와주셔서 그래도 모임을 잘 진행하게되었네요.
+첫번째는 이모티콘 강의
+다들 이 강의를 매우 흥미로워했는데요.
+2차 모임에서 어떤 그룹은 이모티콘을 만들어 돈벌자며
+결의를 다졌다는 후문이 있습니다.
+
+그리고 두번째 강의는 크리에이터 멘땅에 해딩하기~! 이 시간은 어렵지 않아서 많은 분들이 편안! 해 했던 세션이었는데요~
+다양한 시행착오를 거쳐서 틱톡 크리에이터가 되기까지~! 더 유명해지면 얼굴보기 어려운 분이 되실 수도 있겠네요~!
+
+그리고 세번째 강의 세션은 스타트업을 위한 개발 경험 공유, 해왔던 일들을 가지고 이런저런 개발에 필요한 것이 무엇인지 돌아볼 수 있는 시간이었네요.
+
+그리고 4번째강의는 왓챠플래이 신입 재직중인 기정군이 다양한 툴과 정보들을 잘 정리해서 올려주었어요~
+
+5번째 스칼라 강의가 아쉬웠는데...
+시간배분의 문제로 간단한 설명까지밖에 못했네요.
+
+다음 강의 때 해주시기로 하셨어요~! 다음 10월 모임엔 Handson 세션으로 꼭 해야할듯싶네요~!
+
+특별히 간식과 음료를 잘 준비해주셔서~!
+다들 맛있게 잘 먹었다고 합니다..
+
+2차 모임은 48명이 소소하니 모여~!
+치킨과 대화를 나누며 시간들을 보내었습니다.
+몇분이 못오셨는데 기부..
+하신건 맛있게 잘 먹었습니다.
+
+그리고 9시반 이후에 카페로 이동해서
+2차모임의 카페인증까지~!
+
+이번 모임도 다양한 분들과 함께
+유익하고 재미있는 시간을 보내었고요~!
+
+10월 모임을 기대해 봅니다.
+이번 포스터 제작자분도 모임에 와주시고~!
+다양한 분들이 소통했던 9월 모임이었습니다.
+
+이상 감사합니다.</textarea>
+
+   </div>
+</div>
+<div class="itemList slide-child">
+   <div class="imageArea"><img src="img/img3.jpg"/></div>
+   <div class="reviewArea" >
+      <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
+      <textarea readonly>[2019년 9월 19일 7시 곱창팟 치킨모임 후기]
+
+장소 : 홍대 가오곱창
+일시 : 9월 19일 7시
+인원 : 13명
+회비 : 25000 => 24000
+
+후기
+이번 모임은 특별히
+치킨이 아니라 곱창 모임이었다는 점.
+그리고 주말 모임이 아니라
+평일 모임이었다는 점이 달랐습니다~!
+
+제가 도착하는 시점이미
+라면이 세팅되어있었습니다.
+
+그러니까 갑자기 좀 찌개를 먹어야할 것 같은 그런
+분위기!...
+
+그리고 수저는 어디갔는지 안보이는 상황
+솔직히 자리는 좀 좁고... 이 가오곱창집
+가운데 마치 무인도처럼 가운데 저희 모임이 떠있는듯한 형국!
+
+기존의 치킨모임들의 경우 어색함의 시간이
+그리 길지 않았던 반면
+이번 모임은 살짝 어색함이 풀어지는 시간에
+딜레이가 있긴했습니다.
+
+가게 한가운데 저희가 있었기 때문에....
+
+어찌 되었든
+이 가게의 전략을 처음 파악했습니다.
+바로 그것은...
+라면과 주먹밥
+평소 같았다면 빠르게 먹었겠지만
+이 가게의 전략적 승부에 모임분들은 대응하지 않았습니다.
+
+과감하게 본론으로 들어가길 원했는데요~!
+문제는
+생각보다 할게 많다는점...
+고기를 굽거나 곱창을 굽기위한 노동력
+문제로 대화는 버벅버벅
+
+그래도~! 첫인원은 5명
+그리고 7명 10명
+11명 12명 13명 식으로 늘어나는 사람들~!
+
+덕분에 인사를 3번정도 했는데요.
+평일이라 급작스레 못오시는 분도 계셨습니다.
+
+적당한 소개는 10~11명 정도 모인 시점에 한번 진행되었습니다~!
+
+오신분들은 2~3년차 웹개발자~! 개발자를 구하시는 블록체인 대표님 그리고 십여년차 자바 개발자님을 비롯해, 7년간 에이젼시를 운영하신 분, 영화관련 일을 하시다가~! 스타트업에 뛰어드신분! 그리고 작은 사업을 하시다가 개발을 배우신분~! 그리고 스타트업을 기획하시는 분, 그리고 뮤지컬을 보는 일과 다양한 일들을 하시는 디자이너님, 늘 다른분의 이야기를 청종하시는 기획자님, 그리고 디자이너님과 친구분이 추가로 오셨는데요~!
+
+이 정도 인원이 되니 그래도 살짝씩은 대화를 섞어볼수 있어서 좋았습니다.
+
+물론 모임이 이번 모임 특성이 곱창을 뽀개는 것이었기 때문에 리필하는 것에 계속 집중했습니다.
+
+그 결과 저희 테이블은 5번의 리필~!
+다른 테이블까지 조사는 못했네요~!
+
+이런저런 이야기들을 했지만~!
+영화 관련된 이야기들을 많이 듣게 되어 좋았고~!
+무엇보다
+이번 모임때 수금과 결제를 대신해주신
+장원님께 감사의 말씀을 드립니다~!
+
+주말 모임이 쉽지 않은 분들이어서~!
+아이키우는 일에 대한 고충이 충분이 전달된 시간이었습니다.
+
+그리고 전 9월 28일 모임 장소 견학을 다녀오는 것으로~!
+모임을 끝냈네요~!
+
+이상 곱창팟 모임 후기였습니다~!</textarea>
+   </div>
+</div> 
+
+</div>
+
+
+
+</body>
+</html>
```

tiles 세팅 부분 (header, footer, body 를 연결한다. 해더와 하단 부분을 계속 고정할 수 있는 장치)
File review/src/main/webapp/WEB-INF/template/base.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
+<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
+
+<style>
+.wrapper{
+	width:100%;
+	background:#AAAA0011;
+}
+</style>
+
+<!DOCTYPE html>
+
+<html>
+
+	<head>
+		<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
+		<meta http-equiv="X-UA-Compatible" content="IE=edge">
+		<meta name="viewport" id="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,initial-scale=1.0">
+		<script  src="http://code.jquery.com/jquery-latest.min.js"></script>
+	
+	</head>
+ 
+	<body>
+		<div class="wrapper">
+		    
+		    <tiles:insertAttribute name="header" />
+		    
+		    <tiles:insertAttribute name="body" />
+		    
+		    <tiles:insertAttribute name="footer" />
+		    
+		    <tiles:insertAttribute name="modal" />
+	 	</div>
+	</body>
+</html>

```


File review/src/main/webapp/WEB-INF/template/errorbase.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
+
+<!DOCTYPE html>
+
+<html lang="ko">
+
+	<head>
+	    <!-- header -->
+	    <meta charset="UTF-8">
+	    <title>ERROR | Project</title>
+	    <meta http-equiv="X-UA-Compatible" content="IE=edge">
+	    <meta name="viewport" id="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,initial-scale=1.0">
+	    
+	</head>
+	
+	<body>
+	    <div class="wrapper">
+	
+	        <tiles:insertAttribute name="body" />
+	
+	    </div>
+	</body>
+
+</html>
```

해더가 없는 경우를 미리 세팅
File review/src/main/webapp/WEB-INF/template/nobase.jsp ADDED
```
+<%@ page contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" language="java" %>
+<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
+
+<!DOCTYPE html>
+
+<html>
+	<head>
+	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
+		<meta http-equiv="X-UA-Compatible" content="IE=edge">
+		<meta name="viewport" id="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,initial-scale=1.0">
+		<script  src="http://code.jquery.com/jquery-latest.min.js"></script>
+	</head>
+	 
+	<body>
+		<div class="wrapper">
+	    	<tiles:insertAttribute name="body" />
+		    
+		    <tiles:insertAttribute name="modal" />
+	    </div>
+	</body>
+</html>
File review/src/main/webapp/WEB-INF/tiles/tiles.xml ADDED
 Side-by-side diff View file Comment More
+<?xml version="1.0" encoding="UTF-8"?>
+ 
+<!DOCTYPE tiles-definitions PUBLIC "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN" "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
+ 
+<tiles-definitions>
+ 
+        <!-- base tiles layout add -->
+ 
+        <definition name="base" template="/WEB-INF/template/base.jsp">
+                <put-attribute name="header" value="/WEB-INF/layout/header.jsp" />
+                <put-attribute name="body" value="" />
+                <put-attribute name="footer" value="/WEB-INF/layout/footer.jsp" />
+                <put-attribute name="modal" value="/WEB-INF/layout/modal.jsp" />
+        </definition>
+
+        <definition name="nobase" template="/WEB-INF/template/nobase.jsp">
+                <put-attribute name="body" value="" />
+                <put-attribute name="modal" value="/WEB-INF/layout/modal.jsp" />
+        </definition>
+ 
+        <definition name="login/login" extends="base">
+                <put-attribute name="body" value="/WEB-INF/login/login.jsp" />
+        </definition>
+
+        <definition name="error/*" template="/WEB-INF/template/errorbase.jsp">
+                <put-attribute name="body" value="/WEB-INF/error/error.jsp" />
+        </definition>
+        
+        <definition name="*/*" extends="base">
+                <put-attribute name="body" value="/WEB-INF/{1}/{2}.jsp" />
+        </definition>
+ 
+       
+ 
+</tiles-definitions>
```


테스트 진행시 활용가능한 부분
File review/src/test/java/com/chicken/review/ReviewApplicationTests.java ADDED
```
+package com.chicken.review;
+
+import org.junit.Test;
+import org.junit.runner.RunWith;
+import org.springframework.boot.test.context.SpringBootTest;
+import org.springframework.test.context.junit4.SpringRunner;
+
+@RunWith(SpringRunner.class)
+@SpringBootTest
+public class ReviewApplicationTests {
+
+	@Test
+	public void contextLoads() {
+	}
+
+}
```

**목차**

[Chapter 1. 오리엔테이션과 시작하기](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter1)

[Chapter 2. 협업 툴 활용 A-Z](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter2)

[Chapter 3. 버전관리와 자동화 빌드 툴 이해하기](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter3)

[Chapter 4.5. AWS 기본 설정 및 클라우드 서비스 환경 구축,
 AWS 활용 스프링부트 프로젝트 배포 -> 4](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4)


[Chapter 4-1. 스프링 프로젝트 세팅 실습 1](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(1)spring_project)

[Chapter 4-2. jenkins  실습](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(2)jenkins)

[Chapter 4-3. AWS RDS 설정](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(3)aws_rds)

[Chapter 4-4. AWS S3 설정](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(4)aws_s3)

[Chapter 4-5. AWS 카카오 로그인](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(5)aws_kakao)

[Chapter 4-6. AWS S3 이미지 업로드](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(6)s3_upload)

[Chapter 4-7. Jenkins Pipe Line 만들기](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter4/(7)jenkins_pipeline)


[Chapter 6. DOCKER 활용하기-> 5](https://gitlab.com/bloodjino1/fastcampus-lecture-codes_aws-docker/-/tree/master/chapter5)
