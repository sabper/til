# Dockerfile 작성 시 알아 두면 좋은 내용

## Linux Signal 처리

<https://cloud.google.com/architecture/best-practices-for-building-containers#optimize-for-the-docker-build-cache>

* Docker container 내부로 SIGTERM 등 Signal 을 보내기 위해서는 container 내 process 가 PID 1 로 등록 되어 있어야 함
* PID 1 이 아닌 Docker container 내 Process 는 SIGTERM 신호를 받지 못하므로, SIGKILL 로 container 내 process 를 강제 종료 해야 하는데,
  * 이럴 경우, conatiner 내 process 가 사용자 표시 오류, 쓰기 중단 (데이터 저장), 원치 않는 알람 등이 발생 할 수 있다.
* 일반적으로 Linux 에서 zombie process가 생기면 PID 1 Process (systemd 등 init process) 에 등록 되며, PID 1 Process 가 좀비 프로세스에 대한 제거도 담당 한다.
  * Container 내부에서도 PID 1 process가 zombie process 제거를 담당 하기 된다.
  * Container 내부에 PID 1 Process 가 존재 하지 않는 다면, zombile process 로 인한 리소스 부족 문제가 발생 할 수 있다.
  * 흠, Continaer 내부에서 PID 1 Process 가 존재 하지 않는 가능성도 있는지???
  * PID 1 Process 는 continaer 내부에서 첫번째로 실행 된 Process 이니까 무조건 존재 하는건 아닌가?
* Dockerfile 에서 process 실행 할 때, CMD 또는 ENTRYPOINT 로 실행 하게 되면 PID 1 로 container 내에 process 가 실행 된므로, 위의 문제들이 해결 된다.
  * Dockerfile 에서 Shell Script 로 process 를 실행 하는 경우, Shell Script 가 PID 1 Process 가 되고, Shell Script 가 실행 하는 실제 Process 는 PID 1 이 아니게 된다.
    * Shell Script 에서 exec 로 process 실행 하게 끔 해서 Shell Script 에서 내에서 실행 되는 process 가 PID 1 을 가지게 해야 한다
* Docker Container 내부에서 Linux 에서 init process (systemd or SysV) 를 사용 하게끔 구성 해서 위 문제를 해결할 수 있다.
  * tini 같은 것들이 있음
  * 올바른 신호 핸들러를 등록 하고, container 내 app 에서 치리 하도록 확인
  * zombie process 를 제거

## Dockerfile 에서 캐시 최적화

<https://cloud.google.com/architecture/best-practices-for-building-containers#optimize-for-the-docker-build-cache>

* Dockerfile 에서 COPY, RUN, ADD 등은 캐시 레이어를 생성 하게 되고,
* 이러 한 캐시 레이어 들은 Dockerfile 에 정의 된 순서대로 체인 관계를 갖게 되는데,
* 상위에 생성된 레이어가 변경 된 하위에 생성 된 레이어 들에 대한 캐시가 초기화 된다.
* 그러므로 자주 변경 되는 command (COPY, RUN, ADD) 는 하위로 이동 하는 것이 캐시를 효율적으로 사용할 수 있는 방법이다.
* 또 `apt-get update` -> `apt-get install` command 를 Dockerfile 에서 실행 할 경우,
  ```bash
    FROM debian:9

    RUN apt-get update
    RUN apt-get install -y nginx
  ```
  * `apt-get update` 레이어는 캐시 되므로 
  * 다음에 실행 되는 `apt-get install` 은 apt-get update 가 실행 되지 않은 상황에서 실행 되게 됨
  * 그럼 하나의 레이어에서 실행 되게끔 해야 함
    ```bash
      FROM debian:9

      RUN apt-get update && \
          apt-get install -y nginx    
    ```

## 참고 링크
* <https://cloud.google.com/architecture/best-practices-for-building-containers>
* <https://jonnung.dev/docker/2020/04/08/optimizing-docker-images/>