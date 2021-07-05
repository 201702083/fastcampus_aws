
Chapter 4.5. AWS 기본 설정 및 클라우드 서비스 환경 구축,
AWS 활용 스프링부트 프로젝트 배포 -> 4


**AWS를 이해하고 어떻게 클라우드 서비스 환경을 구축하는지 실습을 통해 학습합니다.**

*Chapter 4-6. AWS S3 이미지 업로드*

변경된 파일 목록 리스트
```
+23-12Mreview/pom.xml
+2-1Mreview/src/main/java/com/chicken/review/config/WebSecurityConfig.java
+51-3Mreview/src/main/java/com/chicken/review/login/controller/UserController.java
+1-1Mreview/src/main/java/com/chicken/review/login/mapper/ReviewMapper.java
+61-0Areview/src/main/java/com/chicken/review/login/service/AwsS3Service.java
+8-0Mreview/src/main/java/com/chicken/review/login/vo/ReviewVO.java
+1-1Mreview/src/main/resources/application.properties
+18-0Mreview/src/main/resources/mapper/review.xml
+4-4Mreview/src/main/webapp/WEB-INF/layout/header.jsp
+132-162Mreview/src/main/webapp/WEB-INF/login/login.jsp
```

파일 업로드 관련 라이브러리 추가
File review/pom.xml MODIFIED
```
 	<version>0.0.1-SNAPSHOT</version>
 	<packaging>war</packaging>
 	<name>review</name>
-	  
+	
         
 
 	<description>Review project for Spring Boot</description>
 		<start-class>com.chicken.review.ReviewApplication</start-class>
 	</properties>
 
+
 	<dependencies>
 		<dependency>
 			<groupId>org.springframework.boot</groupId>
 			<artifactId>mybatis-spring-boot-starter</artifactId>
 			<version>2.1.0</version>
 		</dependency>
+		<!-- 
 		<dependency>
 			<groupId>org.springframework.cloud</groupId>
 			<artifactId>spring-cloud-starter-aws-jdbc</artifactId>
 		</dependency>
-
+ 		-->
 		<dependency>
 			<groupId>org.projectlombok</groupId>
 			<artifactId>lombok</artifactId>
             <artifactId>tiles-core</artifactId>
            <version>${org.apache.tiles.version}</version>
         </dependency>
+        <dependency>
+		    <groupId>commons-fileupload</groupId>
+		    <artifactId>commons-fileupload</artifactId>
+		    <version>1.3.3</version>
+		</dependency>
         <!-- 
         <dependency>
         	<groupId>mysql</groupId>
 		    <artifactId>slf4j-api</artifactId>
 		    <version>1.7.10</version>
 		</dependency>
+		
+		<dependency>
+		    <groupId>com.amazonaws</groupId>
+		    <artifactId>aws-java-sdk-s3</artifactId>
+		  </dependency>
         
 	</dependencies>
-
 	<dependencyManagement>
-		<dependencies>
-			<dependency>
-				<groupId>org.springframework.cloud</groupId>
-				<artifactId>spring-cloud-dependencies</artifactId>
-				<version>${spring-cloud.version}</version>
-				<type>pom</type>
-				<scope>import</scope>
-			</dependency>
-		</dependencies>
+	  <dependencies>
+	    <dependency>
+	      <groupId>com.amazonaws</groupId>
+	      <artifactId>aws-java-sdk-bom</artifactId>
+	      <version>1.11.327</version>
+	      <type>pom</type>
+	      <scope>import</scope>
+	    </dependency>
+	  </dependencies>
 	</dependencyManagement>
 
 	<build>
```


웹 시큐리티 파일에서 fileUpload 관련 path 를 추가 
File review/src/main/java/com/chicken/review/config/WebSecurityConfig.java MODIFIED
```
 		.authorizeRequests()
 		 	.antMatchers(staticResources).permitAll()
 		 	.antMatchers("/kakaoLogin").permitAll()
+		 	.antMatchers("/fileUpload").permitAll()
 		 	.anyRequest().authenticated()
 			.and()
 
 			.and()
 		.formLogin() 
 			.loginPage("/login")
-			.defaultSuccessUrl("/")
+			.defaultSuccessUrl("/login")
 			.permitAll()
 			.and()
 		.logout()
```

