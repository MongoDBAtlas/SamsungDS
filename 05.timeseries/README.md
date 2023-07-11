<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/>

# MongoDB Handson Workshop for Samsung DS

## Index and Aggreagation
Time Series Collection을 정의 하고 데이터를 저장 하여 봅니다.

### [&rarr; Define Timeseries Collection](#Timeseries)

### [&rarr; Insert Data into Timeseries](#Insert)

### [&rarr; Data Find](#Find)


<br>

### Timeseries
Time Series는 시계열성 데이터를 다루기 위한 것으로 IoT성격의 장비/설비로 부터 발생되는 데이터를 다루기 위한 것으로 자주 사용 됩니다. 시계열 데이터를 다루기 때문에 데이터가 생성되는 Time stamp가 반드시 필요하며 장비/설비에 대한 ID와 해당 시점에서 측정 (measure)되는 데이터로 구성이 됩니다.   
데이터 사용은 주로 시간을 기준으로 하여 데이터를 조회 하게됩니다. 시계열을 기준으로 데이터가 정렬 되어 있음으로 시간 범위를 지정하는 경우  정렬된 데이터를 읽기 때문에 속도가 빠른 편입니다.   

컬렉션을 생성 하기 위해새 IoT로 부터 생성되는 데이터를 먼저 정의해야 합니다. 
다음과 같이 time stamp 데이터를 저장하고 있는 필드 (timestamp)와 데이터를 생산하는 주체로 설비 (equip)과 센서 (sensor)를 정의 합니다. 측정되는 데이터 값은 온도, 습도, 압력 등이 측정 된다고 가정 합니다. 생성되는 데이터 샘플은 다음과 같습니다.  

````
{
    timestamp: ISODate("2023-06-15T06:00:00.000Z"),
    metadata:{
        equip_id: "DS001",
        sensor_id: "SHISENSOR1",
    },
    temperature: 30,
    humidity: 40,
    pressure: 100
}
````
timestamp, metadata를 제외한 필드는 측정 필드 (measure)가 됩니다.   

Time Series를 위한 컬렉션을 정의 합니다. 일반 MongoDB 컬렉션은 별도의 정의 없이 사용이 가능하지만 Time Series의 경우 특수한 형태의 컬렉션으로 사전에 컬렉션을 정의 해주는 것이 필요 합니다.   

````
db.createCollection(
    <<COLLECTION NAME>>,
    {
       timeseries: {
          timeField: <<TIME STAMP FIELD>>,
          metaField: <<META DATA FIELD>>,
          granularity: <<granularity>>
       }
    }
)
````
사전에 정의한 데이터로 컬렉션을 정의하면 다음과 같습니다.

````
mdb1 [primary] ds> db.createCollection("dsedip", {timeseries:{timeField:"timestamp",metaField:"metadata",granularity:"seconds"}} )
{ ok: 1 }
````
컬렉션 이름은 shipTime 이며 시간 데이터가 들어가는 필드(timestamp), 메타데이터가 들어가는 필드의 명 (metadata)를 입력 하여 줍니다.    
granularity 는 시계열의 발생 주기로 초단위로 발생하는 데이터의 경우 "seconds"를 입력 하고 분단위로 발생 하는 경우 "minutes" 을 지정하고 시간단위로 발생 하면 "hours"를 입력 하여 줍니다.   
timeField, metaField, granularity의 데이터는 String 타입만을 가져 갈 수 있습니다. (배열로 여러개 값을 지정 할 수 없습니다.)   

### Insert
데이터를 생성 하여 입력 하도록 합니다. 예제로 생성한 것과 유사하게 데이터를 생성합니다. 다음과 같이 mongosh에서 MQL을 수행 하여 줍니다.

````
mdb1 [primary] ds> let i=0;

mdb1 [primary] ds> let temperature = 0;

mdb1 [primary] ds> let humidity=0;

mdb1 [primary] ds> let pressure=0;

mdb1 [primary] ds> for (i = 0; i < 60; i++) 
{
    temperature = Math.floor(Math.random() * 30);
    humidity = Math.floor(Math.random() * 80);
    pressure = Math.floor(Math.random() * 100);
    let timestamp="";
    if (i < 10) {timestamp = ISODate("2023-06-15T06:0" + i + ":00.000Z");} 
    else {timestamp = ISODate("2023-06-15T06:" + i + ":00.000Z");} 
    
    let insert = { timestamp: timestamp, metadata: { equip_id: "DS001", sensor_id: "SHISENSOR1" }, measure: { temperature: temperature, humidity: humidity, pressure: pressure } }; 
    db.dsedip.insertOne(insert); 
    db.dsedipNormal.insertOne(insert);
}
````

두개 컬렉션 dsedip과 비교를 위한 일반 컬렉션 dsedipNormal에 동일한 데이터가 생성 됩니다.

60개의 데이터가 생성이 됩니다.

다음은 24시간으로 하여 생성한 것으로 전체 데이터가 44,640건(한달 31일 * 하루 24시간 * 60분)이며 9개 Sensor 로 401,760 건의 데이터가 생성된 것입니다.   

