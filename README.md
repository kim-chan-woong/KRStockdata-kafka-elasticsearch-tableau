# KRStockdata-kafka-elasticsearch-tableau   

## RESULT   
<img width="80%" src="https://user-images.githubusercontent.com/66659846/115668645-54e06480-a382-11eb-8562-cb2dda93cdb6.gif"/>   

## 설명   
kafka와 elasticsearch를 통한 메시징 기반 실시간 수집 예제이며, 원천데이터는 실시간으로 발생되는 데이터가 아닌   
이미 형식이 갖춰진 정형 데이터입니다. (초당 수 십건의 데이터를 api를 통해 호출하기엔 제약이 있으므로)   
국내 시가총액 과거 데이터를 python을 통해 한 줄씩 읽어 들여 비정형(json) 데이터를 초 단위로 생성시켜   
실시간으로 생성되는 데이터로 가정하였음을 알립니다.   

## Process   
1. Collection: 초 단위로 생성되는 데이터(json)를 kafka-python 모듈을 활용하여 producer 메시지 생산 후 전송   
               Nifi의 ConsumerKafka Processor를 통해 메시지 수집   
2. Load: Nifi의 PutElasticsearch를 통해 수집한 메시지를 elasticsearh 인덱스 저장   
3. Visualization: elasticsearch에 적재된 데이터를 tableau를 통해 시각화   
![Screenshot_130](https://user-images.githubusercontent.com/66659846/115668980-bb658280-a382-11eb-95bb-ae4054421af6.png)   

## Servers, spec   
Virtaul Box - 6.1   
MobaXterm - 8.6   
CentOS - 7   
Java - 8   
Python - 3.8.5   
Nifi - 1.9.2   
Kafka - 2.7.0   
zookeeper - 3.4.10   
elasticsearch - 7.12.0   
kibana - 6.2.1   
tableau - 무료 평가판   
![Screenshot_131](https://user-images.githubusercontent.com/66659846/115671901-f5845380-a385-11eb-89c6-377e6b6a4f45.png)   

## 과정   

### 서버 생성 및 MobaXterm 원격 작업   
![Screenshot_133](https://user-images.githubusercontent.com/66659846/115673671-dbe40b80-a387-11eb-8a87-6565bd767467.png)   

### 실시간 데이터 생성 파일(.py) 실행(getdataserver)   
1. jupter notebook 테스트 후 리눅스 내 스크립트 형태로 실행   
2. 앞서 언급했듯이, 실제 원천 데이터는 이미 정형화된 데이터지만 파이썬 코드로 실행함으로써,   
   초 단위로 데이터가 생성된다. 이 때의 데이터를 원천데이터로 가정한다.   
3. 가정한 원천데이터는 1995-05-02 ~ 2020-12-31일 까지의 국내 시가총액 데이터 TOP 20이며,   
   정형화된 데이터를 한 행씩 읽어들여 json형식의 데이터(일별 시가총액 20위)를 생성한다.   
   이 때의 데이터를 실시간으로 생성되는 원천 데이터로 가정한다.   
4. 총 데이터는 128,878행이고 컬럼은 16개이다.   
6. 실시간 데이터 생성 과정에서 주식 장이 열리지 않은 데이터는 제외하였다.(출력)   
7. 파이썬 코드 내에 kafka-python 모듈을 활용하여 kafka producer로 메시지가 생성된다.   
8. 전송 결과 초당 약 16건의 데이터를 처리하였다.   
9. 코드는 별도 첨부   
![Screenshot_142](https://user-images.githubusercontent.com/66659846/115679891-020caa00-a38e-11eb-905c-1aa34a54b06b.png)   
![Screenshot_127](https://user-images.githubusercontent.com/66659846/115673867-0c2baa00-a388-11eb-9713-223916207eee.png)   
![Screenshot_128](https://user-images.githubusercontent.com/66659846/115680189-4ef08080-a38e-11eb-95cf-e9fce03a0d07.png)   

### 카프카 메시지 수집 확인(ka01, ka02, ka03)   
1. Kafka Cluster는 총 3대의 서버로 구축(Kafka, Zookeeper)   
2. 파이썬 코드 내 producer를 통해 메시지가 생성되고 consumer를 통한 수집 확인   
![Screenshot_139](https://user-images.githubusercontent.com/66659846/115677635-a5a88b00-a38b-11eb-8826-a0aa76009eea.png)   
![Screenshot_137 - 복사본](https://user-images.githubusercontent.com/66659846/115675963-1353b780-a38a-11eb-9ea2-73e3ec051d70.png)    

### Nifi 수집 및 적재(rm01:18080)   
1. ConsumerKafka: 메시지를 실시간으로 읽어 들임   
2. PutElasticsearchHttp: 실시간 json데이터를 elasticsearch의 지정한 index에 적재(index: nifi-elk, type: json)   
3. LogAttribute: 로그 확인   
4. Queued: 모니터링   
5. Processor 설정 별도 첨부   
![Screenshot_129](https://user-images.githubusercontent.com/66659846/115676148-4302bf80-a38a-11eb-89e1-ff752ea02a21.png)   

### elasticsearch cluster 확인(elkmaster:9200, elkdn01:9200)   
![Screenshot_150](https://user-images.githubusercontent.com/66659846/115682484-75172000-a390-11eb-9397-0ef07a3a6bcb.png)   

### elasticsearch 적재 확인   
![Screenshot_140](https://user-images.githubusercontent.com/66659846/115678368-63337e00-a38c-11eb-98a5-3429c479663b.png)   

### elasticsearch 인덱스 정보 및 매핑 확인   
index: nifi-elk   
shard: 2   
type: json   
mapping:   
![Screenshot_149](https://user-images.githubusercontent.com/66659846/115682014-06d25d80-a390-11eb-87b6-174e4078bb3b.png)   
![Screenshot_145](https://user-images.githubusercontent.com/66659846/115682022-089c2100-a390-11eb-8ea1-0d01f6ad3031.png)   
![Screenshot_146](https://user-images.githubusercontent.com/66659846/115682043-0c2fa800-a390-11eb-97ef-cc35840820a1.png)   
![Screenshot_147](https://user-images.githubusercontent.com/66659846/115682056-0df96b80-a390-11eb-9d56-ff330eb2fc5a.png)   
![Screenshot_148](https://user-images.githubusercontent.com/66659846/115682065-105bc580-a390-11eb-806d-9156f713aea0.png)   

### kibana를 통해 조건 검색(elkmaster:5601)   
![Screenshot_141](https://user-images.githubusercontent.com/66659846/115678710-c1f8f780-a38c-11eb-847c-378a07f0035b.png)   
