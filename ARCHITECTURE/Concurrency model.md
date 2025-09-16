## Concurrency model

Vector는 **들어오는 데이터 볼륨에 따라 자연스럽게 확장되는 concurrency 모델**을 구현합니다.

* 각 Vector source는 **동시성 단위(unit of concurrency)를 정의**하고 이에 맞게 구현합니다.
* 이를 통해 **Vector 사용 방식에 맞춰 자동으로 적응하는 concurrency**가 가능하며, 복잡한 동시성 튜닝이나 설정이 필요하지 않습니다.

예시:

* **file source**: tail하고 있는 파일 개수 단위로 동시성 구현
* **socket source**: 유지 중인 활성 연결(active open connection) 개수 단위로 동시성 구현

---

### Stateless function transforms

* Pipeline model 문서에서 언급했듯이, Vector의 concurrency는 **병렬 처리(parallelization)가 가능한 stateless function transform**에 의존합니다.
* 현재 **task transform은 병렬화가 불가**하며, 처리 과정에서 병목(bottleneck)을 발생시킬 수 있습니다.

  * 향후 개선 예정

---