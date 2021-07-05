
Chapter 4.5. AWS 기본 설정 및 클라우드 서비스 환경 구축,
AWS 활용 스프링부트 프로젝트 배포 -> 4


**AWS를 이해하고 어떻게 클라우드 서비스 환경을 구축하는지 실습을 통해 학습합니다.**

*Chapter 4-5. AWS 카카오 로그인*



> 변경한 파일 목록 
```
+11-22Mreview/pom.xml
+1-0Mreview/src/main/java/com/chicken/review/config/WebSecurityConfig.java
+38-7Mreview/src/main/java/com/chicken/review/login/controller/UserController.java
+11-0Areview/src/main/java/com/chicken/review/login/mapper/UserMapper.java
+27-0Areview/src/main/java/com/chicken/review/login/service/UserService.java
+32-0Areview/src/main/java/com/chicken/review/login/vo/UserVO.java
+1-1Mreview/src/main/resources/application.properties
+25-0Areview/src/main/resources/mapper/user.xml
+19-1Mreview/src/main/webapp/WEB-INF/layout/header.jsp
+43-4Mreview/src/main/webapp/WEB-INF/login/login.jsp
```


> 라이브 러리 변경 수정 작업  
File review/pom.xml MODIFIED

```
 		<start-class>com.chicken.review.ReviewApplication</start-class>
 	</properties>
 
-
 	<dependencies>
 		<dependency>
 			<groupId>org.springframework.boot</groupId>
 			<artifactId>mybatis-spring-boot-starter</artifactId>
 			<version>2.1.0</version>
 		</dependency>
-		<!-- 
 		<dependency>
 			<groupId>org.springframework.cloud</groupId>
 			<artifactId>spring-cloud-starter-aws-jdbc</artifactId>
 		</dependency>
- 		-->
+
 		<dependency>
 			<groupId>org.projectlombok</groupId>
 			<artifactId>lombok</artifactId>
             <artifactId>tiles-core</artifactId>
            <version>${org.apache.tiles.version}</version>
         </dependency>
-        <dependency>
-		    <groupId>commons-fileupload</groupId>
-		    <artifactId>commons-fileupload</artifactId>
-		    <version>1.3.3</version>
-		</dependency>
         <!-- 
         <dependency>
         	<groupId>mysql</groupId>
 		    <artifactId>slf4j-api</artifactId>
 		    <version>1.7.10</version>
 		</dependency>
-		
-		<dependency>
-		    <groupId>com.amazonaws</groupId>
-		    <artifactId>aws-java-sdk-s3</artifactId>
-		  </dependency>
         
 	</dependencies>
+
 	<dependencyManagement>
-	  <dependencies>
-	    <dependency>
-	      <groupId>com.amazonaws</groupId>
-	      <artifactId>aws-java-sdk-bom</artifactId>
-	      <version>1.11.327</version>
-	      <type>pom</type>
-	      <scope>import</scope>
-	    </dependency>
-	  </dependencies>
+		<dependencies>
+			<dependency>
+				<groupId>org.springframework.cloud</groupId>
+				<artifactId>spring-cloud-dependencies</artifactId>
+				<version>${spring-cloud.version}</version>
+				<type>pom</type>
+				<scope>import</scope>
+			</dependency>
+		</dependencies>
 	</dependencyManagement>
 
 	<build>
```

> 카카오 로그인 관련 path 추가 작업
File review/src/main/java/com/chicken/review/config/WebSecurityConfig.java MODIFIED

```
 			.and()
 		.authorizeRequests()
 		 	.antMatchers(staticResources).permitAll()
+		 	.antMatchers("/kakaoLogin").permitAll()
 		 	.anyRequest().authenticated()
 			.and()
 ```


> UserController 관련 로그인 경로 잡아주기
File review/src/main/java/com/chicken/review/login/controller/UserController.java MODIFIED

