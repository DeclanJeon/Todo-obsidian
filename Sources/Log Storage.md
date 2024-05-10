#### ELK 스택이란 무엇인가요?
https://aws.amazon.com/ko/what-is/elk-stack/

#### ELK(Elasticsearch, Logstash, Kibana)에 대한 간단한 소개 및 구성 예시
출처: [https://mangkyu.tistory.com/282](https://mangkyu.tistory.com/282) [MangKyu's Diary:티스토리]

![[Pasted image 20240510144137.png]]


### **[ Filebeat의 역할 ]**

Filebeat(파일 비트)는 각 서버에 설치되어 로그 파일의 전송을 담당하는 로그 전송 경량화 프로세스이다. 현재 구조에서는 nginx와 tomcat로그가 모두 필요하여 2종류의 로그를 전송하고 있고, 로그의 구분은 필드에 타입을 추가하여 처리한다. 1개의 프로세스를 통해 여러 개의 로그 종류를 전송하는 부분은 Filebeat 설정 쪽에서 자세히 살펴보도록 하자.

Filebeat는 크게 로그를 가져올 입력 플러그인과 로그를 전송할 출력 플러그인으로 구분할 수 있다. [입력 플러그인](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)을 통해서는 Logstash, Kafka, Redis 등으로 로그를 전송할 수 있고, [출력 플러그인](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)을 통해서는 파일, 콘솔 등을 포함해 Logstash, ElasticSearch 등으로 로그를 전달할 수 있다. 생각보다 많은 옵션을 지원하고 있고, 심지어는 HTTP 요청 플러그인도 있어서 필요하다면 찾아보도록 하자.

### **[ Kafka의 역할 ]**

앞서 살펴본대로 Filebeat에서 Logstash로 로그를 바로 전송할 수도 있다. 하지만 중간에 Kafka(카프카)를 둔 이유는 로그의 유실을 최대한 방지하기 위함이다. 현재 구축하고자 하는 시스템의 로그가 중요한 부분이라 네트워크 등에 의한 이슈가 발생하지 않도록 설계하였다. 만약 로그의 유실을 용인할 수 있는 상황이라면 굳이 카프카까지 넣을 필요는 없다고 생각한다. 불필요하게 카프카 운영 비용이 증가할 수 있으므로, 해당 부분은 시스템 상황에 맞게 가져가는 것이 좋다.

### **[ Logstash의 역할 ]**

Logstash는 데이터 처리를 위한 파이프라인으로, Filebeat 또는 Kafa 등의 입력으로부터 받은 로그를 가공할 수 있다. 불필요한 로그는 drop하고, 별도의 필드로 뽑아야하는 부분은 파싱하고, 심지어 루비를 이용해 코드를 작성하는 등 다양한 기능을 제공한다.

Logstash는 [입력 플러그인](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)으로 Kafka Consumer를 지원하므로 Kafka로부터 메세지를 컨슘해올 수 있다.

앞서 설명한대로 Filebeat를 통해 2종류의 로그를 전송하고 있으므로 각각에 대한 Logstash 파이프라인이 필요한데, 현재 구조에서는 1개의 Logstash 프로세스를 통해 톰캣과 엔진엑스 로그를 모두 처리하도록 되어 있다. 이렇게 한 이유는 로그의 종류 별로 Logstash를 따로 두면 관리 포인트가 너무 많아질 것 같기 때문이다. 물론 이는 업무 상황에 따라 달라질 수 있다.

### **[ Elasticsearch와 Kibana의 역할 ]**

Elasticsearch(엘라스틱 서치)에서는 전처리된 로그 데이터를 저장한다. 하나의 데이터를 document(다큐먼트)라고 부르며, 이 데이터들을 묶는 집합이 index(인덱스)이다. 인덱스는 기본적으로 shard(샤드) 단위로 구분되고, 각 노드에 분산되어 저장된다.

일반적으로 하루 단위로 인덱스가 생성되도록 인덱스의 패턴을 맞추어주며, 해당 패턴에 따라서 lifecycle(생명주기, 라이프사이클)을 설정해줄 수 있다. 엘라스틱 서치의 ILM(Index Lifecycle Management) 기능을 통해 오래된 데이터를 자동으로 삭제시킬 수 있다. 데이터를 자동으로 삭제하지 않으면 디스크만 계속 차게 되므로 반드시 설정해주어야 한다.

Kibana는 시각화 도구로써 엘라스틱 서치에 저장된 데이터를 시각화하여 볼 수 있다.

출처: [https://mangkyu.tistory.com/282](https://mangkyu.tistory.com/282) [MangKyu's Diary:티스토리]


Elasticsearch Github : https://github.com/elastic/elasticsearch
Logstash Github : https://github.com/elastic/logstash
Kibana Github : https://github.com/elastic/kibana