<img src="/05.timeseries/images/image01.png" width="90%" height="90%">   

데이터 용량은 56.71MB인 것을 확인 할 수 있습니다.

다음은 timeseries 컬렉션에 동일한 것을 생성 한 것으로 용량이 1.8MB인것을 확인 할 수 있습니다.

<img src="/05.timeseries/images/image02.png" width="90%" height="90%">   

데이터 건수도 다음과 같이 동일 하게 401,760건이 생성된 것을 확인 할 수 있습니다.    

````
mdb1 [primary] ds> db.dsedip.find({}).count()
4320
mdb1 [primary] ds> 
````

### Find

검색을 위한 인덱스를 생성 합니다. 기존 dsedipNormal 의 경우 다음과 같이 인덱스를 생성 하여 줍니다. 

````
mdb1 [primary] ds> db.dsedipNormal.createIndex({"metadata.equip_id":1,"metadata.sensor_id":1,"timestamp":1})
metadata.equip_id_1_metadata.sensor_id_1_timestamp_1
````

동일한 인덱스를 time series 컬렉션에도 생성 하여 줍니다.
````
mdb1 [primary] ds> db.dsedip.createIndex({"metadata.equip_id":1,"metadata.sensor_id":1,"timestamp":1})
metadata.equip_id_1_metadata.sensor_id_1_timestamp_1
````

일반 MQL을 그대로 사용 할 수 있습니다.

````
mdb1 [primary] ds> db.dsedip.find({"metadata.equip_id":"DS001","metadata.sensor_id":"SHISENSOR3"}).explain("executionStats")
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'samsung.dsedip',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { 'metadata.equip_id': { '$eq': 'DS001' } },
        { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
      ]
    },
    queryHash: '5B64697A',
    planCacheKey: '5B64697A',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'metadata.equip_id': { '$eq': 'DS001' } },
          { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
        ]
      },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 44640,
    executionTimeMillis: 270,
    totalKeysExamined: 0,
    totalDocsExamined: 401760,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'metadata.equip_id': { '$eq': 'DS001' } },
          { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
        ]
      },
      nReturned: 44640,
      executionTimeMillisEstimate: 43,
      works: 401761,
      advanced: 44640,
      needTime: 357120,
      needYield: 0,
      saveState: 401,
      restoreState: 401,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 401760
    }
  },
  command: {
    find: 'dsedip',
    filter: {
      'metadata.equip_id': 'DS001',
      'metadata.sensor_id': 'SHISENSOR1'
    },
    '$db': 'samsung'
  },
  serverInfo: {
    host: 'mongo.nosql.site',
    port: 27003,
    version: '6.0.7',
    gitVersion: '202ad4fda2618c652e35f5981ef2f903d8dd1f1a'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1689038255, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("9ca8a69967883323f5c9c7db2cdfb33f7ab01549", "hex"), 0),
      keyId: Long("7253067381931507717")
    }
  },
  operationTime: Timestamp({ t: 1689038255, i: 1 })
}
````
Timeseries의 실행 결과로 센서와 설비 정보에 대해 모든 데이터를 조회 하는 것으로 270ms가 소요된 것을 알 수 있습니다.   
일반 컬렉션에서 동일한 쿼리를 수행 하는 경우  결과는 다음과 같습니다.   

````
mdb1 [primary] shi> db.dsedipNormal.find({"metadata.equip_id":"DS001","metadata.sensor_id":"SHISENSOR3"}).explain("executionStats")
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'samsung.dsedipsNormal',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { 'metadata.equip_id': { '$eq': 'DS001' } },
        { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
      ]
    },
    queryHash: '5B64697A',
    planCacheKey: '5B64697A',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'metadata.equip_id': { '$eq': 'DS001' } },
          { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
        ]
      },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 44640,
    executionTimeMillis: 378,
    totalKeysExamined: 0,
    totalDocsExamined: 401760,
    executionStages: {
      stage: 'COLLSCAN',
      filter: {
        '$and': [
          { 'metadata.equip_id': { '$eq': 'DS001' } },
          { 'metadata.sensor_id': { '$eq': 'SHISENSOR1' } }
        ]
      },
      nReturned: 44640,
      executionTimeMillisEstimate: 131,
      works: 401761,
      advanced: 44640,
      needTime: 357120,
      needYield: 0,
      saveState: 401,
      restoreState: 401,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 401760
    }
  },
  command: {
    find: 'dsedipsNormal',
    filter: {
      'metadata.equip_id': 'DS001',
      'metadata.sensor_id': 'SHISENSOR1'
    },
    '$db': 'samsung'
  },
  serverInfo: {
    host: 'mongo.nosql.site',
    port: 27003,
    version: '6.0.7',
    gitVersion: '202ad4fda2618c652e35f5981ef2f903d8dd1f1a'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
  },
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1689038105, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("1476425ef34cc459ff9195a16e59a0002bbced52", "hex"), 0),
      keyId: Long("7253067381931507717")
    }
  },
  operationTime: Timestamp({ t: 1689038105, i: 1 })
}

````

수행 시간이 378ms가 소요 된 것을 확인 할 수 있습니다. 