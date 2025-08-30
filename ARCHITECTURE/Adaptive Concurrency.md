## Adaptive Request Concurrency (ARC)

Adaptive Request Concurrency(ARC)는 Vector의 네트워킹 기능으로, 고정된(rate limit) 방식의 요청 제한을 없애고 다운스트림 서비스의 응답에 따라 HTTP 동시 처리(concurrency) 한도를 자동으로 최적화합니다.

이 기능은 TCP 혼잡 제어 알고리즘에서 착안한 피드백 루프(feedback loop) 를 기반으로 동작합니다.

이를 통해 관측(Observability) 인프라 전반의 성능과 안정성이 크게 향상됩니다.

자세한 내용은 [ARC 기능 발표](https://vector.dev/blog/adaptive-request-concurrency/)를 참고하세요.