```
 package com.chicken.review.login.controller;
 
+import java.util.ArrayList;
 import java.util.Map;
 import java.util.Optional;
 
 import javax.servlet.http.HttpServletRequest;
 import javax.servlet.http.HttpServletResponse;
+import javax.servlet.http.HttpSession;
 
 import org.slf4j.Logger;
 import org.slf4j.LoggerFactory;
 import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
 //import org.springframework.security.authentication.AuthenticationManager;
 import org.springframework.security.core.Authentication;
 import org.springframework.security.core.context.SecurityContextHolder;
 import org.springframework.stereotype.Controller;
 import org.springframework.ui.Model;
+import org.springframework.web.bind.annotation.PostMapping;
 import org.springframework.web.bind.annotation.RequestMapping;
+import org.springframework.web.bind.annotation.ResponseBody;
 import org.springframework.web.context.request.RequestContextHolder;
 import org.springframework.web.context.request.ServletRequestAttributes;
 
 import com.chicken.review.login.service.ReviewService;
+import com.chicken.review.login.service.UserService;
+import com.chicken.review.login.vo.UserVO;
 
 
 
 	
 	@Autowired
     private ReviewService reviewService;
+	@Autowired
+    private UserService userService;
+	
+	@PostMapping("/kakaoLogin")
+	@ResponseBody
+	public int kakaoLogin(UserVO user) {
+		ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
+    	HttpServletRequest request = attr.getRequest();
+    	HttpServletResponse response = attr.getResponse();
+    	
+		System.out.println("TEST!!!!");
+		System.out.println(user.getEmail());
+		System.out.println(user.getId());
+		System.out.println(user.getNickname());
+		
+		UsernamePasswordAuthenticationToken userAuthenication = new UsernamePasswordAuthenticationToken(user.getId(), "pass");
+		
+		userAuthenication.setDetails(user);
+		userService.updateUserJoin(user);
+		
+		SecurityContextHolder.getContext().setAuthentication(userAuthenication);
+		HttpSession session= request.getSession(false);
+		session.setAttribute("user", user);
+		return 1;
+	}
 	
-
     @RequestMapping(value="/login")
     public String login(Model model, String error, String logout) {
     	
-    	ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
-    	HttpServletRequest request = attr.getRequest();
-    	HttpServletResponse response = attr.getResponse();
+    	
 
     	Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
     	
     	
     	model.addAttribute("reviewList", reviewService.getReviewList());
     	
-    	System.out.println(reviewService.getReviewList());
     	if(authentication != null) {
+    		System.out.println("LOGIN");
+    		
     		
-	    	if(!authentication.getPrincipal().equals("anonymousUser"))
-	    		return "redirect:/";
+    		return "login/login";
+	    	//if(!authentication.getPrincipal().equals("anonymousUser"))
+	    	//	return "redirect:/";
     	}
```     	
     	
> 유저 DB 와 연결 작업 (통칭 Mapper 작업)
File review/src/main/java/com/chicken/review/login/mapper/UserMapper.java ADDED
```
+package com.chicken.review.login.mapper;
+
+import org.apache.ibatis.annotations.Mapper;
+
+import com.chicken.review.login.vo.UserVO;
+
+
+@Mapper
+public interface UserMapper {
+	int updateUserJoin(UserVO userVO);
+}
```

> 유저 Service 와 매퍼 연결 작업
File review/src/main/java/com/chicken/review/login/service/UserService.java ADDED
```
+package com.chicken.review.login.service;
+
+import java.util.List;
+
+import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.stereotype.Service;
+
+import com.chicken.review.login.mapper.ReviewMapper;
+import com.chicken.review.login.mapper.UserMapper;
+import com.chicken.review.login.vo.ReviewVO;
+import com.chicken.review.login.vo.UserVO;
+
+@Service
+public class UserService {
+	
+	
+	@Autowired
+	private UserMapper userMapper;
+	
+
+	public int updateUserJoin(UserVO userVO) {
+		
+		return userMapper.updateUserJoin(userVO);
+	}
+	
+	
+}
```

> 유저 VO 생성작업 (value Object - Data TYPE OBJECT 라고도 함)
File review/src/main/java/com/chicken/review/login/vo/UserVO.java ADDED

```
+package com.chicken.review.login.vo;
+import lombok.Data;
+
+@Data
+public class UserVO {
+
+	String id;
+	String email;
+	String nickname;
+	public String getId() {
+		return id;
+	}
+	public void setId(String id) {
+		this.id = id;
+	}
+	public String getEmail() {
+		return email;
+	}
+	public void setEmail(String email) {
+		this.email = email;
+	}
+	public String getNickname() {
+		return nickname;
+	}
+	public void setNickname(String nickname) {
+		this.nickname = nickname;
+	}
+	
+	
+	
+	
+}
```