UserController 에서 AwsS3 관련 서비스를 추가 (파일 업로드 관련 path 및 함수)
File review/src/main/java/com/chicken/review/login/controller/UserController.java MODIFIED
```
 package com.chicken.review.login.controller;
 
+import java.io.File;
+import java.io.IOException;
 import java.util.ArrayList;
+import java.util.List;
 import java.util.Map;
 import java.util.Optional;
 
 import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
 //import org.springframework.security.authentication.AuthenticationManager;
 import org.springframework.security.core.Authentication;
+import org.springframework.security.core.GrantedAuthority;
+import org.springframework.security.core.authority.SimpleGrantedAuthority;
 import org.springframework.security.core.context.SecurityContextHolder;
 import org.springframework.stereotype.Controller;
 import org.springframework.ui.Model;
 import org.springframework.web.bind.annotation.PostMapping;
 import org.springframework.web.bind.annotation.RequestMapping;
+import org.springframework.web.bind.annotation.RequestParam;
 import org.springframework.web.bind.annotation.ResponseBody;
 import org.springframework.web.context.request.RequestContextHolder;
 import org.springframework.web.context.request.ServletRequestAttributes;
+import org.springframework.web.multipart.MultipartFile;
 
+import com.chicken.review.login.service.AwsS3Service;
 import com.chicken.review.login.service.ReviewService;
 import com.chicken.review.login.service.UserService;
+import com.chicken.review.login.vo.ReviewVO;
 import com.chicken.review.login.vo.UserVO;
 
 
     private ReviewService reviewService;
 	@Autowired
     private UserService userService;
+	@Autowired
+    private AwsS3Service awsService;
 	
 	@PostMapping("/kakaoLogin")
 	@ResponseBody
     	HttpServletRequest request = attr.getRequest();
     	HttpServletResponse response = attr.getResponse();
     	
-		System.out.println("TEST!!!!");
+		System.out.println("TEST");
 		System.out.println(user.getEmail());
 		System.out.println(user.getId());
 		System.out.println(user.getNickname());
 		
-		UsernamePasswordAuthenticationToken userAuthenication = new UsernamePasswordAuthenticationToken(user.getId(), "pass");
+		List<GrantedAuthority> roles = new ArrayList<GrantedAuthority>();
+        roles.add(new SimpleGrantedAuthority("ROLE_USER"));
+
+		UsernamePasswordAuthenticationToken userAuthenication = new UsernamePasswordAuthenticationToken(user.getId(), "pass", roles);
 		
 		userAuthenication.setDetails(user);
 		userService.updateUserJoin(user);
 		
 		SecurityContextHolder.getContext().setAuthentication(userAuthenication);
-		HttpSession session= request.getSession(false);
+		HttpSession session= request.getSession();
 		session.setAttribute("user", user);
 		return 1;
 	}
 	
+	
+	@PostMapping("/fileUpload")
+	@ResponseBody
+	public int fileUpload(@RequestParam("mediaFile") MultipartFile file,
+			@RequestParam("title") String title,
+			@RequestParam("content") String content,
+			Model model) throws IllegalStateException, IOException {
+		
+		
+    	System.out.println("fileUpload11");
+    	System.out.println("title::");
+    	System.out.println(title);
+    	System.out.println("content::");
+    	//System.out.println(content);
+    	ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
+    	HttpServletRequest request = attr.getRequest();
+    	HttpServletResponse response = attr.getResponse();
+    	
+    	HttpSession session= request.getSession();
+    	UserVO userVO = (UserVO)session.getAttribute("user");
+    	
+    	ReviewVO reviewVO = new ReviewVO();
+    	reviewVO.setTitle(title);
+    	reviewVO.setContent(content);
+    	reviewVO.setUserId(userVO.getId());
+    	
+		awsService.s3FileUpload(file, reviewVO);
+		
+		return 1;	
+	}
+	
+	
+	
+	
     @RequestMapping(value="/login")
     public String login(Model model, String error, String logout) {
```

