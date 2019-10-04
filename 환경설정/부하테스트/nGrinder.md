## nGrinder 란?
- Stress Test Platform
- 테스트 스크립트 작성, 테스트 실행, 대상 서버 모니터링 및 결과 작성
- [nGrinder](https://github.com/naver/ngrinder)

### nGrinder 구조

![controller architecture](https://raw.githubusercontent.com/wiki/naver/ngrinder/assets/Architecture-29bb2.png)
- Controller : 테스팅을 위한 인터페이스 제공 (스크립트 작성 및 수정, 테스트 코디네이트)
- Agent : 타겟 서버에 부하를 가함 (프로세스, 스레드)

### 설치 방법 (1/2 컨트롤러 설치)
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

### 설치 방법 (2/2 에이전트 설치)
- 실행방법의 `1. 로그인 ` 을 진행 후, `2. 메인 화면` 에서 진행

![agent_install](https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/agent_install.png)

- 다운로드 된 에이전트 파일을 압축 해제 후 run_agent.sh 혹은 run_agent.bat 실행

![agent_management](https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/agnet_management.PNG)

- Agent Management 에서 에이전트 확인  

- 하나의 Agent 에서 Multi Agent 를 실행할 경우
  * Controller 는 Ip 와 Host Id 의 조합으로 에이전트를 판별함
   > 1. 각각 다른 곳에 설치
      * 1st : ~/apps/ngrinder_agent_1
      * 2nd : ~/apps/ngrinder_agent_2
   > 2. host 이름을 변경하여 각 에이전트 사용 (커멘트 라인 사용)
      * 1st : cd ~/apps/ngrinder_agent-1; ./run_agent.sh --agent-home ~/.ngrinde-agent-1 --host-id first-agent
      * 2nd : cd ~/apps/ngrinder_agent-1; ./run_agent.sh --agent-home ~/.ngrinde-agent-2 --host-id second-agent

<br>
### Docker 를 이용한 가상 설치 방법 (추후 작성 예정)

### 실행 방법

#### 1. 로그인

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/ngrinder_main.PNG" width="500px" height="500px"></img>

 - admin/admin 으로 로그인

#### 2. 메인 화면

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/ngrinder_main2.PNG" width="300px" height="300px"></img>

 - URL 을 입력하여 바로 테스트를 진행할 수 있음

3. 스크립트

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_list.PNG" width="300px" height="300px"></img>

 - 스크립트 목록이 나오며 `Create a script` 로 스크립트를 작성할 수 있음

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_detail.PNG" width="300px" height="300px"></img>

- Sample.groovy 파일을 만들어서 스크립트 작성
- 스크립트 파일은 Groovy 와 Jython, Groovy Maven Project 3가지 형태로 작성 가능
- 작성 샘플 파일은 [Script Collection](https://github.com/naver/ngrinder/wiki/Script-Collection) 참조

4. 테스트 작성

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/performance_list.PNG" width="300px" height="300px"></img>

- 상단에 `Performance Test` 선택 하면 테스트 목록이 나오며, `Create Test` 를 통해 테스트 생성

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_detail(sample).PNG" width="300px" height="300px"></img>

- 테스트 이름, Agent 갯수, Vuser 갯수, Script 등을 설정
  + Vuser 란?
    * 가상 사용자로 스레드에 매핑됨, 즉 1000개의 스레드가 있는 경우 1000명의 가상 사용자가 있음을 의미
  + Target Host
    * 하나의 도메인에 여러 개의 아이피 혹은 포트가 지정되어 있는 경우(클러스터링) Domain 과 IP 를 적어줌.
  + Duration / Run Count
    * Duration : 테스트를 얼마나 진행할 것인지
    * Run Count : 테스트를 몇 번 진행할 것인지 (Ex. Run Count : 2, Vuser : 6, agent : 2 -> 2*6*2 = 24)
  + Enabled Ramp-Up
    * 스레드 혹은 프로세스 수를 점진적으로 증가시키고 싶을 경우 사용
  

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_detail(sample2).PNG" width="300px" height="300px"></img>

- 테스트를 시작할 때 바로 시작하거나 스케쥴을 지정해서 시작할 수 있음

5. 테스트 진행

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/performance_detail(ing).PNG" width="300px" height="300px"></img>

- 테스트 진행 중

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_detail(sample3).PNG" width="300px" height="300px"></img>

- 테스트 완료 후
- Detail Repoprt 를 통해 상세 리포트를 보거나 코멘트를 남길 수 있음

6. 테스트 리포트

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_report.PNG" width="300px" height="300px"></img>

<img src="https://github.com/ysjune/study/blob/master/%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95/%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/resources/script_report2.PNG" width="300px" height="300px"></img>

- 테스트 리포트를 통해 상세한 내용을 볼 수 있으며, CSV 파일로 다운 받을 수도 있음

