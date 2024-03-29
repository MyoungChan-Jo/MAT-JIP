# MAT-JIP server
----------------------------------------------
서버
- aws 클라우드 서비스 접속
- 로그인 후 관리콘솔 접속
- 컴퓨팅 서비스 ec2 접근하여 서울선택
- 인스턴스에 접근하여 인스턴스 시작버튼 클릭
- os 선택에서 리눅스 우분투 18사용
- 프리티어 가능성능으로 모두 선택. 용량은 8기가로 충분
- 검토및 시작을 하면 키페어를 받게 된다. 이름을 선택하고 저장하면 된다.
- 인스턴스 시작을 누르면 서버컴퓨터를 대여받게 된다.
- 접근툴을 설치해야한다.
- winscp 와 putty 를 설치해야 한다.
- ctrl alt p 를 통해 설정으로 들어가 putty 를 연동해준다.
- 빌린 컴퓨터 주소는 인스턴스에서 퍼블릴 ipv4 주소에서 볼 수 있다.
- 해당 아이피는 변동값이기에 보안에서 탄력 ip 로 들어가 고정아이피로 만들어준다.
- 비밀번호는 putty 키파일을 선택해주면 된다.
- 접속 후 번개모양과 모니터 두개가 합쳐져있는 이미지인 putty 접속을 한다.

-관리자 권한 부여하기
1. sudo su 라고 입력하면 unbuntu 에서 root 으로 넘어가게 된다.
2. apt-get update
3. apt-get install eginx -y  (-y 는 설치과정중 물어보는 구문에 모두 yes 하겠다는 의미이다.)
그 뒤 aws 로 넘어가 보안의 인바운드 규칙에서 http 문을 열어준다.
4. apt-get install nodejs 
5. node -v 로 버젼 체크
6. node 업데이트를 위해 https://torbjorn.tistory.com/527 참고 https://velog.io/@moorekwon/MySQL-%EC%99%84%EC%A0%84-%EC%82%AD%EC%A0%9C 완전삭제
7. 설치 후 node -v , npm -v 를 통해 버전 체크 해보기
속성요약
sudo apt update -> 최신버전업데이트
sudo apt install nodejs -> nodejs 설치
sudo apt install npm -> npm설치
curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_14_setup.sh -> 스크립트 다운로드
sudo bash nodesource_setup.sh -> 다운로드한 스크립트 실행
sudo apt install nodejs -> 업데이트한 node14설치
sudo apt install build-essential -> 각종빌드툴

-----------------------------------------------

-mysql 설치
1. apt-get install mysql-server -y 로 설치

-mysql 접근
1. mysql -u root -p 를 통해 접근해보기
2. enter passowrd 에서 그냥 enter 눌러서 접근해본다.
3. https://indienote.tistory.com/585 해당본문 읽어보고 password 설정해보기
4. exit 으로 mysql 에서 나갈 수 있다.

로컴 컴퓨터에서 접근하기 위한 외부접속 허용방법
1. 모든 아이피를 허용해준다.
- grant all privileges on *.* to root@localhost; 혹은
- grant all privileges on *.* to 'root'@'%' identified by '비밀번호';
2. flush privileges; 로 저장해주기.
- sudo /etc/init.d/mysql restart 로 재시작해보기
- mysql -h "IP주소" -P 3306 -u root -p 로 테스트 해보기
3. listen ip 대역 변경해주기
- exit 으로 mysql 종료 후 폴더로 접근한다. vi /etc/mysql/mysql.conf.d/mysqld.cnf  (패키지관리자로 설치한 mysql 설정파일 경로)
4. bind-address = 127.0.0.1 을 0.0.0.0 으로 변경 맨앞에 i 누르고 # 누르면 주석처리됨 그 뒤에 esc 누르고 :wq!눌러서 나가기
5. service mysql restart 로 재시작한다.

workbench 외부접속 접근 
1. 클라이언트프로그램이 필요하다. mysql 에서 기본제공해주는 것이 mysql workbench 이다.
2. 운영체제 설정 후 다운로드 진행하면 워크밴치가 나오게 된다.
3. mysql 에 + 버튼을 누르면 setup new connection 이 나오게 되는데 여기서
- hostname 에는 aws 인스턴스 ip 주소 들어가고
- username 에는 만들어준 이름이 들어가는데 위에 grant 에서 root 으로 지정해 주었으니깐 root 을 쓰면 된다.
- password 도 넣어준뒤 testconnection 을 해준 뒤 포트를 다시 열어준다.
4. aws 콘솔 접근 후 인바운드규칙 편집으로 들어간다.
5. 규칙추가 버튼을 누른뒤 mysql/aurora 소스는 anyway v4 로 연결해준뒤 다시 testconnection을 눌러준다.

3306방화벽 해제방법
1. 방화벽에 mysql 포트 등록
firewall-cmd --zone=public --add-port=3306/tcp --permanent
2. 방화벽 리로드
firewall-cmd --reload
3. 방화벽 포트 리스트 확인
firewall-cmd --list-ports

4. 방화벽 상태 확인
systemctl status firewalld
-active 상태이면 workbrench 접속

5. mysql 밖으로 나와서
sudo ufw allow mysql 로 허용한다.
 
6. 다시 mysql 로 접근해서
- create user 계정아이디@'%' identified by '비번';
- grant all privileges on db이름.* to 계정아이디@'%' with grant option;
- flush privileges;

--------------------------------------------

workbench 부분

조회부분인 select 쿼리가 시간을 많이 잡아먹는다.
api 는 다른 사람이 미리 만들어 놓은 인터페이스 -> application porgramming interface
쿼리스트림
get 요청을 날릴땐 원하는 부분앞에 ? 를 붙인다. retaurants?category="한식"

패킷은 데이터 덩어리 이며 두 파트로 나뉜다.
회원가입을 예로 들면 헤더와 바디로 나뉘며 헤더에는 메타데이터가 들어가서 주소같은게 들어가고 바디에 회원정보가 들어간다.
form, xml, json 으로 구현하며 
form 은 html 에서 본 그거 그대로고
xml 같은 경우 <이름>홍길동</이름> 이런식으로 넘어간다.
json 은 객체형식으로 들어가서 { 이름:"홍길동", 나이:20 }

json 이 편하고 깔끔하기 떄문에 json 을 많이 쓴다.

차단하기 = 차단목록에 친구를 추가하는 생성행위이기 떄문에 post 가 자연스럽다.

node 의 모듈 시스템이란
js 파일간의 에드온을 사용가능하게 해준다.
exports 와 require 가 그 에드온이다.
exports.a = 1; 이것은 다른파일에 a 를 가져다 쓰도록 하겠다 라는 의미이다.
다른 파일에서
const modules1 = require("export한 파일 주소");
console.log(module1);
결과는 {a:1} 이 나오게 된다.
 혹은
 moudle.exports={a:1,}; 이런식으로도 사용한다.

 npm -> 노드 패키지 매니저 라는 의미이다.
 