리뷰를 입력하는 로직을 추가 (매퍼에 함수추가)
File review/src/main/java/com/chicken/review/login/mapper/ReviewMapper.java MODIFIED
```
 public interface ReviewMapper {
 
 	List<ReviewVO> getReviewList();
-	
+	int insertReview(ReviewVO reviewVO);
 }
```


S3 에 파일을 업로드 하는 로직
File review/src/main/java/com/chicken/review/login/service/AwsS3Service.java ADDED
```
+package com.chicken.review.login.service;
+
+import java.io.File;
+import java.io.IOException;
+
+import org.springframework.beans.factory.annotation.Autowired;
+import org.springframework.stereotype.Service;
+import org.springframework.web.multipart.MultipartFile;
+
+import com.amazonaws.auth.AWSCredentials;
+import com.amazonaws.auth.AWSStaticCredentialsProvider;
+import com.amazonaws.auth.BasicAWSCredentials;
+import com.amazonaws.regions.Regions;
+import com.amazonaws.services.s3.AmazonS3;
+import com.amazonaws.services.s3.AmazonS3Builder;
+import com.amazonaws.services.s3.AmazonS3Client;
+import com.amazonaws.services.s3.model.CannedAccessControlList;
+import com.amazonaws.services.s3.model.PutObjectRequest;
+import com.chicken.review.login.mapper.ReviewMapper;
+import com.chicken.review.login.vo.ReviewVO;
+//위는 해당 객체들 선언 부
+
+@Service
+public class AwsS3Service {
+	private static final String BUCKET_NAME = "reviewaws"; //S3 관련 버킷 정보
+	private static final String ACCESS_KEY = "AKIAS7ZSK5WMPFKEIAUK";//ACCESS KEY 정보
+	private static final String SECRET_KEY = "i09f6sKK6JXIoTxnOj0BNbNMIfGNEBpN/XBY6AVN";//SECRET KEY 정보
+	
+	private AmazonS3 s3;
+	
+	@Autowired
+	private ReviewMapper reviewMapper;
+	
+	public AwsS3Service() {
+		AWSCredentials awsCredentials = new BasicAWSCredentials(ACCESS_KEY, SECRET_KEY);
+		
+		s3 = AmazonS3Client.builder()
+				.withRegion(Regions.AP_NORTHEAST_2) /* 서울 리전 */
+				.withCredentials(new AWSStaticCredentialsProvider(awsCredentials)).build(); /* 위 권한 정보로 권한 획득 */
+	}
+	/* s3 관련 함수 정보 멀티 파트 파일을 받아서 올린다. */
+	public void s3FileUpload(MultipartFile file, ReviewVO reviewVO) throws IllegalStateException, IOException {
+		if(file != null && !file.getOriginalFilename().equals("")) {
+			System.out.println(file.getOriginalFilename());
+			
+			File localFile = new File("/Users/jinhobae/Downloads/" + file.getOriginalFilename());
+			file.transferTo(localFile); 
+		    
+			System.out.println(localFile.getName());
+			PutObjectRequest obj = new PutObjectRequest(BUCKET_NAME, localFile.getName(), localFile);
+			obj.setCannedAcl(CannedAccessControlList.PublicRead); /* s3 버킷에 올리는 로직 */
+			String imageUrl = "https://reviewaws.s3.ap-northeast-2.amazonaws.com/"+localFile.getName();
+			s3.putObject(obj);
+			System.out.println(imageUrl);
+			reviewVO.setS3ImageUrl(imageUrl); /* S3에 올라간 내용을 DB 에 업데이트 하기 위해 VO에 세팅 */
+			
+			reviewMapper.insertReview(reviewVO); /* S3에 올라간 내용을 DB 에 업데이트하는 로직 (매퍼 호출) */
+		}else {
+			reviewMapper.insertReview(reviewVO); /* 이미지가 없는 경우 이미지를 제외하고 호출 */
+		}
+	}
+}
```

