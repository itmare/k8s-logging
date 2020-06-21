logging for kubernetes
======================

<br><br><br>

logstash vs. fluentd
--------------------

### Logstash

-	elastic의 대표 서비스인 elk stack중 하나
-	elk는 오픈 소스 데이터 수집 엔진
-	여러형태의 데이터 소스를 elasticsearch에 저장한다. (system metrics, logs, web application)
-	data ingestion을 통해 raw데이터를 parsing, normalizing 하여 es에 저장한다.
-	apache 2 라이센스

### fluentd

-	CNCF foundation
-	오픈 소스 데이터 컬렉터
-	structured data format으로 변환하여 es나 object storage같은 저장소로 저장
-	apache 2 라이센스

### Event routing

-	logstash
	-	if-then statement 알고리즘 기반
-	fluentd
	-	tag 기반

### plugin

-	logstash
	-	하나의 elastic github repo에 모든 플러그인 관리 (centralized)
	-	200여개의 plugin 제공
-	fluentd
	-	여러 repo에서 플러그인 관리 (decentralized)
	-	상대적으로 빈약한 plugin (30여개)

### transport

-	logstash

	-	persistent internal message queue가 부족, 20개의 fixed-size event를 on-memory queue에 보관 (logstash의 고질적 이슈)
	-	redis와 같은 external queue에 의존할 수 있음

-	fluentd

	-	easy-to-configure buffering system
	-	파라미터 수정을 통해 in-memory 또는 on-disk 선택가능
	-	상대적으로 복잡한 컨피그 설정을 따로 해줘야 함

### performance

-	logstash
	-	약 120MB 메모리 사용
	-	parent/child 구조 필요시, beat 사용 (거의 필수)
	-	beat: 하나의 데이터 소스에만 포커스
-	fluentd
	-	약 40MB 메모리 사용
	-	parent/child 구조 필요시, fluent-bit 사용해도 됨 (선택)

### monitoring (처리 현황 추척을 위한 모니터링)

-	logstash
	-	독립적인 모니터링 불가
	-	다양한 필터 매트릭을 통해 데이터를 추적, graphite로 보내고, grafana로 시각화
-	fluentd
	-	처리 흐름 내 상태를 모니터링 할 수 있는 쿼리 제공
	-	에이전트를 통해 처리
	-	모니터링 플로그인을 통해 확인 가능

<br><br><br>

fluentd vs. fluent-bit
----------------------

|              | Fluentd                           | Fluent Bit                        |
|--------------|-----------------------------------|-----------------------------------|
| scope        | Containers/Servers                | Embedded Linux/Containers/Servers |
| Language     | C & Ruby                          | C                                 |
| Memory       | ~ 40MB                            | ~650KB                            |
| Performance  | High Performance                  | High Performance                  |
| Dependencies | requires a certain number of gems | Zero dependencies                 |
| Plugins      | More than 1000 plugins            | Around 70 plugins                 |
| License      | Apache 2.0                        | Apache 2.0                        |

### Fluentd

-	log Aggregators
-	40MB 메모리 사용
-	초당 10,000개의 이벤트를 처리
-	buffering과 queuing을 위해 disk와 memory를 사용 (to handle transmission failure, data overload)
-	kubernetes의 로깅에서 사실상 표준
-	다양한 input과 데이터 처리, 다양한 output을 위해 사용됨
-	무거운 처리량을 다루기 위해 디자인

### Fluent Bit

-	log forwarder
-	고도로 분산된 환경에서 사용
-	450KB 메모리 사용
-	not pluggable, not flexible
-	단순히 데이터를 보내기 위한 용도로 사용

<br><br><br>

kubernetes에 fluentd가 적합한 이유?
-----------------------------------

-	적은 메모리 사용
-	분산환경에 agent가 설치되는 lightweight shipper
-	크지않은 쿠버 클러스터 환경이라면 fluent-bit으로 로그만 저장소로 보내기
-	만약 일정부분의 pre-processing과 aggregating이 필요하다면 fluentd를 사용
