Thread Safe
===========
    
Java를 이용해 프로그래밍을 진행하다보면 멀티 쓰레드를 사용해야하는 경우가 있다.                               
단, 멀티 쓰레딩 프로그래밍을 진행하면 `동시성`, `동기화`에 대해서 세밀한 주의가 필요하다.                   
    
```java
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}
```  
위와 같은 코드는 동시성 문제를 일으킬 수 있다.                    
`if (singletonObject == null)`를 여러 쓰레드가 동시에 통과한다면              
`singletonObject = new Singleton();`이 여러 번 실행될 수 있기 때문이다.                    
         
멀티 스레드 환경에서 발생할 수 있는 많은 문제들 중 `Race Condition`을 고려하지 못하면서 발생하는 문제다.            
이러한 문제를 해결하기 위해서 여러 방법들이 존재하는데 대표적으로 `synchronyzed`, `volatile`, `Atomic Type`가 있다.         

# 📕 synchronyzed  
`synchronized` 키워드는 임계 영역을 설정하는데 사용된다.     
`synchronized` 키워드를 사용해 임계 영역을 설정하는 방법은 2가지가 있다.     
     
1. 메서드 전체를 임계 영역으로 지정 
2. 특정 영역을 임계 영역으로 지정 

## 📖 메서드 전체를 임계 영역으로 지정
```java
public synchronized void calcSum() {
    // ...
}
```
메서드에 `synchronized` 키워드를 붙이면 메서드 전체가 임계 영역으로 지정된다.        
     
```java
public class Singleton {
    private static Singleton singletonObject;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (singletonObject == null) {
            singletonObject = new Singleton();
        }
        return singletonObject;
    }
}    
```
쓰레드는   
메서드가 호출된 시점부터 객체의 `lock`을 얻어 작업을 수행하다가       
메서드가 종료되면 `lock`을 반환하여 Thread safe를 보장한다.               
      
 
## 📖 특정한 영역을 임계 영역으로 지정
```java
synchronized (클래스.class or 참조변수 or this) {
    // ... 
}
```
메서드 내의 코드 일부를 `{}`으로 감싸고 이를 임계 영역으로 설정한다.                  
이때 **동기화 대상은 락을 걸고자하는 객체 또는 클래스 타입이어야한다.**               
         
```java
public class Singleton {
    private static volatile Singleton singletonObject;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singletonObject == null) {
            synchronized (Singleton.class) { // 자기 자신도 가능(this 가능) 
                if(singletonObject == null) {
                    singletonObject = new Singleton();
                }
            }
        }
        return singletonObject;
    }
}
```

임계 영역은 멀티쓰레드 프로그램의 성능을 좌우하기 때문에 가능하면             
메서드 전체 보다 `synchronized 블럭`으로 임계 영역을 최소화하여 락을 거는 것이 좋다.          

# 📗 volatile         
CPU는 Thread를 동작시킬 수 있으며, CPU 내에는 성능 향상을 위한 L1 cache가 내장 되어있다.        
CPU의 L1 cache를 **CPU cache**라고 부르며 **Main Memory에서 읽은 변수 값을 CPU Cache에 저장한후 캐싱 전략을 이용한다.**          
쉽게 표현하면, **CPU는 Thread를 수행시키면서 성능 향상을 위해 내부의 CPU chache 를 이용한 캐싱 전략을 사용하고 있다.**               