리뷰를 올리기 위한 VO 객체 (userId 정보를 추가 한다)
File review/src/main/java/com/chicken/review/login/vo/ReviewVO.java MODIFIED
```
 	String title;
 	String content;
 	String s3ImageUrl;
+	String userId;
 	public int getSeq() {
 		return seq;
 	}
 	public void setS3ImageUrl(String s3ImageUrl) {
 		this.s3ImageUrl = s3ImageUrl;
 	}
+	public String getUserId() {
+		return userId;
+	}
+	public void setUserId(String userId) {
+		this.userId = userId;
+	}
+	
 	
 }
```

연결이 끊어지는 현상 때문에 자동 연결을 설정해준다.
File review/src/main/resources/application.properties MODIFIED
```
 spring.mvc.view.suffix=.jsp
 
 spring.datasource.driverClassName=com.mysql.jdbc.Driver
-spring.datasource.url=jdbc:mysql://review.cm03m1qy5nzi.ap-northeast-2.rds.amazonaws.com:3306/innodb?useUnicode=true&characterEncoding=utf8
+spring.datasource.url=jdbc:mysql://review.cm03m1qy5nzi.ap-northeast-2.rds.amazonaws.com:3306/innodb?useUnicode=true&autoReconnection=true
 spring.datasource.username=admin
 spring.datasource.password=abcd1234
```
 
리뷰를 입력하는 insert 로직 해당 쿼리를 review.xml 에 추가한다.
File review/src/main/resources/mapper/review.xml MODIFIED
```
 	</select>
 	
+	<update id="insertReview"  parameterType="com.chicken.review.login.vo.ReviewVO"  >
+		
+		INSERT INTO review(
+		  userId,
+		  title,
+		  content,
+		  s3ImageUrl,
+		  createDt
+		) VALUES (
+		  #{userId},
+		  #{title},
+		  #{content},
+		  #{s3ImageUrl},
+		  now()
+		)
+			
+	</update>
+	
 	
 </mapper>
```


디자인적으로 간단히 수정한 부분 (해더 부분 수정)
File review/src/main/webapp/WEB-INF/layout/header.jsp MODIFIED
```
 	.logo{
 		padding-top:15px;
 		font-weight:bold;
-    	color: #010101;
+    	color: #FFF;
     	float:left;
 	}
 	.logininfo{
 	    width: 150px;
-	    height: 20px;
-	    text-align: center;
+	    height: 50px;
 	    float: right;
-	    padding: 15px;
+	    color: #FFF;
+	    padding: 12px;
 	}
 </style>
 <div class="header">
```

