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
# 내부 아이피로는 AWS에 SSH로 접속했을 때 콘솔 창에 나오는 아이피를 입력하기
# 혹은 사설 Ubuntu 서버를 이용하는 경우 0.0.0.0으로 입력하기 (또한 추가적으로 sudo ufw allow 8888로 방화벽 허용 설정)
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
* Jenkins에서 PHP를 위한 Docker를 구동시키고자 함.
* 이러한 구조를 Docker in Docker라고 함.
* Jenkins에서 Docker를 이용하기 위해서는 Jenkins 컨테이너에 Docker가 설치되어 있어야 하고, 호스트와 docker.sock을 공유해야 함.
* 또한 Jenkins에서 PHP Docker 이미지를 실행하므로 80번 포트도 미리 열어 주어야 할 것 같지만, Jenkins에서 PHP Docker를 실행할 때 열어주면 됨.
```
docker pull jenkins
docker run -d -p 8080:8080 -v /home/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -u root jenkins
# 실행 중인 Jenkins 확인
docker ps -a
# Jenkins 로그를 열어 비밀번호 확인하기
docker logs {Container ID}
# [8080]번 포트 개방 및 8080 포트로 서버 접속하기
# 비밀번호 입력 후 Jenkins 설치 진행 - [Install suggested plugins]을 눌러 권장 설치 - 설치 이후 관리자 계정 설정하기
```
## [실습] Jenkins 관리 기본 - 부록
* Jenkins 초기화: /home/jenkins 폴더 삭제 이후 jenkins 컨테이너 재시작
* 2-1) Jenkins 비밀번호 분실시: /home/jenkins/config.xml 파일에서 <useSecurity>false</useSecurity>로 변경 - 젠킨스 재시작(도커 컨테이너 지우고 재시작)
* 2-2) [Jenkins 관리] - [Configure Global Security] - [Enable security] 체크 - [Jenkins' own user database] - [사용자의 가입 허용] 체크 - [Save] - 이제 [Manager Users] 사용 가능
* 2-3) [Jenkins 관리] - [Manage Users] - [관리자 계정 설정] - 비밀번호 변경 - 젠킨스 재시작(도커 컨테이너 지우고 재시작)
* 2-4) 관리자 계정(admin)으로 로그인 - [Jenkins 관리] - [Configure Global Security] - [Matrix-based security] - [Add user or group...] - 관리자 계정(admin) 입력 - 모든 권한에 체크 - [Save] - 설정
## [실습] Jenkins Docker 빌드
* [새 작업] - 이름: [Example] - [Freestyle Proejct]로 생성 - [Build] 탭 - [Execute Shell] - Command 입력
```
docker pull ndb796/docker-php
docker run -p 80:80 -v /home/Docker-PHP/Project:/var/www/html ndb796/docker-php
```
* [저장] - [Build Now] - 새로고침 이후 Build History 확인 - [Console Output] - 결과 
* Docker가 설치되어 있지 않아서 Jenkins 컨테이너에서 Docker를 사용할 수 없음
```
# 실행 중인 Jenkins 확인
docker ps -a
# 컨테이너에 접속하기
docker exec -it {Container ID} /bin/bash
# 컨테이너 내부에 Docker 설치하기
curl -fsSLO https://get.docker.com/builds/Linux/x86_64/docker-17.04.0-ce.tgz
tar xzvf docker-17.04.0-ce.tgz
mv docker/docker /usr/local/bin
rm -r docker docker-17.04.0-ce.tgz
# Docker의 원활한 사용을 위해 로그인하기
docker login
# Git을 통해 소스코드 다운로드하기
cd /home
git clone https://github.com/ndb796/Docker-PHP
# 이후에 컨테이너에서 나오기
exit
```
* [Example] 작업 - 다시 [Build Now] 누르기 - 새로고침 이후 Build History 확인 - [Console Output] - 결과
* 콘솔 출력이 종료되지 않고 계속 실행 중이라면 서버가 정상적으로 구동 중인 것임.
* Docker Container 명단에 PHP 컨테이너가 추가되어 구동 됨. 이는 Jenkins Container가 생성한 것임.
* 혹여 Build 내용을 수정하고자 한다면 [X] 버튼을 눌러서 구동을 종료시킨 뒤에 [구성]에서 Build 정보 변경.
* 결과적으로 웹 서버에 접속해서 결과 확인 가능.
## [실습] Jenkins 원격으로 빌드 유발하기
* 기존에 존재하는 Jenkins 실행 상태 [X] 눌러 종료 - PHP Docker 컨테이너 종료.
* [구성] - [Build] - Command 작성하기
```
cd /home/Docker-PHP
git pull
docker rm -f php || true
docker pull ndb796/docker-php
docker run -p 80:80 -v /home/Docker-PHP/Project:/var/www/html --name php ndb796/docker-php
```
* [빌드 유발] - [빌드를 원격으로 유발] - 토큰: [rebuild_token] - [저장]
* 이제 Git Pull 명령을 위해 Jenkins와 Git 연동.
```
1) [GitHub] 접속 - [Settings] - [Developer settings] - [Personal access tokens] - [Generate new token]
2) 토큰 설명: [jenkins-token] - [repo] 체크 - [admin:repo_hook] 체크 - 생성 - 토큰 복사
3) Jenkins 컨테이너의 bash 실행.
cd /home/Docker-PHP
# GitHub 로그인 정보 저장 만료 시간을 2시간으로 설정
git config --global credential.helper "cache --timeout 7200"
# GitHub 계정 입력 및 패스워드로는 액세스 토큰을 입력해서 로그인
git pull
# 이후에 한 번 더 Git 명령을 사용할 때는 2시간 동안 재로그인 할 필요 없음
git pull
```
* 브라우저에서 http://{IP 주소}:8080/job/Example/build?token=rebuild_token 경로에 접속.
* 원격으로 빌드가 유발되는 것 확인.
## Kubernetes 설치 ①
```
# 신뢰할 수 있는 APT 키 추가
sudo apt -y install apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
# Kubernetes 설치.
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
sudo apt install kubelet kubeadm kubectl kubernetes-cni
# 이 때 /etc/apt/sources.list가 임의로 수정되어 있으면 설치가 불가능할 수 있음
```
## [부록] apt-get 리스트 초기화 방법
```
# sources.list 파일 찾기
sudo dpkg -S apt | grep sources.list
# /usr/share/doc/apt/examples/sources.list 위치에서 샘플 발견 (패키지 서버 주소가 적을 수 있음)
cat /usr/share/doc/apt/examples/sources.list
# 전체 내용 붙여 넣은 뒤에 update 구문 실행 후에 오류 없음
sudo vi /etc/apt/sources.list
sudo apt-get update
```
## Kubernetes 설치 ②: 추천
```
# 가장 먼저 쿠버네티스의 정상적인 동작을 위해 CPU 및 RAM의 성능을 높일 필요가 있음.
# 인스턴스를 중지시킨 이후에 t2.medium으로 성능을 높인 뒤에 다시 시작.
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
## Minikube 설치 및 실행 ①
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube 
sudo mv minikube /usr/local/bin/
minikube version
# Minikube 실행
sudo -i
minikube start --vm-driver=none
# minikube start 과정에서 CPU 개수가 부족하다는 오류가 출력되면 다음과 같이 처리.
kubeadm init --pod-network-cidr=10.244.0.0/16  --ignore-preflight-errors=NumCPU flag
minikube start --vm-driver=none
```
## Minikube 설치 및 실행 ②: 추천
```
# Minikube 설치하기
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
minikube version
# Minikube 시작하기
sudo -i
minikube start --vm-driver=none
# 시작하기 위해 5분 이상의 시간이 필요.
minikube status
# 예제 컨테이너 구동하기
kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
kubectl expose deployment hello-minikube --type=NodePort
# 구동된 서비스 확인하기
kubectl get services
# 인바운드 포트 열어서 브라우저에서 접속하기
# 접속 결과 확인하기
# 서비스 죽이기
kubectl delete services hello-minikube
kubectl get services
# 컨테이너 죽이기
kubectl delete deployment hello-minikube
# 미니쿠베 죽이기
minikube stop
# 약 1분 뒤에 미니쿠베 동작 종료.
```
## Minikube Node.js 예제
```
minikube start --vm-driver=none
# Hello Node 예제를 기반으로 하는 컨테이너 실행.
kubectl create deployment hello-node --image=gcr.io/hello-minikube-zero-install/hello-node
# 인터넷에 파드 노출시키기.
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
# 생성된 서비스 확인하기.
kubectl get services
# 인바운드 설정 및 접속 테스트
# 클러스터 리소스 제거.
kubectl delete service hello-node
kubectl delete deployment hello-node
# 필요시 가상 머신 정지 및 삭제
minikube stop
minikube delete
```
## 되는 예제부터
```
https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/
```
