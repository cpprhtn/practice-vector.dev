# Log events

다음은 log event를 JSON으로 표현한 예시입니다:

```json
{
  "log": {
    "custom": "field",
    "host": "my.host.com",
    "message": "Hello world",
    "timestamp": "2020-11-01T21:15:47+00:00"
  }
}
```

---

## Schema

> Vector log event는 특정 시점의 이벤트를 구조적으로 표현하며, 임의의 필드 집합을 포함할 수 있습니다. Vector는 스키마 중립성을 유지하여 기존 스키마는 물론 향후 다양한 스키마와도 호환됩니다. 특정 필드를 요구하지 않으며, 각 구성 요소는 제공하는 필드를 문서화합니다.
>> ## Options
>>> **\*** (common, optional, *)  
>>> 무한히 중첩 가능한 임의의 key/value 쌍 집합입니다.
> ### 예시
> ```json
> {
>   "custom": "field",
>   "host": "my.host.com",
>   "message": "Hello world",
>   "timestamp": "2020-11-01T21:15:47+00:00"
> }
> ```

---

## How it works

### Schemas

Vector는 특정 스키마에 구애받지 않으며, 다양한 스키마와 호환됩니다. 이벤트는 구조화된 데이터를 포함하고, 필요에 따라 임의의 필드를 추가할 수 있습니다.

---

### Types

* **Strings**  
UTF-8 호환 문자열. 시스템 메모리에 의해서만 제한됩니다.

* **Integers**  
부호 있는 64비트 정수

* **Floats**  
64비트 부동소수점 수. IEEE 754 표준을 따릅니다.

* **Booleans**  
true/false 값을 나타내는 이진(Boolean) 타입

* **Timestamps**  
[Rust의 `DateTime` structs](https://docs.rs/chrono/latest/chrono/struct.DateTime.html)로 표현되며 UTC로 저장됩니다.

* **Timestamp Coercion**  
JSON 등 정식 타임스탬프 정의가 없는 형식과 상호작용할 경우, Vector는 해당 필드를 문자열이나 정수로 수집합니다. 이후 VRL 함수 `parse_timestamp`를 사용하여 필드를 timestamp로 변환할 수 있습니다.

* **Time zones**  
Timezone 정보가 없는 timestamp는 로컬 시간으로 간주 후 UTC로 변환됩니다.

* **Null values**  
JSON 호환을 위해 `null` 값도 지원됩니다.

* **Maps**  
문자열 키와 임의의 타입 값을 매핑하는 연관 배열(associative array)입니다.

* **Arrays**  
임의 타입 값을 순차적으로 나열한 시퀀스입니다.