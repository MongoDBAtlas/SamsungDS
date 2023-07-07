<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Handson Workshop for Samsung DS

## Ops Manager Management
Ops Manager를 이용한 관리 방법을 실습합니다. MongoDB가 설치 운용이 될 Machine을 등록하고 실제 MongoDB를 배치 합니다.

### [&rarr; Agent Installation](#Agent)

### [&rarr; Deploy Replicaset](#Deploy)

### [&rarr; Enable Monitor](#Monitor)

### [&rarr; Database Account](#Account)


<br>


### Agent

Ops Manager는 설치형 MongoDB를 관리 하기 위한 솔루션으로 전반적인 모니터링 및 관리를 제공 합니다. Ops Manager는 별도의 머신에 설치가 필요 하며 데이터 저장을 위한 별도의 저장소 (MongoDB Replica Set)이 필요 합니다.  
설치 자료는 다음에서 확인 할 수 있습니다.   

https://www.mongodb.com/docs/ops-manager/current/installation/

실습과정에서는 사전에 설치된 Ops Manager를 사용합니다.
Ops Manager에 개인 계정으로 로그인하고 프로젝트로 이동하여 줍니다. 

<img src="/04.operation/images/image01.png" width="70%" height="70%">     


데이터 베이스 배포 버튼을 클릭하면 Agent 설치 정보를 볼 수 있습니다. Redhat Linux와 호환 (Amazon linux2)으로 이를 선택 합니다.   

<img src="/04.operation/images/image02.png" width="80%" height="80%">     

Agent가 설치될 머신의 OS 정보에 맞추어 다운로드 할 수 있습니다. 

<img src="/04.operation/images/image03.png" width="30%" height="30%">     


이후 설치를 위한 정보가 표시 되며 이를 순서대로 실행 하여 줍니다.  

<img src="/04.operation/images/image04.png" width="80%" height="80%">     


다음과 같이 설치를 진행 합니다.  
````
[# curl -OL http://ops.nosql.site:8080/download/agent/automation/mongodb-mms-automation-agent-manager-12.0.23.7711-1.x86_64.rhel7.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 18.5M    0 18.5M    0     0   110M      0 --:--:-- --:--:-- --:--:--  109M
# sudo rpm -U mongodb-mms-automation-agent-manager-12.0.23.7711-1.x86_64.rhel7.rpm

````

설치 후 Agent 의 Config 파일을 수정 하여 줍니다. GroupId와 API Key, Opsmanager의 Url을 등록 하여 줍니다.   

````
# sudo vi /etc/mongodb-mms/automation-agent.config

$ vi

#
# REQUIRED
#
# Enter your Project ID - It can be found in the Ops Manager UI under Settings ->
# Group Settings.
#
mmsGroupId=64a7d04************


#
# REQUIRED
#
# Enter your API key - It can be found in the Ops Manager UI under Settings ->
# Group Settings.
#
mmsApiKey=64a80cef02cd0d33f016ef**********

#
# Hostname of the Ops Manager web server. The hostname will match what is used
# to access the Ops Manager UI. The default port for an Ops Manager install
# is 8080.
#
# ex. http://opsmanaager.<company>.com:8080
#mmsBaseUrl=
mmsBaseUrl=http://ops.nosql.site:8080

#
# Path to log file
#
logFile=/var/log/mongodb-mms-automation/automation-agent.log

....

````

Agent 는 mongod 계정으로 구동이 됩니다. MongoDB 또한 해당 계정으로 설치 구동 됨으로 설치될 폴더의 소유권을 설정 해 주어야 합니다. Handson 에서는 /data 폴더에 데이터베이스 파일을 구성 합니다. 이후 자동으로 agent를 구동하기 위해 서비스로 등록 하여 줍니다.  
````
# chown mongod:mongod /data
# sudo systemctl start mongodb-mms-automation-agent.service
# sudo /sbin/service mongodb-mms-automation-agent start
Redirecting to /bin/systemctl start mongodb-mms-automation-agent.service

````

Agent 가 설치가 완료 되었음으로 실제 Cluster를 배포 합니다.   


### Deploy
Agent 가 설치가 되어 있음으로 Build New 를 클릭 하고 Replica Set를 선택 하여 줍니다.

<img src="/04.operation/images/image05.png" width="70%" height="70%">     

배포 대상을 지정 하는 것으로 제한된 리소스로 인해 한대의 머신에 Replica Set를 설치 하도록 합니다. 

<img src="/04.operation/images/image06.png" width="80%" height="80%">     

한대의 머신임으로 host 정보는 지정된 머신을 선택 하고 3개의 인스턴스가 구동될 포트 정보를 겹치지 않게 27001,27002,27003으로 부여 하고 데이터 베이스 폴더 또한 구분이 되도록 입력 하여 줍니다.

Create Replica Set 버튼을 클릭 하면 데이터 베이스가 배포 됩니다.

실제 배포를 위해서 Review and Deploy 버튼을 클릭 하면 실제 배포가 진행 됩니다.   

<img src="/04.operation/images/image07.png" width="90%" height="90%">     

배포는 수분의 시간이 소요 되며 다음과 같이 설치 상태가 초록색으로 보여 지게 됩니다.    

<img src="/04.operation/images/image08.png" width="90%" height="90%">     


#### Monitor

최초 클러스터가 배포되면 monitoring 은 enable 되어 있지 않습니다. enable 하기 위해서는 클러스터 정보에 Servers를 선택 하면 다음과 같은 화면을 볼 수 있습니다.   

Monitoring을 enable 하기 위해 activate를 선택 합니다.

<img src="/04.operation/images/image09.png" width="70%" height="70%">     

변경 사항 적용을 위해 Review & deploy 버튼을 클릭 하여 줍니다.   


#### Account

데이터베이스 사용자를 생성 하여 주는 과정입니다. 기존 설치형의 경우 db.createUser를 이용하여 Account를 생성하지만 관리를 위하여 Console에서 사용자를 생성 하게 됩니다.   

데이터베이스 계정 로그인 방법을 선택 하기 위해 Cluster 메뉴 중 Security를 클릭 하고 Settings를 선택 합니다.    
계정과 비밀번호를 이용한 로그인 방법을 선택 하고 저장 합니다.  

<img src="/04.operation/images/image13.png" width="80%" height="80%">     


Cluster 메뉴에 Security를 클릭 합니다.    

<img src="/04.operation/images/image11.png" width="80%" height="80%">     

Add New user를 클릭 합니다.   
사용자 계정과 권한 (편의상 모든 데이터베이스에 읽기 쓰기가 가능한 권한을 부여 하여 줍니다.)을 선택 하고 비밀번호를 입력 하여 줍니다. 방식은 SCRAM-SHA-256을 선택합니다.

<img src="/04.operation/images/image12.png" width="60%" height="60%">     

