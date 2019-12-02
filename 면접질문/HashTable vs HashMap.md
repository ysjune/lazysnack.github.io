# HashTable VS HashMap
<br/>
메모리 내에 key/value 짝으로 되어있는 데이터이다. 두 데이터 타입의 차이점을 알아보고자 한다.

<br/>

### 1. 비슷한 점

- 추가, 삭제, 접근이 비슷하다.

 > 1. get() : 키를 통해 아이템 검색
 > 2. remove() : 키를 통해 삭제
 > 3. put() : 아이템을 추가

- java.util 패키지 내에 속해있다.

<br/>

### 2. 다른 점

- 두 데이터 타입은 매우 많은 다른 점이 존재하지만, 주된 것 몇 가지만 알아보자

 > 1. HashTable 의 경우 암묵적으로 동기화되어 멀티 스레드 환경에서 동작하기 용이하다. <br/>
 엑세스 시 스레드는 HashTable 을 락해 다른 스레드가 동시에 변경하는 것을 방지한다. <br/>
 반대로 HashMap 의 경우에는 단일 스레드 환경에서 적합하다. 만약 멀티 스레드 환경에서 HashMap 을 사용하고자 한다면 <br/>
 ConcurrentHashMap 을 사용할 수 있다.
 
 > 2. HashTable 의 경우 단일 스레드 환경에서도 각 메소드 호출할 때 암묵적으로 동기화를 하기 때문에 HashMap 보다 느리다.
 
 > 3. HashMap 은 Null 을 저장할 수 있지만, HashTable 은 저장할 수 없다.
 
 > 4. HashMap 의 경우 fail-fast 로 고려되어지는 Iterator 을 사용할 수 있다. 즉, iterator 하고 있는 도중에 <br/>
 다른 스레드가 Map 을 변경할 경우 ConcurrentModificationException 을 던진다. 하지만, HashTable 의 경우 fail-fast 가 아닌 Enumerator 로 순회한다.
 
 >5. HashMap 의 경우 LinkedHashMap 과 TreeMap 을 구현함으로써 정렬이 가능하지만, HashTable 은 그렇지 못하다.
 
<br/>

##### 결국 HashTable 은 deprecate 되거나 공식적으로 ConcurrentHashMap 으로 대체된다고 합니다.

<br/>
HashTable 의 경우 좀 생소했습니다. ConcurrentHashMap 은 사용해본 적이 있지만요. <br/>
어찌보면 차이점은 동기화 라고 볼 수도 있을 것 같은데, 그것도 ConcurrentHashMap 이 나오면서 해결이 된 느낌이네요.<br/>
그러면 드는 생각은 HashTable vs ConcurrentHashMap 인데요. 이 부분도 나중에 찾아봐야겠군요.


<br/><br/>
[출처] - https://www.programmergate.com/hashtable-vs-hashmap/
