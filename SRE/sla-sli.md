# SLA, SLI 는 무엇을 의미 하는가?

SRE 에서는 적절한 수준의 안정성을 지속적으로 달성 하는 것이 목표인 조직 이고,

적절한 수준의 안정성을 정량적으로 정의 하는 것이 필요

적절한 수준의 안정성이란 100% 안정적인 시스템을 만들기 위해 극악의 노력을 해야 할 수 있는 상황이 오므로, 
적절한 수준의 안정성을 목표를 정하고 달성 하는 것이 더 효율적이다

### SLO (Service-Level Objective) 서비스 수준 목표
적절 수준의 안정성에 대한 목표 수치

* 이들 SLI 지표를 이용해서 404의 비율을 2% 미만으로 한다. 
* 500의 비율을 1% 미만으로 한다. 
* 응답의 90% 이상이 500msec 안에 있어야 한다 

등의 서비스 수준 목표를 설정할 수 있다.

### SLI (Service-Level Indicator) 서비스 수준 지표
SLO 를 어느 정도 달성 했는지 판단 하기 위한 근거 자료

#### SLI 예
* 쿼리 성공과 실패의 비율
* HTTP 404, 500 에러의 비율
* Long query의 갯수
& 응답까지 걸린 시간

### 관련 링크
* https://www.joinc.co.kr/w/man/12/sre
* https://sre.google/workbook/implementing-slos/