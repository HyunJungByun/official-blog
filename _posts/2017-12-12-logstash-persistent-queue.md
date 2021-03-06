---
title: 로그스태시는 장애 시 어떻게 데이터 손실을 방지할까?
date : 2017-12-12 10:30:00
author : 최용호
---

# 로그스태시는 장애시 어떻게 데이터 손실을 방지할까?

로그스태시는 엘라스틱 스택에서 데이터를 수집하고 가공하여 엘라스틱 서치로 전송하는 역할을 담당하고 있습니다. 이 과정은 입력,필터,출력 플러그인을 통해 하나의 파이프라인을 형성하며 이루어지는데 이 처리 과정 중에 장애가 발생하여 로그스태시가 종료되면 처리중인 데이터는 손실됩니다. 이러한 손실을 방지하기 위해 로그스태시는 어떠한 노력을 하고 있을까요?

로그스태시는 유입된 이벤트를 처리할 때 출력이 완료되어야만 해당 이벤트가 완료된 것으로 판단하고, 그 전에는 큐에 이벤트 레코드를 유지합니다. 로그스태시를 Ctrl+C 또는 SIGTERM 신호를 통해 정상 종료할 경우에는 데이터 수집을 중지하고, 필터와 출력 플러그인에 의해 처리 중인 이벤트들을 정상적으로 처리 완료한 후에 종료합니다. 기본적으로 이 이벤트들이 저장되는 큐는 메모리상에 저장이 되지만 설정에 의해 디스크에 저장(Persistent Queue)되도록 변경할 수 있습니다. 메모리에 기록할 경우에는 로그스태시에 장애가 발생하면 데이터가 손실되기 때문에 디스크에 저장할 것을 권장하고 있습니다.

로그스태시가 비정상적으로 종료되는 경우에는 처리되지 않은 이벤트들은 Persistent Queue(이하 PQ)를 설정한 경우 디스크에 기록되어 로그스태시가 다시 시작되면 디스크에서 이벤트 정보를 읽어와서 처리되지 않은 이벤트들을 이어서 처리하게 됩니다.

PQ는 로그스태시 5.1 버전부터 지원하기 시작했는데 비정상적인 오류가 발생할 경우 데이터 손실을 방지하고 At-Least-Once delivery(최소 한번 배달) 보장을 제공하기 위해 디자인 되었습니다. 이는 데이터 복원 과정 중 중복 데이터가 발생할 수는 있지만 데이터가 손실되는 일은 없도록 하는 것을 의미합니다.

![](http://tech.javacafe.io/img/blog/20171212/PQ_diagram.png)

PQ는 이벤트가 파이프라인을 통해 처리 될 때 입력과 필터 플러그인 사이에서 동작하여 아직 처리되지 않은 이벤트로 큐에 저장합니다. 이 후 출력 플러그인을 통해 데이터 전달하고 처리가 완전히 완료되면 이벤트 처리가 완료된 것으로 큐에 기록됩니다(상태값이 ACK로 변경됨). 디스크 저장으로 인해 PQ를 사용하면 성능에 영향이 있기 때문에, 속도와 내구성 사이의 트레이드 오프가 발생합니다. 또한 이벤트가 저장될 파일의 최대 크기(기본값 1GB)를 넘어서면 로그스태시로의 데이터 유입을 막고 여유 공간을 확보할 때까지 대기하게 되므로 주의해야합니다.

기본적으로 수신된 이벤트가 1024개가 될 때마다 디스크에 작성(queue.checkpoint.writes 설정)하여 데이터 손실을 방지하기 때문에 1024개의 이벤트가 도달되지 않은 채로 로그스태시가 종료되면 데이터가 손실될 수 있습니다.  queue.checkpoint.writes 설정 값을 1로 설정하는 경우 이벤트가 발생할 때마다 디스크에 작성하게 되기 때문에 내구성은 올라가겠지만 성능에 막대한 영향을 끼치게 됩니다. 또한 OS 레벨에서 크래시가 발생할 경우에도 디스크에 안전하게 기록되지 않은 데이터는 손실 될 수 있습니다. 이러한 손실까지 방지하고자 하는 경우 디스크를 RAID 구성과 같은 복제 기술을 사용해야 합니다. (하드웨어적인 문제까지 로그스태시가 대응하지는 못함)

큐에 저장된 이벤트들은 디스크 상에서 페이지라는 단위로 관리하게 되는데 하나의 페이지에는 여러개의 이벤트들이 포함되어 있습니다. 해당 페이지에 속한 이벤트들이 전부 처리가 완료된 상태(ACK)가 되면 디스크 가비지 콜렉션에 의해서 해당 페이지가 삭제됩니다. 만일 페이지 내에 이벤트들 중 하나라도 처리가 완료되지 않은 경우에는 해당 페이지가 디스크에 유지됩니다.  

Elastic 블로그에 따르면 AWS EC2 c3.4xlarge 인스턴스에서 로그스태시 5.4.1 버전으로 in-memory 큐와 PQ의 성능을 벤치 마크한 결과 in-memory에 비해 10%의 성능 저하가 발생하였는데, 이 수치는 테스트한 환경에 따라 달라질 수 있습니다. In-memory 큐를 사용한 경우 초당 약 10600개의 이벤트가 발생하고, PQ를 사용한 경우 초당 약 9500개의 이벤트가 발생하였습니다.

![](http://tech.javacafe.io/img/blog/20171212/pq_chart.png)

개인적인 생각으로는 대부분의 경우에는 PQ를 사용하고, in-memory를 사용해야 한다면 데이터를 생성하는 응용프로그램에서 Logstash와 병행하여 별도의 저장소에 데이터를 전달하여 언제든 복원이 가능한 환경을 구성하는 것이 좋을 것이라 생각합니다.


## 참고

* https://www.elastic.co/blog/logstash-persistent-queue
* https://www.elastic.co/guide/en/logstash/current/persistent-queues.html#durability-persistent-queues
