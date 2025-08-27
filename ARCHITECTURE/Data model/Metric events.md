# Metric events

## Schema
Vector metric events는 시계열 데이터 상에서 수행되는 수치 연산을 나타냅니다. 다른 도구와 달리 벡터의 메트릭은 구조화된 로그로 표현되지 않는 일급 객체입니다. 따라서 별도의 변환 없이 다양한 메트릭 수집/분석 서비스와 상호 운용할 수 있습니다.

Vector의 메트릭 데이터 모델은 이념적 순수성보다 정확성과 일관성을 중요시합니다. 따라서 Vector의 메트릭 유형은 Prometheus 및 StatsD와 같이 현장에서 발견되는 다양한 메트릭 유형을 통합한 것입니다. 이를 통해 메트릭 데이터가 시스템 간에 올바르게 상호 운용될 수 있습니다.


 ### Options
> **counter** *(`common` `optional` `table`)*  
> 증가하거나 0으로 재설정할 수 있지만 감소할 수 없는 단일 값입니다.
>> **value** *(`required` `float`)*  
>> 카운터를 증가시킬 값입니다. 양수만 허용됩니다. 
>>  
>> **Examples:**  
>> 1 10 500

> **distribution** *(`common` `optional` `table`)*  
> 샘플링된 값들의 분포를 나타냅니다.  
> 글로벌 histogram과 summary를 지원하는 서비스에서 사용됩니다.

>> **samples** *(`required` `array`)*  
>> 샘플링된 값 집합
>>
>> **statistic** *(`required` `string` `literal` `enum`)*  
>> 값들에서 계산할 통계 
>>
>> **Examples:**  
>> histogram summary

> **gauge** *(`common` `optional` `table`)*  
> **point-in-time 값**을 나타내며, 증가/감소 모두 가능합니다.
> Vector의 gauge 타입은 값이 시간에 따라 변동하는 상황을 표현하며, 메모리 사용량이나 CPU 사용량처럼 변동(fluctuation)이 있는 값을 추적하는 데 적합합니다.
>> **value** *(`required` `float`)*  
>> 특정 시점의 값
>>
>> **Examples:**  
>> 1 10 500

> **histogram** *(`common` `optional` `table`)*  
> timer라고도 불립니다. histogram은 관찰값을 샘플링하여 버킷(bucket)에 카운트하고, 전체 합도 제공합니다. 주로 request duration이나 response size 같은 데이터를 측정할 때 사용됩니다.
>> **buckets** *(`required` `array`)*  
>> histogram 값을 담는 버킷 집합
>>
>> ---
>> **count** *(`required` `uint`)*  
>> histogram에 포함된 값의 총 개수
>>
>> **Examples:**  
>> 1 10 25 100
>>
>> ---
>> **sum** *(`required` `float`)*  
>> histogram에 포함된 값의 총합
>>
>> **Examples:**  
>> 1 10 25 100
>>

> **interval_ms** *(`optional` `uint`)*  
> 이 metric 값이 나타내는 시간 간격

> **kind** *(`required` `string`)*  
> metric 값의 종류

> **name** *(`required` `string`)*  
> metric 이름

> **namespace** *(`optional` `string`)*  
> metric 네임스페이스 (서비스에 따라 name 앞에 붙거나 네이티브 네임스페이스 기능 사용)

> **set** *(`common` `optional` `table`)*  
> 고유 값들의 배열을 나타냅니다.
>> **values** *(`required` `array`)*  
>>고유 값 리스트

> **summary** *(`common` `optional` `table`)*  
> histogram과 유사하게 관찰값을 샘플링합니다. 전체 개수와 합을 제공하며, sliding time window 상에서 configurable quantile을 계산합니다.
>> **count** *(`required` `uint`)*  
>> summary에 포함된 값의 총 개수
>>
>> **Examples:**  
>> 54
>>
>> ---
>> **quantiles** *(`required` `array`)*  
>> 관찰값 집합
>>
>> ---
>> **sum** *(`required` `float`)*  
>> summary에 포함된 값의 총합
>>
>> **Examples:**  
>> 1 10 25 100

> **tags** *(`required`, `table`)*  
> metric tag를 나타내며, 각 tag 이름을 key로, 단일 값 또는 값의 리스트를 매핑합니다. (값은 string 또는 null)
>> **\*** *(`optional`)*  
>> tag 이름을 key로, 단일 값 또는 값의 리스트를 매핑 (string 또는 null)

> **timestamp** *(`required` `timestamp`)*  
> metric이 생성된 시각
