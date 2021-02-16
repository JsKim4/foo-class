# ==================== jenkins-instance ==============================
# install cli
sudo systemctl yum install wget
sudo systemctl yum install maven
sudo systemctl yum install git
sudo systemctl yum install docker
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

# jenkins 최초 진입시 initAdminPassword 입력이 필요한데
# 아래 명령어로 확인할 수 있다.

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# 계정 생성 후 Publish Over SSH 설치
# ssh 키 생성
sudo ssh-keygen -t rsa -f ~/.ssh/id_rsa

# 아래 명령어로 ssh 개인키를 확인 할 수있다.
sudo cat ~/.ssh/id_rsa

# 아래 명령어로 ssh 공개키를 확인할 수 있다.
sudo cat ~/.ssh/id_rsa.pub

# 공개키는 GCP의 메타데이터 => SSH 키에 저장하면 이후 GCP에서 ~/.ssh/authorized_keys 에 자동으로 등록하여 생성해준다.
# ( 자동으로 삭제도 되기 때문에 GCP에 등록해서 사용하는게 정신건강에 이로움)

# 젠킨스 설정에서 Publish over SSH에 위에서 만든 개인키를 등록한다.
# SSH Servers에 해당 개인키를 복호화 할수 있는 서버를 적어준다. (공개키를 GCP에 등록했기 때문에 GCP에서 생성한 인스턴스들은 모두 복호화 가능함)
# ----- SSH Servers 각 항목의 설명 ------
# Name : 배포시에 Publish On SSH 에서 사용되는 이름
# Hostname : 복호화하는 호스트 주소
# username : 호스트에서 공개키에 접근 가능한 유저 이름
# Remote Directory : 원경 폴더 위치 user가 접근 가능한 디렉토리

# 젠킨스 실행 후 post build step(빌드 후 조치) => Send build artifacts over SSH 추가
# Name 위에서 등록한 SSH Servers의 이름
# Exec Command 스크립트 작성 => 
docker run -p 8080:80 jskim4/spring-boot-cpu-bound
# docker 오류 발생시 => (cpu-instance)도커 명령어를 못찾는 경우 도커 설치 확인 및 실행
# ------ docker permission 관련 오류(cpu-instance 명령어) ------
# 영상에서 설명하는 해결법 =>  
sudo chmod 666 /var/run/docker.sock
# 영상보기전에 내가 해결한 해결법 =>
sudo groupadd docker
sudo usermod -a -G docker $USER
sudo systemctl restart docker
# 잘 성공했으면 명령어 변경 => 
nohup docker run -p 8080:80 jskim4/spring-boot-cpu-bound > nohup.out 2>&1 &