화면 수정 기존에 텍스트 형태로 저장된 영역을 DB 에서 불러와서 처리하도록 수정된 부분
데이터 바인딩이 모두 끝났기 때문에 기존에 이미지 URL 을 활용했던 부분이나 그런 부분들을 DB 에서 가져와 사용하는 방식으로 대체
File review/src/main/webapp/WEB-INF/login/login.jsp MODIFIED
```
 	}
 	.itemListWrapper{
 		width: 960px;
-		height: 480px;
 		margin: 5px auto;
 		text-align: center;
 	}
 	.itemList{
-		float:left;
-		width: 306px;
-		height: 400px;
-		margin: 5px;
-		border-radius: 5px;
-		border : 2px solid #DDDDDD;
+	    width: 300px;
+	    display: inline-block;
+	    height: 400px;
+	    margin: 5px;
+	    border-radius: 5px;
+	    border: 2px solid #DDDDDD;
 	}
 	.imageArea{
 		width: 100%;
 		transform: translateY(0px);
         opacity: 1;
     }
-    
+    .write-btn{
+    	width: 100px;
+    	height: 28px;
+    	margin: 0px auto;
+    	background: linear-gradient(to right, #C02425, #F0CB35);
+    	color:white;
+    	border-radius: 5px;
+    	opacity: 0.8;
+    	cursor:pointer;
+    	padding-top:5px;
+    	margin-top:20px;
+    	border-top: 1px solid #DDDDDD99;
+    } 
+    .write-btn:hover{
+    	
+    	opacity: 1;
+    }
    
+    .cover-form{
+	    width: 300px;
+	    height: 450px;
+	    background: white;
+	    position: fixed;
+	    z-index: 10;
+	    border-radius: 10px;
+	    padding-top: 10px;
+    }
+    .form-title{
+    	font-size: 25px;
+    	font-weight: bold;
+    	width: 100%;
+    	text-align: center;
+    }
+    .form-desc{
+    	font-size: 14px;
+    	text-align: left;
+    	padding: 20px;
+    }
+    
+    .input-title{
+    	width: 150px;
+    	height: 40px;
+    	text-align: left;
+    	font-size: 14px;
+    }
+    .input-content{
+    	width: 250px;
+    	height: 200px;
+    	font-size: 14px;
+    }
 </style>
 
 <script type="text/javascript">
 
 $(document).ready(function(){
+	
+	$("#cover-form").hide();
 	<c:if test="${ user == null }" >
+	$("#write-btn").hide();
+	
 	Kakao.init('76a34fcaefc9ee59cfe964e231bf3bbf');
     // 카카오 로그인 버튼을 생성합니다.
     Kakao.Auth.createLoginButton({
       container: '#kakao-login-btn',
       success: function(authObj) {
-        
-        
+        //alert(JSON.stringify(authObj));
        Kakao.API.request({url:'/v1/user/me',
     	   success:function (res){
-    		   alert(JSON.stringify(res));
     		   var id = res.id;
     		   var email = (res.kaccount_email ? res.kaccount_email : '');
     		   var nickname = (res.properties && res.properties.nickname ? res.properties.nickname : '');
     		   
-    		   alert(id);
-    		   alert(email);
-    		   alert(nickname);
+    		  
     		   nickname = '치킨';
     		   
     		   $("#logininfo").text(nickname);
 	   			 		if(data == 1){
 	   			 			alert("로그인이 완료 되었습니다.");
 	   			 			$("#kakao-login-btn").hide();
-		   			 		
+		   			 		$("#write-btn").show();
 	   			 		}
 	   			 	}
     		   )
     		   
     		   
     		   
-    		   
     	   },
     	   fail:function (error){
     		   
       fail: function(err) {
          alert(JSON.stringify(err));
       }
-      
     });
     </c:if>
     var slideAelements = $('.slide-child')
     }
     animateSlideA() ;
     
+    
+    $("#write-btn").click(function (){
+    	
+    	$("#cover-form").show();
+    });
+	$("#cancelBtn").click(function (){
+    	
+    	$("#cover-form").hide();
+    });
+    
+    
+    $("#btnSubmit").click(function (event) {
+    	 
+        //preventDefault 는 submit을 막음
+        event.preventDefault();
+ 
+        var form = $('#fileUploadForm')[0];
+ 
+        var data = new FormData(form);
+ 
+        $("#btnSubmit").prop("disabled", true);
+ 
+        $.ajax({
+            type: "POST",
+            enctype: 'multipart/form-data',
+            url: "/fileUpload",
+            data: data,
+            processData: false,
+            contentType: false,
+            cache: false,
+            timeout: 600000,
+            success: function (data) {
+                alert("complete");
+                $("#btnSubmit").prop("disabled", false);
+                $("#cover-form").hide();
+                location.reload(true);
+            },
+            error: function (e) {
+                console.log("ERROR : ", e);
+                $("#btnSubmit").prop("disabled", false);
+                alert("fail");
+                $("#cover-form").hide();
+            }
+        });
+ 
+    });
+    
 });
 </script>
 
 		
 	</div>
 	</c:if>
+	<div id="cover-form" class="cover-form">
+		<div class="form-title">리뷰 쓰기</div>
+		<div class="form-desc">오프라인 행사 리뷰를 작성해주세요.</div>
+		<form method="POST" action="/fileUpload" enctype="multipart/form-data" id="fileUploadForm">
+			제목 <input type=text name="title" class="input-title" />
+			<br/>내용<br/>
+			<textarea name="content"  class="input-content" ></textarea>
+		
+		    <input type=file name="mediaFile" > <br/>
+		    <input type="submit" value="저장하기" id="btnSubmit"/>
+		    <input type="button" value="취소하기" id="cancelBtn"/>
+		</form>
+		
+	</div>
+	<div id="write-btn" class="write-btn">
+		글쓰기
+	</div>
 	
 </div>
 
 <div class="itemListWrapper">
+<c:forEach var="item" items="${reviewList}" varStatus="status">
+
 <div class="itemList slide-child">
-   <div class="imageArea"><img src="<c:out value="${ reviewList[0].s3ImageUrl }" />"/></div>
-   <div class="reviewArea">
-   	  <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
-      <textarea readonly><c:out value="${ reviewList[0].content }" /></textarea>
-   </div>
-</div>
-<div class="itemList slide-child">
-   <div class="imageArea"><img src="img/img2.jpg"/></div>
+   <div class="imageArea"><img src="<c:out value="${ item.s3ImageUrl }" />"/></div>
    <div class="reviewArea">
-  	  <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
-      <textarea readonly>일시 : 2019년 9월 28일
-장소 : 공덕 IOT COC (1차), 바른치킨(2차)
-시간 : 3시, 6시
-인원 1차 60명 2차 50명
-
-이번 모임은 공덕에서 진행되었습니다.
-장소를 빌려주신 IOT COC 담당분들께는 감사말씀 드립니다.
-
-이번 모임은 1차에서는 주로 세미나 위주
-2차에서는 치킨을 위주로 진행되었습니다.
-
-폰이 말썽이라 배터리가 다 닳아버린 이후로는
-개인적으로는 고통의 시간들이었는데요.
-
-연락들도 많이오고 하는 과정에서 차가막히고 늦게되는 바람에...
-이기정 군이 고생을 많이해주었고요~!
-
-그리고 다른 분들도 도와주셔서 그래도 모임을 잘 진행하게되었네요.
-첫번째는 이모티콘 강의
-다들 이 강의를 매우 흥미로워했는데요.
-2차 모임에서 어떤 그룹은 이모티콘을 만들어 돈벌자며
-결의를 다졌다는 후문이 있습니다.
-
-그리고 두번째 강의는 크리에이터 멘땅에 해딩하기~! 이 시간은 어렵지 않아서 많은 분들이 편안! 해 했던 세션이었는데요~
-다양한 시행착오를 거쳐서 틱톡 크리에이터가 되기까지~! 더 유명해지면 얼굴보기 어려운 분이 되실 수도 있겠네요~!
-
-그리고 세번째 강의 세션은 스타트업을 위한 개발 경험 공유, 해왔던 일들을 가지고 이런저런 개발에 필요한 것이 무엇인지 돌아볼 수 있는 시간이었네요.
-
-그리고 4번째강의는 왓챠플래이 신입 재직중인 기정군이 다양한 툴과 정보들을 잘 정리해서 올려주었어요~
-
-5번째 스칼라 강의가 아쉬웠는데...
-시간배분의 문제로 간단한 설명까지밖에 못했네요.
-
-다음 강의 때 해주시기로 하셨어요~! 다음 10월 모임엔 Handson 세션으로 꼭 해야할듯싶네요~!
-
-특별히 간식과 음료를 잘 준비해주셔서~!
-다들 맛있게 잘 먹었다고 합니다..
-
-2차 모임은 48명이 소소하니 모여~!
-치킨과 대화를 나누며 시간들을 보내었습니다.
-몇분이 못오셨는데 기부..
-하신건 맛있게 잘 먹었습니다.
-
-그리고 9시반 이후에 카페로 이동해서
-2차모임의 카페인증까지~!
-
-이번 모임도 다양한 분들과 함께
-유익하고 재미있는 시간을 보내었고요~!
-
-10월 모임을 기대해 봅니다.
-이번 포스터 제작자분도 모임에 와주시고~!
-다양한 분들이 소통했던 9월 모임이었습니다.
-
-이상 감사합니다.</textarea>
-
+   	  <div class="reviewTitle" ><c:out value="${ item.title }" /> </div>
+      <textarea readonly><c:out value="${ item.content }" /></textarea>
    </div>
 </div>
-<div class="itemList slide-child">
-   <div class="imageArea"><img src="img/img3.jpg"/></div>
-   <div class="reviewArea" >
-      <div class="reviewTitle" ><c:out value="${ reviewList[0].title }" /> </div>
-      <textarea readonly>[2019년 9월 19일 7시 곱창팟 치킨모임 후기]
-
-장소 : 홍대 가오곱창
-일시 : 9월 19일 7시
-인원 : 13명
-회비 : 25000 => 24000
-
-후기
-이번 모임은 특별히
-치킨이 아니라 곱창 모임이었다는 점.
-그리고 주말 모임이 아니라
-평일 모임이었다는 점이 달랐습니다~!
-
-제가 도착하는 시점이미
-라면이 세팅되어있었습니다.
-
-그러니까 갑자기 좀 찌개를 먹어야할 것 같은 그런
-분위기!...
-
-그리고 수저는 어디갔는지 안보이는 상황
-솔직히 자리는 좀 좁고... 이 가오곱창집
-가운데 마치 무인도처럼 가운데 저희 모임이 떠있는듯한 형국!
-
-기존의 치킨모임들의 경우 어색함의 시간이
-그리 길지 않았던 반면
-이번 모임은 살짝 어색함이 풀어지는 시간에
-딜레이가 있긴했습니다.
-
-가게 한가운데 저희가 있었기 때문에....
-
-어찌 되었든
-이 가게의 전략을 처음 파악했습니다.
-바로 그것은...
-라면과 주먹밥
-평소 같았다면 빠르게 먹었겠지만
-이 가게의 전략적 승부에 모임분들은 대응하지 않았습니다.
-
-과감하게 본론으로 들어가길 원했는데요~!
-문제는
-생각보다 할게 많다는점...
-고기를 굽거나 곱창을 굽기위한 노동력
-문제로 대화는 버벅버벅
-
-그래도~! 첫인원은 5명
-그리고 7명 10명
-11명 12명 13명 식으로 늘어나는 사람들~!
-
-덕분에 인사를 3번정도 했는데요.
-평일이라 급작스레 못오시는 분도 계셨습니다.
-
-적당한 소개는 10~11명 정도 모인 시점에 한번 진행되었습니다~!
-
-오신분들은 2~3년차 웹개발자~! 개발자를 구하시는 블록체인 대표님 그리고 십여년차 자바 개발자님을 비롯해, 7년간 에이젼시를 운영하신 분, 영화관련 일을 하시다가~! 스타트업에 뛰어드신분! 그리고 작은 사업을 하시다가 개발을 배우신분~! 그리고 스타트업을 기획하시는 분, 그리고 뮤지컬을 보는 일과 다양한 일들을 하시는 디자이너님, 늘 다른분의 이야기를 청종하시는 기획자님, 그리고 디자이너님과 친구분이 추가로 오셨는데요~!
-
-이 정도 인원이 되니 그래도 살짝씩은 대화를 섞어볼수 있어서 좋았습니다.
-
-물론 모임이 이번 모임 특성이 곱창을 뽀개는 것이었기 때문에 리필하는 것에 계속 집중했습니다.
-
-그 결과 저희 테이블은 5번의 리필~!
-다른 테이블까지 조사는 못했네요~!
-
-이런저런 이야기들을 했지만~!
-영화 관련된 이야기들을 많이 듣게 되어 좋았고~!
-무엇보다
-이번 모임때 수금과 결제를 대신해주신
-장원님께 감사의 말씀을 드립니다~!
-
-주말 모임이 쉽지 않은 분들이어서~!
-아이키우는 일에 대한 고충이 충분이 전달된 시간이었습니다.
-
-그리고 전 9월 28일 모임 장소 견학을 다녀오는 것으로~!
-모임을 끝냈네요~!
-
-이상 곱창팟 모임 후기였습니다~!</textarea>
-   </div>
-</div> 
-
+</c:forEach>
 </div>
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
