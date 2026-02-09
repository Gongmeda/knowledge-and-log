# Hashtable vs ConcurrentHashMap

간단한 Todo-list API를 만드는 스터디를 진행하면서 DB를 연결하기 전에 Repository 인터페이스를 in-memory 기반의 구현체로 처리했었습니다.

이때, `Map` 타입의 변수를 통해 저장하고 조회하도록 구현했습니다. 다만, Spring MVC의 멀티스레드 환경을 고려하여 thread-safe한 구현체를 사용하려 했고, 대표적으로 `Hashtable`과 `ConcurrentHashMap`이 있었습니다. 이 둘을 비교하여 `ConcurrentHashMap`을 사용하게 된 과정을 알아보겠습니다.

## 역사

`Hashtable`은 Java 1.0부터 존재했으며 1.2버전부터는 `Map` 인터페이스를 구현하도록 수정되었다고 합니다.

`ConurrentHashMap`은 `HashTable`의 동시성 성능을 개선하기 위해 Java 1.5에 등장한 구현체입니다.

## 성능 테스트

각 구현체의 성능을 측정해보기 위해 JUnit으로 테스트 코드를 작성해봤습니다.

```java
public class MapConcurrencyTest {

    static final int ITERATION_COUNT = 10000000;
    static CountDownLatch latch;
    static Map<Integer, Integer> map;
    static List<Long> times;

    private void init(Map<Integer, Integer> m) {
        latch = new CountDownLatch(20);
        map = m;
        for (int i = 1; i <= 1000; i++) {
            map.put(i, i);
        }
    }

    @Test
    public void test() throws InterruptedException {
        for (Map m : new Map[]{
                new HashMap<Integer, Integer>(),
                new ConcurrentHashMap<Integer, Integer>(),
                new Hashtable<Integer, Integer>()
        }) {
            times = new ArrayList<>();

            for (int t = 0; t < 10; t++) {
                init(m);

                long start = System.currentTimeMillis();

                // Create 10 Writer Threads
                for (int counter = 0; counter < 10; counter++) {
                    new Thread(() -> {
                        for (int i = 1; i <= ITERATION_COUNT; i++) {
                            int key = i % 1000 + 1;
                            map.put(key, key);
                        }
                        latch.countDown();
                    }).start();
                }

                // Create 10 Reader Threads
                for (int counter = 0; counter < 10; counter++) {
                    new Thread(() -> {
                        for (int i = 1; i <= ITERATION_COUNT; i++) {
                            int key = i % 1000 + 1;
                            map.get(key);
                        }
                        latch.countDown();
                    }).start();
                }

                // Wait for all threads to complete
                latch.await();

                long time = System.currentTimeMillis() - start;
                times.add(time);
            }

            System.out.println("Map: " + m.getClass().getSimpleName());
            System.out.println("Execution Times: " + Arrays.toString(times.toArray()));
            System.out.println("Average execution Time: " + times.stream().mapToLong(Long::longValue).average().getAsDouble() + "ms");
            System.out.println();
        }
    }
}

```

이를 실행하여 아래와 같은 결과가 나왔습니다.

```
Map: HashMap
Execution Times: [848, 723, 794, 751, 770, 748, 748, 743, 743, 741]
Average execution Time: 760.9ms

Map: ConcurrentHashMap
Execution Times: [3338, 3820, 3637, 3732, 3806, 3692, 3627, 3580, 3765, 3987]
Average execution Time: 3698.4ms

Map: Hashtable
Execution Times: [23102, 22990, 23054, 23021, 22986, 22965, 22892, 22974, 22970, 22960]
Average execution Time: 22991.4ms
```

결과를 통해 알 수 있듯이 성능은 `HashMap` > `ConcurrentHashMap` > `Hashtable`인 것을 알 수 있습니다.

`HashMap`의 경우는 thread-safe하지 않아 동기화 오버헤드가 없어 가장 빠른 것이 자명합니다. 하지만, `ConcurrentHashMap`과 `Hashtable`은 왜 이렇게 차이가 많이 날까요?

## 왜 Hashtable은 ConcurrentHashMap보다 느릴까?

`Hashtable`은 `get()`, `put()` 모두 `synchronized`를 통해 테이블 전체를 lock하는 반면, `ConcurrentHashMap`은 `get()`에는 lock을 걸지 않고, `put()`에서만 사용되는 키 값에 따라 세분화된 lock을 제공하여 여러 스레드가 동시에 다른 부분을 수정할 수 있습니다.

실제 구현체의 코드를 통해 이를 확인할 수 있습니다.

### Hashtable 내부 코드

```java
public synchronized V get(Object key) {
    // 세부 구현
}
```

```java
public synchronized V put(K key, V value) {
    // 세부 구현
}
```

둘 다 `synchronized` 키워드로 선언되어 동작 시 한번에 하나의 스레드만 접근할 수 있는 것을 확인할 수 있습니다.

### ConcurrentHashMap 내부 코드

```java
public V get(Object key) {
    // 세부 구현
}
```

```java
public V put(K key, V value) {
	return putVal(key, value, false);
}

final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 기타 로직 생략

    for (ConcurrentHashMap.Node<K,V>[] tab = table;;) { // 무한 루프를 통해 삽입될 인덱스를 확인
        ConcurrentHashMap.Node<K,V> f; int n, i, fh; K fk; V fv;
        if () // 세부 구현: 해당 인덱스에 노드가 없는 경우 lock을 걸지 않고 삽입
        else {
            synchronized (f) {
                // 실제 삽입 로직 (동기화 블록)
            }
        }
    }

    // 기타 로직 생략

    return null;
}
```

`get()`은 `synchonized`가 없고, `put()`의 경우는 키값(인덱스)에 따라 필요한 경우에만 사용하여 블로킹을 최소화 한 것을 확인할 수 있습니다.

> ConcurrentHashMap의 내부 구현에 대한 자세한 분석이 궁금하다면 해당 블로그 아티클을 참고하기 바랍니다.

## 결론

위와 같은 특징 때문에, 스레드간 동기화가 필요하다면 대부분 `ConcurrentHashMap`을 사용한다고 합니다.

## 참고

- [HashMap vs HashTable vs ConcurrentHashMap](https://tecoble.techcourse.co.kr/post/2021-11-26-hashmap-hashtable-concurrenthashmap/)
- [Java Hashtable, HashMap, ConcurrentHashMap – Performance impact](https://blog.gceasy.io/2022/04/22/java-hashtable-hashmap-concurrenthashmap-performance-impact/)