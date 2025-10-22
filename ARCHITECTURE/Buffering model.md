## Buffering model

Vector는 **buffering model**을 구현하여, 이벤트가 sink가 처리할 수 있는 용량을 초과할 때 **성능(performance)** 과 **내구성(durability)** 중 어떤 것을 우선할지 선택할 수 있도록 합니다.

---

### Backpressure와 버퍼링 필요성

운영자는 일반적으로 Vector 배포가 예상 부하에 맞게 적절히 구성되도록 하지만, 때때로 문제가 발생할 수 있습니다.
  * example: 애플리케이션이 평소보다 많은 로그를 생성하거나, 데이터를 전송하는 다운스트림 서비스가 평소보다 느리게 응답하는 경우.

Vector topology 설계의 일부는 **backpressure 전파**를 포함합니다.

* Backpressure: 이벤트가 수신되는 속도보다 처리 속도가 느릴 때 발생하는 신호
* 한 component가 다른 component로 처리할 수 있는 양보다 더 많은 이벤트를 보내려 하면, **간접적으로(sender component)에 알려집니다**.
* Backpressure는 **sink에서 source까지**, 심지어 **HTTP로 로그를 전송하는 클라이언트까지** 전달될 수 있습니다.

Backpressure는 시스템이 **더 많은 작업을 처리할 수 있는지, 혹은 너무 바쁜지**를 나타내는 방법입니다.
이를 통해 Kafka나 HTTP 소켓과 같은 source에서 이벤트를 pull하거나 수신 속도를 조절할 수 있습니다.

그러나 항상 즉시 backpressure를 전파하는 것이 바람직하지 않을 수 있습니다.

* upstream component가 계속 느려지거나, Vector 외부에서 문제를 유발할 수 있기 때문
* component가 포화 상태(threshold)를 살짝 넘어서는 경우, 전체 속도를 늦추지 않고 임시 지연이나 다운타임을 처리할 수 있어야 합니다.

**Buffering**은 이러한 문제를 해결하기 위한 Vector의 접근 방식입니다.

---

### Component 간 버퍼링

Vector topology의 모든 component는 **in-memory buffer**를 갖습니다.

* 기본 목적: 두 component 간 통신 채널
* 추가 목적: 수신 component가 바쁠 때도 **약간의 공간(보통 100 이벤트)** 을 사용하여 이벤트를 전송 가능
* 이를 통해 비균일한 workload 상황에서도 **throughput 최대화** 가능

임시 과부하나 장애(outage)를 보호하려면 **workload에 맞춘 buffering 솔루션**이 필요합니다.

---

### Sink에서의 버퍼링

Vector 구성에서는 **sink의 버퍼 설정(buffer configuration)** 을 다룹니다.

* 이유: sink는 topology에서 **주요 backpressure 발생 지점**

  * 네트워크를 통한 서비스 통신 시 지연(latency) 발생 가능
  * 장애(outage)가 일시적으로 발생할 수 있음

* 기본적으로 sink는 다른 component처럼 **메모리 버퍼**를 사용

* 기본 버퍼 크기: 500 이벤트로 조금 더 큼

* 필요 시 **버퍼 타입과 가득 찼을 때 동작(when\_full) 설정**을 완전히 제어 가능

---

### Buffer types

#### In-memory buffers

* 이름 그대로 **메모리에서 이벤트를 버퍼링**

* 가장 빠른 버퍼 타입이지만 두 가지 단점:

  1. **메모리 사용**: 이벤트 수에 비례하여 메모리 소비
  2. **내구성 없음**: Vector나 호스트가 크래시하면 이벤트 손실

* 이벤트 크기 가변: JSON 인코딩 시 예상 크기로 용량 계획 가능

* pull 기반 source(S3, Kafka)는 재시도 가능, push 기반 source는 메시지 재전송 불가할 수 있음

#### Disk buffers

* 데이터 내구성이 **성능보다 중요할 때** 사용

* Vector 재시작/크래시 시에도 이벤트 유지 가능

* **write-ahead log 방식**:

  1. 이벤트를 버퍼에 먼저 전달
  2. 데이터 파일에 기록
  3. 다시 읽어 처리

* 기본 동작: 모든 쓰기마다 디스크 동기화 X, 500ms 간격으로 동기화 → 높은 throughput 유지 + 데이터 손실 위험 최소화

* 디스크 버퍼는 **읽기 속도와 쓰기 속도 균형** 최적화

* 최대 크기 설정 가능, 최소 크기 \~256MiB

* 파일 시스템상 **append-only log 파일**, 최대 128MiB, 처리 완료 시 삭제

* **데이터 손상 감지 시 체크섬 검사**, 가능한 한 많은 이벤트 자동 복구

* 손상 시 metric 생성

---

### 운영자 요구 사항

* 디스크 버퍼는 메모리 버퍼와 달리 **저장 공간 모니터링** 필수
* I/O 오류 발생 시 Vector는 강제로 종료

  * 오류 메시지 출력 후 종료 (예: "no storage space")
  * 재시작 시 손상되지 않은 이벤트 복구 시도
* Vector 데이터 디렉터리(data\_dir) 충분한 여유 공간 확보 필수
* 다른 프로세스가 여유 공간을 사용하지 않도록 주의

---

### “When full” 동작

버퍼 타입 선택만큼 중요한 것이 **버퍼가 가득 찼을 때 동작**입니다.

#### Blocking (block)

* 설정 시, Vector는 **버퍼가 가득 차면 무기한 대기**
* 기본 동작이며, 이벤트 순서(order)를 보장하고 upstream에 backpressure 제공
* 클라이언트로부터 직접 데이터를 수신할 경우, 블로킹은 적합하지 않을 수 있음

#### Drop the event (drop\_newest)

* 메모리 버퍼 + bursty workload에서는 추천하지 않음
* 버퍼가 즉시 채워지고 남은 이벤트는 드롭될 수 있음
* idempotent 데이터나 debug/trace 로그에서는 유용
* topology의 in-flight 이벤트 수 감소 + upstream 블로킹 방지

#### Overflow to another buffer (overflow)

* 아직 프로덕션에는 적합하지 않음(버그 및 데이터 손실 가능)
* 여러 버퍼를 순차적으로 연결하여, 첫 버퍼가 가득 찰 경우 다음 버퍼로 overflow 가능
* 메모리 버퍼 → 디스크 버퍼 순으로 이벤트 버퍼링 가능

**예시 구성**:

```yaml
sinks:
  overflow_test:
    type: blackhole
    buffer:
    - type: memory
      max_events: 1000
      when_full: overflow
    - type: disk
      max_size: 1073741824 # 1GiB.
      when_full: drop_newest
```

* 메모리 버퍼에 공간 생기면 새 이벤트는 메모리로 먼저 버퍼링
* 이벤트 순서(order) 보장 X
* 마지막 버퍼는 overflow 모드 불가 → 가득 찼을 때 block 또는 drop 설정 필요

---

### 권장 버퍼링 구성

| 시나리오                      | 권장 설정                               |
| ------------------------- | ----------------------------------- |
| Vector에 저장 공간 없음          | **In-memory buffer 사용**             |
| 성능 최우선                    | **In-memory buffer + drop\_newest** |
| 내구성 최우선                   | **Disk buffer 사용**                  |
| 일부 source는 클라이언트로부터 직접 수신 | 필요 시 **버퍼가 가득 찰 때 이벤트 drop** 가능     |