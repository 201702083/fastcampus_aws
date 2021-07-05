
Chapter 4.5. AWS 기본 설정 및 클라우드 서비스 환경 구축,
AWS 활용 스프링부트 프로젝트 배포 -> 4


**AWS를 이해하고 어떻게 클라우드 서비스 환경을 구축하는지 실습을 통해 학습합니다.**

*Chapter 4-7. Jenkins Pipe Line 만들기*


젠킨스 > 구성 > 스크립트 

```
pipeline {
   agent any

   tools {
      maven "maven＂ // 메이븐 기존에 세팅한 명칭
   }

   stages {
       stage('git Pull') {
         steps {
             git 'https://baejinho@bitbucket.org/devOpsReview/reviewaws.git’ // 세팅한 git 주소
         }
       }
      stage('Build') {
         steps {
            sh "mvn -Dmaven.test.failure.ignore=true -N -f review/pom.xml clean package＂ // 빌드하기, pom.xml 의 위치
         }

      }
      stage('Deploy'){
           steps {
                sh "scp -i \"/Users/jinhobae/Documents/key/chicken.pem\" review/**/*.war ec2-user@13.209.48.204:/opt/tomcat/webapps"  // WAR 파일 복사 구문
           }
      }
       stage('RESTART'){
           steps {
                sh "ssh -i \"/Users/jinhobae/Documents/key/chicken.pem\" ec2-user@13.209.48.204 sudo systemctl restart tomcat＂  // 재시작 명령어
           }
          
      }
   }
}
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
