이상하게 맥에서 로컬 mysql 이든 mariadb든 다 제대로 설치가 안되서,  
주변에서 도커로 설치해보라는 얘기를 듣고... 도커로 설치 진행 (현재 진행된 부분도 삽질을 많이 하면서 왔기 때문에.. 다시 설치할 경우 최소한 검색시간이라도 줄이기 위해 적어둔다)  

### 도커 설치
- Mac : https://docs.docker.com/docker-for-mac/install/

### mariadb OR mysql 설치
<pre><code>
# mysql
$ docker run --name mysql-db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secure -d mysql

# mariadb
$ docker run --name maria-db -p 3306:3306 -e MYSQL_ROOT_PASSWORD=secure -d mariadb
</code></pre>

### mariadb 도커 컨테이너 실행
<pre><code>
docker container run -d -p 13306:3306     \
-e MYSQL_ROOT_PASSWORD=root         \
-v /Users/Shared/data/mariadb:/var/lib/mysql     \
--name mariadb_local mariadb
</code></pre>

볼륨 로컬에서 저장하는 건 처음에는 *-v `pwd`/wp_db:/var/lib/mysql \* 뭐 이런식으로 하려고 했는데 맥에서 진행하니 권한이니 뭐니 나와서 빠르게 포기  

### 컨테이너 접속
<pre><code>
# 도커에 있는 mysql 컨테이너 접근
docker exec -it mariadb_local bash

# 접속방법
mysql -u root -p
 
# 도커로그 보기
docker logs -f --tail=10 maria
</code></pre>

### 컨데이너 접속했을 시 vi가 동작 안하면..
<pre><code>
apt-get update
apt-get install vim
</code></pre>