> DB 와 연결 작업 기존에 RDS 와 연결했다면 RDS endpoint 와 연결 그리고 root id 와 패스워드 정보를 입력 
File review/src/main/resources/application.properties MODIFIED
```
 spring.mvc.view.suffix=.jsp
 
 spring.datasource.driverClassName=com.mysql.jdbc.Driver
-spring.datasource.url=jdbc:mysql://review.cm03m1qy5nzi.ap-northeast-2.rds.amazonaws.com:3306/innodb
+spring.datasource.url=jdbc:mysql://review.cm03m1qy5nzi.ap-northeast-2.rds.amazonaws.com:3306/innodb?useUnicode=true&characterEncoding=utf8
 spring.datasource.username=admin
 spring.datasource.password=abcd1234
 
```


> 유저 관련된 DB 쿼리 (입력, 로그인 관련)
File review/src/main/resources/mapper/user.xml ADDED

```
+<?xml version="1.0" encoding="UTF-8"?>
+<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
+<mapper namespace="com.chicken.review.login.mapper.UserMapper">
+
+	
+
+	<update id="updateUserJoin"  parameterType="com.chicken.review.login.vo.UserVO"  >
+		
+		INSERT INTO user(
+		  id,
+		  nickname,
+		  email
+		) VALUES (
+		  #{id},
+		  #{nickname},
+		  #{email}
+		)
+		 ON DUPLICATE KEY UPDATE
+		 nickname = #{nickname},
+		 email =  #{email}
+			
+	</update>
+	
+	
+</mapper>
```

> 유저 관련된 DB 쿼리 (입력, 로그인 관련)
File review/src/main/webapp/WEB-INF/layout/header.jsp MODIFIED
```
 		padding-top:15px;
 		font-weight:bold;
     	color: #010101;
+    	float:left;
+	}
+	.logininfo{
+	    width: 150px;
+	    height: 20px;
+	    text-align: center;
+	    float: right;
+	    padding: 15px;
 	}
 </style>
 <div class="header">
 	<div class="headerWrapper">
-	<div class="logo">OffREV</div>
+		<div class="logo">OffREV
+		
+		</div>
+		<div id="logininfo" class="logininfo">
+				
+		<c:if test="${ user != null }" >
+			<c:out value="${ user.nickname }" />
+		</c:if>
+		</div>
 	</div>
+	
+	
 </div>
 
 </html>
```

File review/src/main/webapp/WEB-INF/login/login.jsp MODIFIED
```
 <script type="text/javascript">
 
 $(document).ready(function(){
-	Kakao.init('b30526376249afa40b9d4f5c977a841f');
+	<c:if test="${ user == null }" >
+	Kakao.init('76a34fcaefc9ee59cfe964e231bf3bbf');
     // 카카오 로그인 버튼을 생성합니다.
     Kakao.Auth.createLoginButton({
       container: '#kakao-login-btn',
       success: function(authObj) {
-        alert(JSON.stringify(authObj));
+        
+        
+       Kakao.API.request({url:'/v1/user/me',
+    	   success:function (res){
+    		   alert(JSON.stringify(res));
+    		   var id = res.id;
+    		   var email = (res.kaccount_email ? res.kaccount_email : '');
+    		   var nickname = (res.properties && res.properties.nickname ? res.properties.nickname : '');
+    		   
+    		   alert(id);
+    		   alert(email);
+    		   alert(nickname);
+    		   nickname = '치킨';
+    		   
+    		   $("#logininfo").text(nickname);
+    		   $.post("/kakaoLogin",
+	   			   {id:id, email : email, nickname : nickname}
+	   			 	, function (data){
+	   			 		
+	   			 		
+	   			 		if(data == 1){
+	   			 			alert("로그인이 완료 되었습니다.");
+	   			 			$("#kakao-login-btn").hide();
+		   			 		
+	   			 		}
+	   			 	}
+    		   )
+    		   
+    		   
+    		   
+    		   
+    	   },
+    	   fail:function (error){
+    		   
+    	   }})
+       
       },
       fail: function(err) {
          alert(JSON.stringify(err));
       }
+      
     });
-    
+    </c:if>
     var slideAelements = $('.slide-child')
     
     
 		<div class="slide-child">오프라인 후기 정보들을</div>
 		<div class="slide-child">모아모아 제공합니다.</div>
 	</div>
+	<c:if test="${ user == null }" >
 	<div id="kakao-login-btn">
 		
 	</div>
-	
+	</c:if>
 	
 </div>
 
 
 </body>
 </html>
+
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
