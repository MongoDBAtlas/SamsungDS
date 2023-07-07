<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Handson Workshop for Samsung DS

### Installation


#### MongoDB Binary
MongoDB 설치는 다양한 옵션으로 가능하며 binary는 아래 사이트에서 다운로드 할 수 있습니다.    
https://www.mongodb.com/try/download/enterprise

인터넷이 연결된 상태인 경우 Yum을 이용한 설치가 가능 합니다.  

다음은 Yum을 이용한 설치 과정 입니다.    

`````
$ sudo vi /etc/yum.repos.d/mongodb-enterprise-6.0.repo
[mongodb-enterprise-6.0]
name=MongoDB Enterprise Repository
baseurl=https://repo.mongodb.com/yum/redhat/$releasever/mongodb-enterprise/6.0/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc

$ sudo yum install -y mongodb-enterprise

`````

#### Config 파일 생성
데이터 베이스는 Config 파일과 함께 구동이 됩니다. Config 파일에 데이터베이스 구성 정보를 기술 합니다.   
구성정보는 데이터베이스 파일이 생성될 경로와 로그 파일, 서비스될 포트 정보입니다.

`````
$ vi mongod.config
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
storage:
   dbPath: /data/mongod/
   journal:
      enabled: true
processManagement:
   fork: true
net:
   bindIp: 127.0.0.1
   port: 27017
replication:
  replSetName: mymdb
...

`````
주요한 Configuration 정보는 다음에서 얻을 수 있습니다.   
https://www.mongodb.com/docs/manual/reference/configuration-options/

데이터 베이스 시작은 다음 명령어로 시작 합니다.


`````
$ mongod -f mongod.config
..
`````
processManagement.fork 가 true 로 되어 있으면 백그라운드 프로세스로 구동이 됩니다. ReplicaSet은 기본 3개의 인스턴스로 구성됨으로 3개의 config 파일을 생성하고 mongod를 실행 하여 줍니다.   
초기에는 계정에 대한 정보가 없음으로 mongosh을 이용하여 3개의 인스턴스 중 한개에 접속 하여 줍니다.   
접속후 Replica Set을 구성하여 줍니다. (rs.initiate()로 초기화 하고 add()를 이용하여 replica set을 구성합니다)

`````
$ mongosh --port 27017
> rs.initiate()
> rs.add('<server address>:<port>')
> rs.add('<server address>:<port>')
> rs.status()
> db.createUser({
    user: "<<userid>>",
    pwd: "<<Password>>",
    roles: [
      {role: "root", db: "admin"}
    ]
  })

..
`````
생성 완료 후 계정을 이용한 접근만 허용하기 위해 다음과 같이 보안 설정을 하고 다시 구동 하여 줍니다.   

Replica Set 간에 네트워크 통신을 위한 암호 파일을 생성 하고 공유합니다. 생성한 파일을 3개의 인스턴스에 복사하여여 줍니다.   

`````
$ openssl rand -base64 741 > keyfile

$ chmod 400 <path-to-keyfile>

`````

Replica Set 간에 통신을 위한 암호 파일 경로와 접근을 제안 하는 옵션을 추가 하여 줍니다.

`````
$ vi mongod.config
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
storage:
   dbPath: /data/mongod/
   journal:
      enabled: true
processManagement:
   fork: true
net:
   bindIp: 127.0.0.1
   port: 27017
replication:
  replSetName: mymdb
security:
  authorization: enabled
  keyFile: /etc/keyfile

`````

#### 기타 필요한 소프트웨어
클라이언트 애플리케이션 테스트를 위한 Nodejs 필요합니다.
MongoDB에 접속하고 데이터를 조회 하는 GUI Tool (Compass)를 다운로드 합니다.

Nodejs : 
https://nodejs.org/en/download/

Compass :   
https://www.mongodb.com/products/compass

Mongosh :
https://www.mongodb.com/docs/mongodb-shell/install/