![다운로드](https://user-images.githubusercontent.com/50267433/132934172-671224b7-5ae1-4524-9501-98887b915a54.png)   

```java
public class SharedObject {
    public int counter = 0; 
}
```

그러나, Multi Thread 환경에서 공유 객체를 사용한다고 가정했을 때,       
각각의 Thread마다 CPU Cache에 저장된 값이 다르기에 값 불일치 문제가 발생하게 된다.              
**그렇다면 어떻게 해결해야할까? 🤔 -> volatile 을 이용하면 된다.**             
      
**volatile**    
* volatile는 Java 변수를 Main Memory에 저장하겠다라는 것을 명시하는 것이다.
* 매번 변수의 값을 Read할 때마다 CPU cache에 저장된 값이 아닌 Main Memory에서 읽는 것이다.
* 또한 변수의 값을 Write할 때마다 Main Memory에 까지 작성하는 것이다.
   
```java
public class SharedObject {
    public volatile int counter = 0; 
}
```

`volatile`는 변수의 값을 `Read & Write`할 때마다 CPU Cache가 Main Memory에 저장된 값을 읽어 오도록 하는 키워드다.       
CPU Cache 가 아닌 Main Memory에 저장된 값을 바로 읽으므로 Multi Thread 환경에서의 불일치 문제를 해결할 수 있다.    
즉, 여러 쓰레드에서 공용으로 사용되는 변수의 값과 변동 사항에 대해서 일관성을 제공해준다.        
           
## 🤔 Long 과 Double 에서의 Volatile      
Java 는 long과 double을 제외한 primitive 타입의 변수를 읽고 쓰는 동작에 원자성을 지원해준다.            
그렇다면 왜 long과 double 변수에 대해서 원자성을 지원해주지 못하는걸까?    
        
이는 사실 JVM 비트수와 관련이 있다.           
자바 메모리 모델에 의하면 32비트 메모리에 값을 할당하는 연산은 중단이 불가능하다.(원자적이다)   
그렇지만 long과 double의 경우에는 64비트의 메모리 공간을 갖고있기 때문에 원자성 지원이 안 되었던 것이다.
   
**[참고 링크](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.7)**            
* JVM 연산 비트수는 32비트이기에 long, doulbe에 대한 쓰기는 두번에 이루어진다 
    * 먼저 첫번째 32비트를 쓰고 다음 쓰기에서 두번째 32비트를 쓴다.
    * 첫번째 쓰기와 두번째 쓰기에서  다른 쓰레드가 끼어들어   
      두 32비트 값중 하나를 변경할 수 있기 때문에 long, double은 원자적 연산이 될 수 없다.
* 대신 volatile이 설정된 long, double이라면 항상 원자적이다.    
    * volatile을 사용을 한다는 것은 여러 쓰레드에서 하나의 변수가 같은 값을 읽도록 보장하는 것이기 때문에,   
      메모리를 2번 접근 하더라도 같은 값을 읽도록하는 변수에 접근하는 연산을 원자적으로 수행하게 보장한다는 것이다.     
* 이와 마찬가지로 shared 되는 64bit 값은 volatile이나 synchronize 로 선언하는게 좋다.    
   
이처럼, shared 되는 64bit 값은 volatile이나 synchronize 로 선언하는게 좋으며       
long, double 변수를 원자적으로 사용하고 싶다면 volatile로 선언하는게 좋다.     


# 📘 Atomic    
![다운로드](https://user-images.githubusercontent.com/50267433/132934156-dce6060e-61b3-44e6-8c8a-2f8566b014b7.png)

앞서 언급했듯이 Multi Thread 환경에서는 Thread가 값을 읽어올 때 각각의 CPU Cache에 저장된 값이 다르기에 값 불일치 문제가 발생한다.          
volatile은 Cpu Cache 가 아닌 Main Memory에서 바로 값을 가져오는 형태로 문제를 해결했다.      
      
**Atomic은 CAS를 기반으로 Cpu Cache 값이 Main Memory 값과 동일한 경우에만 동작하는 방식으로 문제를 해결한다.**          
즉, 현재 쓰레드 값과 메인 메모리 값을 비교하여 일치하는 경우 새 값으로 교체하고, 일치하지 않는다면 실패하고 재시도 한다.         
     
## 📖 CAS(CompareAndSwap)      
CAS란, 기존에 가지고 있던 값이 내가 예상하던 값과 같을 경우에만 새로운 값으로 할당하는 알고리즘이다.           
   
```java
// CAS 예시
public class AtomicExample {
    int val;
    
    public boolean compareAndSwap(int oldVal, int newVal) {
        if(val == oldVal) {
            val = newVal;
            return true;
        } else {
            return false;
        }
    }
}
```
`이전 예상 값`과 `변경할 값`을 동시에 입력해서 일치하는지 검증한 후      
`이전 예상 값`이 일치한다면 값을 변경하는 작업을 수행하는 것이다.     
   
## 📖 Java에서의 Atomic과 CAS               
Java에서 제공하는 Atomic Type은 이러한 CAS를 하드웨어(CPU)의 도움을 받아             
**한 번에, 하나의 쓰레드만 접근 가능하고 메서드도 단 한번만 호출할 수 있다.**             
    
```java
// AtomicInteger 예시 
public class AtomicInteger extends Number implements java.io.Serializable {

	private volatile int value; 
        
        public final int incrementAndGet() { 
	    int current; 
            int next;
            do { 
	        current = get(); 
                next = current + 1; 
            } while (!compareAndSet(current, next));
            return next; 
        } 
        
        public final boolean compareAndSet(int expect, int update) { // 메서드 이름은 Swap 이 아닌 Set이다.   
        	return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
        }	
            
}
```
Atomic 타입의 대표적인 클래스인 `AtomicInteger`는 위와 같은 형태로 정의되어 있다.         
앞서 언급했듯이 CPU의 도움을 받아 하나의 쓰레드만 접근 가능하고 메서드도 단 한번만 호출하면서 일관성을 유지한다.      
더불어 Main Memory에 존재하는 값을 확인하기 위해서 volatile 키워드도 사용하는 것을 알 수 있다.        
이러한 Atomic 타입의 가장 큰 장점은 **NonBlocking 하면서 동기화 문제를 해결할 수 있다**는 점이다.          
     
|메서드 시그니처|설명|  
|--------------|---|   
|`incrementAndGet()`|1씩 증가|    
|`accumulateAndGet(int x, IntBinaryOperator accumulatorFunction)`|x 만큼 증가|   
|`compareAndSet(int expectedValue, int newValue)`|expectedValue 값과 비교해 같을 경우에만 newValue 값으로 변경|

# 📙 Concurrent 클래스    
Thread Safe 하다고 알려진 **Concurrent 클래스는 synchronized와 CAS를 이용한 클래스다.**               
단, 클래스마다 구현하고 있는 부분이 다르므로 실제 클래스 내부를 살펴보면서 클래스마다 특징을 이해하면 좋다.              

**ConcurrentHashMap**        
```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    public V get(Object key) {}

    public boolean containsKey(Object key) { }

    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
}
```
putVal()을 살펴보면 **synchronized(f)를 통해 lock을 걸고 있음을 확인할 수 있다.**               
get()에 대해서는 단순히 값을 읽어오는 작업을 진행하기에 lock 을 걸고 있지는 않다.(쓰기는 lock, 읽기는 그냥)         
이 외에도 **Unsafe 클래스의 [CAS(CompareAndSwap)](http://tutorials.jenkov.com/java-concurrency/compare-and-swap.html)구현 메서드를 이용한 동기화도 진행한다.**         
         
**이외 다른 클래스들**        
* `ConcurrentLinkedQueue`는 synchronized는 없고 **VarHandle의 compareAndSet()를 사용한다.**     
* [varHandle vs Unsafe](https://countryxide.tistory.com/88), Unsafe 성능개선이 varHandle 로 이해하면 된다.        
      
