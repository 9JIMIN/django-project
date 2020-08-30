# django - blog book

## part 1 - aws

### 계정생성

- 주소도 넣어야 하더라, 진주 주소로 했다. 영어주소로 넣어야하는데, 바꿔주는 사이트를 이용하면 된다.
- 결제정보인 카드번호를 넣는데, 바로 1200원이 빠져나간다. (인증비용)
- 전화번호 인증까지 한다.
- 그리고 콘솔에 로그인을 한다. (루트 사용자)

### EC2 instance 생성

- AMI 선택(우분투)
- 인스턴스 유형 선택(프리티어 micro)
- 인스턴스 세부정보, 스토리지, 태그, 보안그룹은 기본값 그대로 해서 생성.

- 키 페어 생성.
- 인스턴스 생성 완료!

### EC2 instance 연결

- 키 파일의 권한을 사용자만 읽기가능 하도록(400) 으로 바꿔준다.

```shell
$ chmod 400 aws-key.pem 
# chmod 400 [키 파일경로]
```

- 그리고 접속한다.
- 리눅스는 ssh가 기본으로 깔려있어서 아래 명령어로 가능하지만, 
  윈도우는 putty로 접속해야함.

```shell
$ ssh -i aws-key.pem ubuntu@ec2-3-129-66-159.us-east-2.compute.amazonaws.com
# ssh -i [키 파일경로] [사용자명@서버주소]
# 사용자명은 ubuntu, 서버주소는 인스턴스의 퍼블릭 DNS를 입력
```

- 질문에 yes를 하면 아래와 같이 연결이 된다.

```shell
jimin@fossa:~/Desktop$ chmod 400 aws-key.pem
jimin@fossa:~/Desktop$ ssh -i aws-key.pem ubuntu@ec2-3-129-66-159.us-east-2.compute.amazonaws.com
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 5.3.0-1032-aws x86_64)
...

ubuntu@ip-172-31-23-132:~$ 
# 연결 성공!
```

### 파일전송을 위한 연결

- SCP (Secure Copy Protocol) 를 이용하여 연결한다.

```shell
#### - 파일 올리는 법
$ scp -i aws-key.pem test.txt ubuntu@ec2-3-129-66-159.us-east-2.compute.amazonaws.com:~
# scp -i [키 파일경로] [전송파일경로] [사용자명@서버주소]:[대상 디렉토리]
# :~ 는 루트 디렉토리
ubuntu@ip-172-31-23-132:~$ ls
test.txt # 이렇게 확인가능



#### - 파일 받는 법
$ scp -i aws-key.pem ubuntu@ec2-3-129-66-159.us-east-2.compute.amazonaws.com:~/test.txt test3.txt
# scp -i [키 파일경로] [사용자명@서버주소]:[대상파일경로] [호스트파일경로]
```

### RDS 생성

- Relational Database Service 관계형 데이터베이스
- 서비스에서 '데이터베이스 - RDS'를 선택
- 서울 리전으로 하고 생성
- 엔진선택 - Aurora를 하면 좋지만, 프리티어는 사용불가. MySQL을 선택한다.
- '템플릿' - 프리티어, 인스턴스 식별자, 마스터 암호 설정
- 퍼블릭 엑세스 가능성 - 예
- 생성은 5분쯤 걸림.

### RDS - MySQL 연결하기

- 내 컴퓨터에서 aws의 rds를 만질 수 있다.

- DB엔진에 따라서 연결방법이 다 다름. 여기서는 MySQL

- MySQL workbench 다운 (스토어에 있음)

- 워크벤치 - new connection

  ```shell
  - Hostname: test-instance.cco2lud6qune.ap-northeast-2.rds.amazonaws.com # 데이터베이스 - 엔드포인트
  - 유저, 비번 넣으면 연결 완료.
  - 근데 난 왜 안되냐....
  ```
  


### Elastic beanstalk 생성

- application정보, 플랫폼 선택, 샘플앱
- 몇분후,.. 배포가 끝나면 웹에 접속할 수 있음.