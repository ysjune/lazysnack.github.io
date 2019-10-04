## nGrinder 란?
- Stress Test Platform
- 테스트 스크립트 작성, 테스트 실행, 대상 서버 모니터링 및 결과 작성
- [nGrinder](https://github.com/naver/ngrinder)

#### nGrinder 구조

![controller architecture](https://raw.githubusercontent.com/wiki/naver/ngrinder/assets/Architecture-29bb2.png)
- Controller : 테스팅을 위한 인터페이스 제공 (스크립트 작성 및 수정, 테스트 코디네이트)
- Agent : 타겟 서버에 부하를 가함 (프로세스, 스레드)

#### 설치 방법
- [nGrinder Controller 파일](https://github.com/naver/ngrinder/releases)
- Tomcat 을 이용한 로컬 설치 방법
  * 다운로드 받은 war 파일을 `${TOMCAT_HOME}/webapps` 에 놓는다. 
    + (Context-Path 를 변경하고 싶지 않을 경우 ROOT.war 로 파일명 변경)
  * catalina.sh 혹은 catalina.bat 파일에 라인 추가
    + ``` JAVA_OPTS="-Xms600m -Xmx1024m -XX:MaxPermSize=200m"    # for catalina.sh ```
    + ``` set JAVA_OPTS=-Xms600m -Xmx1024m -XX:MaxPermSize=200m   # for catalina.bat ```
  * 톰캣 실행 (startup.sh 혹은 startup.bat)
  * 브라우저 실행 (http://localhost:8080/ngrinder-controller-X.X)
    + (ROOT.war 로 한 경우 http://localhost:8080)      
<br>
<br>
- Docker 를 이용한 가상 설치 방법 (추후 작성 예정)



#### 실행 방법
