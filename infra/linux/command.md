# 리눅스 명령어 모음
## ec2 인스턴스 접속

```bash
ssh -i key-does.pem ubuntu@15.164.211.129
```

## 자바 설치

```bash
 wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add -
 sudo add-apt-repository 'deb https://apt.corretto.aws stable main'

```

```bash
 sudo apt-get update; sudo apt-get install -y java-11-amazon-corretto-jdk

```

## mysql ip 주소로 접속

```bash
sudo mysql -h 192.168.0.216 -u does -p1234
```

## 배포용 application.properties

```bash
spring.datasource.initialization-mode=always

spring.datasource.url=jdbc:

mysql://192.168.0.216:3306/does_shopping?serverTimezone=UTC&characterEncoding=utf8

spring.datasource.username=does

spring.datasource.password=1234
```

## 파일 복사해서 옮기기

```bash
cp /home/ubuntu/application.properties /home/ubuntu/jwp-shopping-cart/src/main/resources/application.properties
```

## 실행 중인 포트 보기

```bash
ubuntu@ip-192-168-0-57:~$ lsof -i :8080
```

## 포트 죽이기

```bash
kill -9 33634
```

## 8080에서 실행되는 프로세스 종료시키기

```bash
sudo kill -9 $(sudo lsof -t -i :8080
```

## 빌드 및 배포

```bash
nohup java -jar jwp-shopping-cart-0.0.1-SNAPSHOT.jar &
```

```bash
echo ---------기존 서버 프로세스 종료---------

pid=$(pgrep -f smody)
if [ -n "${pid}" ]
then
        kill -9 ${pid}
        echo kill process ${pid}
else
        echo no process
fi

echo -----------기존 프로젝트 제거------------

cd /home/ubuntu/repository
rm -r -f 2022-smody

echo ---------—최신 프로젝트 클론—----------

git clone -b develop --single-branch https://github.com/woowacourse-teams/2022-smody.git

echo -------------설정 파일 세팅--------------

cd /home/ubuntu/repository/2022-smody/backend/smody/src/main/

mkdir resources

cd /home/ubuntu/production-config

cp application.properties /home/ubuntu/repository/2022-smody/backend/smody/src/main/resources

cd /home/ubuntu/test-config

cp application.properties /home/ubuntu/repository/2022-smody/backend/smody/src/test/resources

echo --------------bootJar 생성---------------

cd /home/ubuntu/repository/2022-smody/backend/smody
./gradlew bootJar

echo ---------------서버 실행-----------------

cd /home/ubuntu/repository/2022-smody/backend/smody/build/libs
chmod 755 smody-0.0.1-SNAPSHOT.jar
nohup java -jar -Duser.timezome=Aisa/Seoul smody-0.0.1-SNAPSHOT.jar &

echo ----------디렉토리 위치 초기화-----------

cd /home/ubuntu

echo ---------------배포 완료-----------------
```

## 로깅

쉘 스크립트 권한 주기: `chmod 755 {sh 파일 이름}`

```bash
echo ----------최신 20줄씩 로그 출력 시작----------

cd repository/2022-smody/backend/smody/build/libs

tail -20f nohup.out

cd /home/ubuntu
```

```bash
echo ----------모든 로그 출력----------

cd repository/2022-smody/backend/smody/build/libs

cat nohup.out

cd /home/ubuntu
```

```bash
echo ----------예외 로그 보기----------

cd repository/2022-smody/backend/smody/build/libs

cat nohup.out | grep -A 20 'Exception'

cd /home/ubuntu
```

- ‘Exception’ 문자열 찾고 그 밑 20줄까지 출력

```bash
echo ---단어 개수 세기---
grep -o [단어] [파일명] | wc -w
```

## ec2로 파일 전송

```bash
scp -i key-smody-image.pem cat.jpg ubuntu@3.36.114.37:/home/ubuntu/images
```

chmod 755 smody-0.0.1-SNAPSHOT.jar
nohup java -jar -Dspring.profiles.active=dev -Duser.timezone=Asia/Seoul smody-0.0.1-SNAPSHOT.jar &

## WARNING: UNPROTECTED PRIVATE KEY FILE! (Window)

```java
icacls.exe myec2.pem /reset
icacls.exe myec2.pem /grant:r %username%:(R)
icacls.exe myec2.pem /inheritance:r
```
