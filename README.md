## Docker PHP Example
> 도커 설치부터 PHP 개발까지 다루는 예제
## AWS 인스턴스 생성 및 시작
1. AWS EC2 관리 페이지로 이동: [EC2 관리 페이지](https://ap-northeast-2.console.aws.amazon.com/ec2/v2/home)
2. [인스턴스 시작] 버튼을 눌러 [Ubuntu Server 18.04 LTS (HVM), SSD Volume Type]로 인스턴스 생성
3. [t2.micro] 인스턴스 유형을 선택한 뒤 [검토 및 시작] (도커를 이용하므로 가능하다면 24GB 설정)
4. 키 페어를 이용하여 [인스턴스 시작]
## AWS 인스턴스 접속
1. 키 페어 파일 준비
2. 키 페어에 적절한 권한 주기
> 리눅스: 다음의 명령어로 권한 설정
<pre>
chmod 400 {키 페어}
</pre>
> 윈도우: [속성] - [보안] - [고급] - [상속 사용 안 함] - [명시적 사용] - [관리자 제외하고 모든 그룹 삭제] - [모든 그룹에 대해서 '읽기 및 실행' 권한만 사용] - [관리자 권한으로 CMD 실행]
3. SSH 명령어로 서버 접속
<pre>
ssh -i "{키 페어}" {AWS 인스턴스 URL}
</pre>
## [부록] Jupyter Notebook 이용 방법
* 별도의 개발 리눅스 서버가 존재하는 경우 Jupyter Notebook 이용 가능. 실 서버에는 보안상 적용하지 않음.
* 파이썬, pip, jupyter 설치
```
sudo apt-get update
sudo apt-get install python3-pip
sudo pip3 install notebook
```
* jupyter 접속 비밀번호 설정
```
python3
>> from notebook.auth import passwd
>> passwd()
# 비밀번호 설정한 뒤에 SHA1 값 기록하기
```
* jupyter 환경 설정 파일 만들기
```
jupyter notebook --generate-config
sudo vi /home/ubuntu/.jupyter/jupyter_notebook_config.py
```
* jupyter 환경 설정하기
```
c = get_config()
c.NotebookApp.password = u'sha1:{해시 값}'
c.NotebookApp.ip = '{내부 아이피}'
c.NotebookApp.notebook_dir = '/'
# 내부 아이피로는 SSH로 접속했을 때 콘솔 창에 나오는 아이피를 입력하기
```
* jupyter 실행하기
```
sudo jupyter-notebook --allow-root
```
* [인바운드] - [편집] - [규칙 추가] - [사용자 지정 TCP] - [8888]번 포트 열기 - 허용 IP로 [0.0.0.0/0] 설정
* (http://{Host}:8888) 같은 형태로 주피터 노트북에 접속
* jupyter notebook 항상 실행 상태로 만들기
```
sudo jupyter-notebook --allow-root
# [Ctrl] + Z 입력하여 실행 종료하기
bg
disown -h
```
## [부록] Jupyter Notebook에 HTTPS 적용하기
* SSL 키 생성하기
```
cd /home/ubuntu
mkdir ssl
cd ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "cert.key" -out "cert.pem" -batch
```
* jupyter 환경 설정하기
```
sudo vi /home/ubuntu/.jupyter/jupyter_notebook_config.py
# 다음의 내용 입력하기
c.NotebookApp.certfile = u'/home/ubuntu/ssl/cert.pem'
c.NotebookApp.keyfile = u'/home/ubuntu/ssl/cert.key'
```
* (https://{Host}:8888) 같은 형태로 주피터 노트북에 접속
## [부록] Jupyter Notebook을 시스템 서비스로 등록하기
```
# 실행 중인 Jupyter 서버 종료하기
# jupyter-notebook 실행 파일 경로 찾기
which jupyter-notebook
# jupyter.service 작성하기
sudo vi /etc/systemd/system/jupyter.service
# Jupyter Notebook 서비스 작성하기
[Unit]
Description=Jupyter Notebook Server

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/sudo /usr/local/bin/jupyter-notebook --allow-root --config=/home/ubuntu/.jupyter/jupyter_notebook_config.py

[Install]
WantedBy=multi-user.target
# Jupyter 서비스 구동시키기
sudo systemctl daemon-reload
sudo systemctl enable jupyter
sudo systemctl start jupyter
# Jupyter 서비스 상태 확인하기
sudo systemctl status jupyter
# 오류 발생시 jupyter_notebook_config.py에서 IP 주소 확인하기
# Jupyter 서비스 재시작 방법
sudo systemctl restart jupyter
```
## [부록] Docker 설치하기 (Ubuntu 18.04)
```
# 시작하기 전에 볼륨 크기 넉넉하게 만들기
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce
# 설치 완료 후에 도커 상태 확인
sudo systemctl status docker
# Hello World 이미지 다운로드 및 실행
docker pull hello-world
docker images
docker run hello-world
docker ps -a
docker rm {Container ID}
# 모든 컨테이너 삭제
docker rm -f `docker ps -a -q`
# 모든 이미지 삭제
docker rmi -f `docker images`
# 실행 중인 도커 접속
docker attach {Container ID}
```
## [부록] Dockerfile 작성하기
```
# Dockerfile 만들기
mkdir /home/ubuntu/example
sudo vi /home/ubuntu/example/Dockerfile
# Dockerfile 작성하기
FROM ubuntu:18.04
MAINTAINER Dongbin Na <ndb796@naver.com>

RUN apt-get update
RUN apt-get install -y apache2 # Install Apache web server (Only 'Yes')

EXPOSE 80 # Open HTTP Port

CMD ["apachectl", "-D", "FOREGROUND"]
# Dockerfile 빌드하기
cd /home/ubuntu/example
docker build -t example .
docker images
docker run -p 80:80 example
* [인바운드] - [편집] - [규칙 추가] - [사용자 지정 TCP] - [80]번 포트 열기 - 허용 IP로 [0.0.0.0/0] 설정
* (http://{Host}:80) 같은 형태로 웹 사이트 접속
```
## [부록] PHP 추가하기
```
# Dockerfile 수정하기
sudo vi /home/ubuntu/example/Dockerfile
# Dockerfile 작성하기
FROM ubuntu:18.04
MAINTAINER Dongbin Na <ndb796@naver.com>

# Avoiding user interaction with tzdata
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get install -y apache2 # Install Apache web server (Only 'Yes')
RUN apt-get install -y software-properties-common # For Installing add-get-repository
RUN add-apt-repository -y ppa:ondrej/php # For Installing PHP 5.6
RUN apt-get update
RUN apt-get install -y php5.6 # Install PHP 5.6

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
# Dockerfile 빌드하기
cd /home/ubuntu/example
docker build -t example .
docker images
docker run -p 80:80 -v /home/ubuntu/example/html:/var/www/html example
# /home/ubuntu/example/html에 웹 문서 생성하기
cd /home/ubuntu/example/html
sudo vi index.php
# index.php 작업하기
<?php phpinfo(); ?>
* [인바운드] - [편집] - [규칙 추가] - [사용자 지정 TCP] - [80]번 포트 열기 - 허용 IP로 [0.0.0.0/0] 설정
* (http://{Host}:80) 같은 형태로 웹 사이트 접속
```
## [부록] MySQL 추가하기
```
# MySQL 설치 및 실행 (MySQL 5.6이 아니면 오류 발생할 수 있는 예제임)
docker run -d -p 9876:3306 -e MYSQL_ROOT_PASSWORD=password mysql:5.6
# Docker Container ID 확인
docker ps -a
# Docker MySQL 컨테이너 접속 및 MySQL 동작 확인
docker exec -it {Container ID} /bin/bash
mysql -u root -p
# 비밀번호: password
CREATE DATABASE TEST;
SHOW DATABASES;
# 접속 확인 및 [exit] 명령어로 나가기
# Docker Container IP 확인
docker inspect {Container ID}
# Docker MySQL 컨테이너 접속 및 MySQL 동작 확인
# 1) mysql -u root -p --host {Container IP} --port 3306
# 2) mysql -u root -p --host 127.0.0.1 --port 9876
# 비밀번호: password
SHOW DATABASES;
# 접속 확인 및 [exit] 명령어로 나가기
# 외부 접속을 위한 MySQL 계정 생성
docker exec -it {Container ID} /bin/bash
mysql -u root -p
# 비밀번호: password
use mysql;
CREATE USER 'test'@'%' IDENTIFIED BY 'password'; # DB 비밀번호 설정
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%'; # 외부 접속이 가능하도록 설정
FLUSH PRIVILEGES;
# 권한 설정 및 [exit] 명령어로 나가기
docker restart {Container ID}
* [인바운드] - [편집] - [규칙 추가] - [사용자 지정 TCP] - [9876]번 포트 열기 - 허용 IP로 [0.0.0.0/0] 설정
* 외부에서 MySQL 접속 도구를 이용해 [test] 계정으로 [9876] 포트에 접속하는 경우 정상적으로 접속 성공 (MySQL Workbench의 경우 오류 발생 가능)
```
## [부록] PHP MySQL 연동하기
```
# Dockerfile 수정하기
sudo vi /home/ubuntu/example/Dockerfile
# Dockerfile 작성하기
FROM ubuntu:18.04
MAINTAINER Dongbin Na <ndb796@naver.com>

# Avoiding user interaction with tzdata
ENV DEBIAN_FRONTEND=noninteractive

# Install Apache Web Server
RUN apt-get update
RUN apt-get install -y apache2 # Install Apache web server (Only 'Yes')

# Install PHP 5.6
RUN apt-get install -y software-properties-common # For Installing add-get-repository
RUN add-apt-repository -y ppa:ondrej/php # For Installing PHP 5.6
RUN apt-get update
RUN apt-get install -y php5.6 # Install PHP 5.6

# Connect PHP & MySQL
RUN apt-get install -y php5.6-mysql

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
# Dockerfile 빌드하기
cd /home/ubuntu/example
docker build -t example .
docker images
docker run -p 80:80 -v /home/ubuntu/example/html:/var/www/html example
# /home/ubuntu/example/html에 웹 문서 생성하기
cd /home/ubuntu/example/html
sudo vi index.php
# index.php 작업하기
<?php
$conn = mysqli_connect(
  '{데이터베이스 IP}',
  'test',
  'password',
  'TEST',
  '9876');
if (mysqli_connect_errno())
{
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
}
$sql = "SELECT VERSION()";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);
print_r($row["VERSION()"]);
?>
# 웹 사이트 접속 및 결과 확인시 MySQL 버전인 5.6 출력
```
## [부록] AWS RDS를 이용한 데이터베이스 구축
* [AWS RDS](https://ap-northeast-2.console.aws.amazon.com/rds) 서비스 페이지로 이동하기
* [파라미터 그룹] - [파라미터 그룹 생성] - MySQL 5.6 선택 이후에 [그룹 이름] 및 [설명]으로 'Hangul-Parameter' 넣기
* 이후에 [파라미터 편집] - 'char' 검색하여 전부 'utf8'으로 변경 - 'collation' 검색하여 전부 'utf8_general_ci'로 변경
* [데이터베이스 생성] - [MySQL] - 프리티어로 사용
* 인스턴스 식별자로 'docker-mysql' 넣기 - 마스터 사용자 이름으로 user 넣기 - 암호 입력하기
* [퍼블릭 액세스 가능성]을 '예'로 설정 - 데이터베이스 이름은 'TEST'로 생성 - [DB 파라미터 그룹]으로 아까 만든 'Hangul-Parameter' 설정 - [데이터베이스 생성]
* 생성 완료 이후에 [보안 그룹]에서 누구나 접속할 수 있도록 소스(Source)를 0.0.0.0/0로 넣기
```
# 만들어진 AWS RDS를 테스트하기 위한 소스코드 수정
sudo vi index.php
# index.php 작업하기
<?php
$conn = mysqli_connect(
  '{Host 엔드 포인트 주소}',
  'user',
  '{DB 비밀번호}',
  'TEST',
  '3306');
if (mysqli_connect_errno())
{
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
}
$sql = "SELECT VERSION()";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);
print_r($row["VERSION()"]);
?>
# 웹 사이트 접속 및 결과 확인시 MySQL 버전인 5.6 출력
```
## [부록] GitHub에 소스코드 올리기
* [깃 허브](https://github.com/) 접속하기
* [New Repository] - Repository Name으로 'Docker-PHP' 넣기 - [Private]으로 생성
```
cd C:\Docker Study
git clone https://github.com/ndb796/Docker-PHP.git
# 이후에 'Docker-PHP' 폴더에 Dockerfile 생성하기
# Dockerfile 수정하기
sudo vi /home/ubuntu/example/Dockerfile
# Dockerfile 작성하기
FROM ubuntu:18.04
MAINTAINER Dongbin Na <ndb796@naver.com>

# Avoiding user interaction with tzdata
ENV DEBIAN_FRONTEND=noninteractive

# Install Apache Web Server
RUN apt-get update
RUN apt-get install -y apache2 # Install Apache web server (Only 'Yes')

# Install PHP 5.6
RUN apt-get install -y software-properties-common # For Installing add-get-repository
RUN add-apt-repository -y ppa:ondrej/php # For Installing PHP 5.6
RUN apt-get update
RUN apt-get install -y php5.6 # Install PHP 5.6

# Connect PHP & MySQL
RUN apt-get install -y php5.6-mysql

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
# 이후에 'Docker-PHP' 폴더에 Project 폴더 생성하기
# Project 폴더에 index.php 생성하기
# index.php 작업하기
<?php
$conn = mysqli_connect(
  '{Host 엔드 포인트 주소}',
  'user',
  '{DB 비밀번호}',
  'TEST',
  '3306');
if (mysqli_connect_errno())
{
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
}
$sql = "SELECT VERSION()";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);
print_r($row["VERSION()"]);
?>
# 작성한 소스코드를 깃에 올리기
cd C:\Docker Study\Docker-PHP
git add .
git commit -m "초기 프로젝트 구성"
git push
# 이후에 자신의 Private Repository에서 소스코드 확인
```
## [부록] DockerHub와 GitHub 연동하기
* [도커 허브](https://hub.docker.com/) 접속하기
* [회원가입] - [로그인] - [Create Repository]
* [Name]으로 'docker-php' - [Visibility]로 Private 선택 - [Build Settings]에서 'GitHub' 선택 이후에 'Docker-PHP' Repository 선택 - [Create & Build]
* DockerHub와 GitHub가 연동되었으므로 이제부터 프로젝트 소스코드가 변경되어 Git Push가 발생하면 자동으로 Docker Build 진행
## [부록] GitHub에 README 추가하기
* 자신의 Private GitHub Repository로 이동 - [Add a README]
```
## Docker PHP
### Installation
<pre>
cd /home
git clone https://github.com/ndb796/Docker-PHP
cd Docker-PHP
</pre>
### Run
<pre>
# Login For Private Docker Repository
docker login
docker pull ndb796/docker-php
docker run -p 80:80 -v /home/Docker-PHP/Project:/var/www/html ndb796/docker-php
</pre>
```
* 실제로 해당 README 내용에 따라서 완성된 Docker 프로젝트 구동시키기
## [실습] Jenkins 설치
```
docker pull jenkins
docker run -d -p 8080:8080 -v /home/jenkins:/var/jenkins_home -u root jenkins
# 실행 중인 Jenkins 확인
docker ps -a
# Jenkins 로그를 열어 비밀번호 확인하기
docker logs {Container ID}
# [8080]번 포트 개방 및 8080 포트로 서버 접속하기
# 비밀번호 입력 후 Jenkins 설치 진행 - [Install suggested plugins]을 눌러 권장 설치 - 설치 이후 관리자 계정 설정하기
```
## [실습] Jenkins 관리 기본
* Jenkins 초기화: /home/jenkins 폴더 삭제 이후 jenkins 컨테이너 재시작
* 2-1) Jenkins 비밀번호 분실시: /home/jenkins/config.xml 파일에서 <useSecurity>false</useSecurity>로 변경 - 젠킨스 재시작(도커 컨테이너 지우고 재시작)
* 2-2) [Jenkins 관리] - [Configure Global Security] - [Enable security] 체크 - [Jenkins' own user database] - [사용자의 가입 허용] 체크 - [Save] - 이제 [Manager Users] 사용 가능
* 2-3) [Jenkins 관리] - [Manage Users] - [관리자 계정 설정] - 비밀번호 변경 - 젠킨스 재시작(도커 컨테이너 지우고 재시작)
* 2-4) 관리자 계정(admin)으로 로그인 - [Jenkins 관리] - [Configure Global Security] - [Matrix-based security] - [Add user or group...] - 관리자 계정(admin) 입력 - 모든 권한에 체크 - [Save] - 설정
## [실습] Jenkins Docker 빌드
* [새 작업] - 이름: [Example] - [Freestyle Proejct]로 생성 - [Build] 탭 - [Execute Shell] - Command 입력
```
docker pull ndb796/docker-php
docker run ndb796/docker-php
```
* [저장] - [Build Now] - 새로고침 이후 Build History 확인 - [Console Output] - 결과 
> https://stackoverflow.com/questions/44850565/docker-not-found-when-building-docker-image-using-docker-jenkins-container-pipel
