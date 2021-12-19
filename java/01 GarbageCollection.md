Java Memory Garbage Collection
=================================
# 🎞 Garbage Collection 서론 
     
* **Garbage Collection** 이란, 이름 그대로 `쓰레기 수집`을 의미한다.        
   
**그렇다면 어떠한 쓰레기를 수집하는 것일까? 🤔**             
자바에서는 **더 이상 사용되지 않는(참조 되지 않는)객체**를 의미한다.      
                    
C언어에 빗대어 설명하자면 메모리를 동적으로 할당하면 명시적으로 `free()`를 사용해서 메모리를 해야했다.(누수방지)                   
하지만, **매번 명시적으로 입력해줘야 했으므로 누락이 발생할 수 있고 이로 인해 발생하는 휴먼 에러는 상당했기에**                     
이를 해결하기 위해 **Garbage Collection가 등장했고 이전에 비해 동적 메모리 해제에 대해서 비교적 덜 신경 쓸 수 있게 되었다.(로직에 집중)**          
단, **GC 또한 완벽한 것은 아니기에 우리 개발자들은 메모리 관리는 물론, GC의 동작 원리에 대해서 알 필요가 있다.**                

# 📕 Garbage Collection 이란?        
Garbage Collection은 아래 2가지 전제 조건을 가정을 하고 만들어졌다.      

* 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
* 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
       
**대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.**         
애플리케이션을 운영하는데 지속적으로 사용되는 객체(메모리)의 수는 비교적 적은편이다.         
실제 개발을 하면서도 필요에 따라 메모리를 할당하고 이를 해제하는 경우가 대다수이다.          
       
**오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.**               
대부분 참조가 이루어지고나면 다른 객체로 재참조하는 일은 거의 없다.          
즉, 이미 생성된 참조에서 새로운 객체로의 참조는 아주 적게 존재한다.      
(예외적인 경우로 `상태 패턴`이 있지만 이는 조금 결이 다르다고 생각이 든다.)        

## 📖 Hotspot Heap Structure   

* 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
* 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
    
2가지 전제 조건을 기반으로 `객체가 저장되는 공간`이자 `Garbage Collection의 대상`인 **Heap 메모리를 아래 2가지 구조로 나눈다.**         
(사실 Heap 메모리 구조는 보다 세부적이지만 GarbageCollection 기준으로만 나타내면 아래와 같다.)        
        
<img width="512" alt="gc-heap" src="https://user-images.githubusercontent.com/50267433/131223789-4889ae62-89ca-486d-8287-0433c892a7c3.png">   

* Young Generation 
* Old Generation 

**hotspot heap structure**로 불리며 `Young Generation`과 `Old Generation`으로 나뉘어져 있다.                

```text  
위 그림은 JDK 1.8 을 기준으로 작성된 메모리 구조이며   
적용되는 메모리 길이를 조절할 수 있는 virtual이 있다.   
보다 자세한 설명은 뒤에서 언급하고자 한다.      
```
                
### 📄 Young 과 Old로 나눈 이유     

* young Generation은 자주 발생 + 삭제되는 데이터 많음    
* Old Generation은 간혹 발생 + 삭제되는 데이터 적음     

객체들은 생성 시간에 따라 비슷한 영역/지역에 뭉쳐져 있다.                  
하지만, 이러한 특징을 고려하지 않고 마킹을 위해 메모리를 풀스캔할 경우 이는 성능저하를 유발한다.    
       
* OLD + Young가 하나의 영역이라면 전체 영역을 순회해야함  
* OLD 한 객체들은 계속 살아있으므로 객체 그래프 탐색 지속적으로 이루어짐      
* 그래프 탐색 시간 길어짐에 따라 STW 도 길어지고 이로인해 성능 저하      
           
이를 해결하기 위해 Yound/Old 로 영역을 나누어 그래프 탐색 영역을 제한하는 방식을 택했다.         
그리고 Young 에서 메모리 할당과 해제가 더욱 빈번하게 이루어지기에 이를 Eden, Survival0/1 영역으로 나누었다.           
즉, 영역을 나누어 비교적 작은 영역을 순회하면서 STW의 시간을 줄이고 메모리내의 데이터 응집성도 높인다.                    
그리고 Young에 존재하는 객체가 일정 age 이상이 되면 Old로 빼네어 해당 영역만 GC를 수행토록한다.                       
         
이 외에도 Young 과 Old로 나누면서 얻게되는 이점으로      
비슷한 시기에 생성된 객체들은 서로간 참조가 될 가능성이 높은 객체들로 이들을 한데 모으면 관리가 용이하고       
단편화 측면에서도 영역이 큰 영역에 넓은 분포보다 작은 영역에서의 좁은 분포가 Compaction 에 유리하다.                 
               
## 📖 Young Generation      
Young Generation 은 비교적 최근에 생성된 객체들이 저장되는 공간이다.         
    
![image](https://user-images.githubusercontent.com/50267433/131223929-de5f8087-cd3d-49b7-9cfb-cc70e48ab331.png)
          
* **Eden :** 메모리에 새롭게 할당된 객체 대부분이 처음 위치하는 곳        
* **Survival 0/1 :** Eden 영역에서 GC가 한번 발생한 후 살아남은 객체들이 존재하는 곳  
            
Eden 영역이 포화 되면 `Minor GC`를 발생시켜              
UnReachable 객체를 메모리에서 해제하고 Reachable 객체를 Survival 0/1에 이동시킨다.               
이때 **Survival 0/1** 중에 이미 Reachable 객체가 있는 곳으로 객체를 보낸다.           
그리고 이 과정에서 Survival 0/1로 보내진 Reachable 객체들의 Age 값을 1증가시킨다.             
                     
## 📖 Old Generation     

![image](https://user-images.githubusercontent.com/50267433/131223961-481bf68a-f5e6-461d-94ac-f5386dde3a2c.png)
     
OldGeneration 영역이 포화 되면 `Major GC`를 발생시켜         
UnReachable 객체를 메모리에서 해제하고 Reachable 객체를 Survival 0/1에 이동시킨다.          


# 📘 GC Steps  
> GC가 동작하는 과정을 이야기해보고자 한다.    
  
대표적으로 GC는 아래와 같은 순서로 동작한다.
 
1. Marking      
2. Normal Deletion / eletion with Compacting   

위 과정을  Mark and Sweep 이라고도 부른다.        
Compacting 하는 과정은 GC 마다 다르다.      
   
## 📖 STW - (Stop the World)      
`Stop the world`란 Garbage Collection 과정을 수행하기 위해               
Garbage Collection 쓰레드를 제외한 모든 애플리케이션 쓰레드를 중지시키는 것을 의미한다.                 
           
이를 달리 말하자면 STW 가 발동함으로써 애플리케이션 쓰레드는 모두 Blcok(대기) 상태가 되며                   
이 시간이 길면 길어질 수록 애플리케이션의 성능을 떨어뜨리는 주요한 요소가 된다.                
반대로 말하면, **STW 시간을 줄이면 이는 곧 Garbage Collection 튜닝 중 하나가 된다는 의미기도 한다.**               
      
GC마다 STW가 발생하는 시기가 다르다.         
Mark and Sweep 전체 과정에서 STW 를 발생시키기도 하고       
도중 도중 STW를 풀었다 다시 발생시키기도 한다.  


## 📖 Marking
**GC 는 삭제할 객체와 삭제하지 않을 객체를 구분할 수 있는 것일까? 🤔**         
이에 대한 답으로 Marking 을 이야기할 수 있을 것 같다.      
       
![marking](https://user-images.githubusercontent.com/50267433/132496428-de939a22-ada7-42a2-ac98-34cb85826547.png)
       
             
Marking은 메모리에서 **사용되고 있는 객체**를 식별하는 과정이다.               
**객체의 참조 여부(마크)에 따라 객체가 사용중인지 판단한다.**             
    
* reachable 객체 : 참조가 되고 있는 객체  
* unreachable 객체 : 참조가 되고 있지 않는 객체 

Marking 즉, 객체 그래프 탐색하는 과정에서 `reachable 객체`에 마킹하여      
마킹이 되지 않은 `unreachable 객체`를 GC의 대상으로 한다.                  
        
## 📖 Normal Deletion 

![normal deletion](https://user-images.githubusercontent.com/50267433/132496436-d0d3f49c-f617-44ce-a12c-05c2bc8d6f30.png)


Normal Deletion은 Marking으로 찾아낸 unreachable 객체를 삭제하는 단계이다.             
삭제 후에 생기는 빈 공간은 Memory Allocator가 참조하고 있어서,      
메모리를 할당해야 할 일이 생기면 빈 공간을 찾아준다.          
     
   
## 📖 Deletion with compacting   
   
![deletion with compacting](https://user-images.githubusercontent.com/50267433/132496442-2fca2258-9c58-410c-ba72-12a0274a7632.png)
   
GC 수행 후 단편화 발생을 방지하고 효율적이고 빠른 메모리 사용을 위해 Compaction을 고려하는 경우도 있다.       
Compaction을 수행하면 메모리가 압축되어 Memory Allocator가 메모리를 할당하기 훨씬 쉬워진다.      
단, 모든 GC가 아닌 몇몇 GC 에서만 수행하므로 이점 참고하자.     

# 📗 JDK 1.8 이전과 JDK 1.8 이후  
## 📖 JDK 1.8 이전 Heap 메모리 구조 

```cmd
<----- Java Heap ----->             <--- Native Memory --->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Permanent | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
                        <--------->
                       Permanent Heap
```
      
* **Eden :** 새로 생성한 객체 대부분이 처음 위치하는 곳       
* **S0, S1:** Eden 영역에서 GC가 한번 발생한 후 살아남은 객체들이 존재하는 곳       
* **Old :** Young Generation에 대한 GC가 반복되는 과정속에 살아남은 객체가 살아남는 곳          
    * Age의 값이 특정 기준치를 달성하였을 때 `Young -> Old`로 이동한다.        
* **Perm :** `Class / Method` 의 **Meta 정보**, **static 변수/상수**들이 저장되는 곳      
   
## 📖 JDK 1.8 이후 Heap 메모리 구조 
```cmd
<----- Java Heap -----> <--------- Native Memory --------->
+------+----+----+-----+-----------+--------+--------------+
| Eden | S0 | S1 | Old | Metaspace | C Heap | Thread Stack |
+------+----+----+-----+-----------+--------+--------------+
```

* **Perm :** JVM에 의해 크기가 강제되던 영역(고정 크기)        
* **Metaspace :** Native memory 영역, OS가 자동으로 크기를 조절(가변 크기)        
    * 옵션으로 Metaspace의 크기를 줄일 수도 있다.
    * 기존과 비교해 큰 메모리 영역을 사용할 수 있게 되었다.
      
JDK 8부터 **Permanent 영역이 제거되었으며 대신 Metaspace 영역이 추가되었다.**                 
정확히는 Permanent 영역이 Metaspace로 바뀌었으며 Heap이 아닌 Native 영역에 포함하게 되었다.                     
  
## 📖 Permanent VS Metaspace
    
|구분|Permanent|Metaspace|
|---|---------|---------|
|저장 정보|클래스 meta / 메소드 meta / static 변수, 상수 |클래스 meta / 메소드 meta|
|관리 포인트|Heap 영역 튜닝 + Perm 영역 별도| Native 영역 동적 조정|
|GC|Full GC|Full GC|
|메모리 측면| -XX: PermSize / -XX: MaxPermSize | -XX: MetaSpaceSize / -XX: MaxMetaspaceSize |
       
가장 중요한 핵심은 **Perm 영역이 Heap 이 아니라 Native 영역으로 바뀌었다**는 점이다.       
**Native** 영역의 가장 큰 특징 중의 하나는 **JVM에 의해서 크기가 강제되지 않고,**             
**프로세스가 이용할 수 있는 메모리 자원을 최대로 활용 할 수 있다.**              
      
결과적으로 Permanent 영역 크기로 인한 `java.lang.OutOfMemoryError`를 보기 힘들어졌으며              
JEP 122에서는 JRockit과 Hotspot을 통일시키기 위해 Permanent 영역을 삭제한다고 한다.           
   
# 📙 Old 영역에 대한 GC
Old 영역은 기본적으로 데이터가 가득 차면 GC를 실행한다.         
여러 GC 방식마다 처리 절차가 다르기에 하나씩 짚고 넘어가자.              
  
* Serial GC
* Parallel GC
* Parallel Old GC(Parallel Compacting GC)
* Concurrent Mark & Sweep GC(이하 CMS)
* G1GC (Garbage First GC)   
* ZGC (추가 예정)     
     
이 중에서 운영 서버에서 절대 사용하면 안 되는 방식이 Serial GC다.    
Serial GC는 데스크톱의 CPU 코어가 하나만 있을 때 사용하기 위해서 만든 방식이다.   
Serial GC를 사용하면 애플리케이션의 성능이 많이 떨어진다.     
  
## 📖 Serial GC (-XX:+UseSerialGC) 
  
![serial-gc](https://user-images.githubusercontent.com/50267433/132518115-82057d6a-2441-4b42-9bdf-68914a6fab77.png)   
             
CPU 코어가 1개일 때 사용하는 방식으로 GC를 처리하는 스레드도 1개이다.         
`mark-sweep-compact`이라는 알고리즘을 사용하는데 앞서 언급했던 [GC Steps](#-gc-steps) 이다.             
     
Compact 과정에 주소 비교를 통해 누수된 객체를 만나면 하위 주소로 보낸다.(오름차순, 재배치 정보 추가)    
결과적으로 낮은 메모리 주소로 모든 누수된 객체를 압축시키는 작업을 진행한다.    
    
Serial GC는 적은 메모리와 CPU 코어 개수가 적을 때 적합한 방식이므로 현대에는 어울리지 않는다.   
        
## 📖 Parallel GC (-XX:+UseParallelGC) (Throughput GC)    

![parallelgc](https://user-images.githubusercontent.com/50267433/132518304-03c801d9-b438-46dd-8794-9023fa7d109f.png)

Parallel GC 또한 `mark-sweep-compact` 알고리즘을 사용한다.              
그러나 Parallel GC는 GC를 처리하는 쓰레드가 여러 개다.      
메모리가 충분하고 코어의 개수가 많을 때 사용되며 SerialGC 보다 빠르게 처리할 수 있다.     
      
## 📖 ConCurrent Mark Sweep GC (CMS GC (-XX:+UseConcMarkSweepGC)) (Low Latency GC)       
   
![cmsgc](https://user-images.githubusercontent.com/50267433/132518333-12966ad4-6299-4b79-8408-b50bd28ea5d5.png)
   
STW 시간을 단축하면서 속도가 중요한 애플리케이션에서 보다 효율적으로 사용하도록 만든 GC 이다.     

* initail Mark : 
    * 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝낸다.(STW 짧게)
    * Stack의 모든 변수를 스캔하면서 참조된 객체에 마킹한다. (검증 x)          
* Concurrent Mark :   
    * reachable 객체에서 객체 그래프 탐색을 이용해 다른 reachable 객체를 확인한다.     
    * 이 단계의 특징은 다른 스레드가 실행 중인 상태에서 동시에 진행된다는 것이다.(애플리케이션이랑 동시에 수행)    
* Remark : 
    * Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.    
    * 즉, Concurrent Mark가 애플리케이션이랑 동시에 수행되다보니 그 사이 새로 생긴 객체를 확인한다.     
* Concurrent Sweep :
    * unreachable 객체를 정리하는 작업을 실행한다. 
    * 이 작업도 다른 스레드가 실행되고 있는 상황에서 진행한다.
      
정리하자면 STW가 짧아서 애플리케이션의 응답 시간이 빨라야 할때 CMS GC를 사용한다.     
단, 다른 GC 방식보다 메모리 CPU를 더 많이 사용하며 Compaction 단계가 제공되지는 않는다.       
      
## 📖 G1GC   
> G1 GC를 이해하려면 지금까지의 Young 영역과 Old 영역에 대해서는 잊는 것이 좋다.     
   
G1GC는 `jdk11` 부터 공식적인 GC 알고리즘으로 적용되었고,     
하드웨어가 점점 발전하면서 대용량 메모리에 적합한 솔루션을 제공하기 위해 나타났다.      
      
![g1gc](https://user-images.githubusercontent.com/50267433/132506087-5598edec-21f1-48c1-8c71-522c531cabc2.PNG)
              
G1GC는 기존 Eden, Survivor, Old 영역이 존재하지만,            
해당 영역은 고정된 크기가 아니며 전체 Heap 메모리 영역을 Region 이라는 특정한 크기로 나누어져있다.            
Region의 상태에 따라 그 Region의 역할(Eden, Survivor, Old)가 동적으로 변동한다는 특징을 가지고 있으며               
GC가 일어날 때 Region만 조회함으로써 STW 시간을 줄이고 Compaction을 사용한다.             
Region은 기본적으로 (전체 Heap 메모리 ) / 2048 로 default 값이 지정되어 있습니다.             
                      
이 외에도 아래 두 영역이 추가되었다.            
* Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간           
* Available/Unused: 아직 사용되지 않은 Region          
      
![g1gc steps](https://user-images.githubusercontent.com/50267433/132517432-bd9e9d6e-10f0-44d2-9e41-bbeb93b4ddba.png)    
    
`G1GC` 에서도 마찬가지로 **Minor GC** 가 존재하며        
GC 과정에는 살아남은 객체들을 Survivor Region으로 옮기고                  
Eden에 대한 영역을 사용 가능한(Availabe)Region으로 돌리는 형태로 과정이 일어나게 된다.               
                        
반면 `G1GC` 에는 **Full GC** 와 유사한 **Concurrent Cycle** 이라는 과정이 존재하는데               
해당 과정은 `IHOP(InitiatingHeapOccupancyPercent)` 에서 정한 수치를 초과하면 실행하게 된다.              
                         
1. **Initial Mark :** Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾는다.(STW)                      
2. **Root Region Scan :** 위에서 찾은 Survivor 객체들에 대한 스캔 작업을 실시한다.           
3. **Concurrent Mark :** 전체 Heap의 scan 작업을 실시하고, GC 대상 객체가 발견되지 않은 Region은 이후 단계를 제외한다.             
4. **Remark :** 애플리케이션을 멈추고(STW) 최종적으로 GC 대상에서 제외할 객체를 식별한다.            
5. **Cleanup :** 애플리케이션을 멈추고(STW) 살아있는 객체가 가장 적은 Region에 대한 미사용 객체를 제거한다.           
6. **Copy :** GC 대상의 Region이었지만, Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을             
    새로운 Region(Available/Unused) Region에 복사하여 Compaction을 수행한다.              
7. 살아있는 객체가 아주 적은 Old 영역에 대해 `GC pause(mixed)` 를 로그로 표시하고, Young GC가 이루어질 때 수집되도록 한다.          
                
# 🙇‍♂️ 참고   
* [we-hate-jvm](https://github.com/Road-of-CODEr/we-hate-jvm/blob/master/GarbageCollection/README.md)       
* [JVM에 대해 알아보자](https://iann.tistory.com/17)   
* [Naver D2 - Garbage Collection](https://d2.naver.com/helloworld/1329)   
* [JVM이란? JVM 메모리구조](https://coding-start.tistory.com/205)   
* [Java의 GC는 어떻게 동작하나?](https://mirinae312.github.io/develop/2018/06/04/jvm_gc.html)   
* [JVM Garbage Collection](https://renuevo.github.io/java/garbage-collection/)   
* [oracle G1GC](https://www.oracle.com/technical-resources/articles/java/g1gc.html)   
* [Getting started with Z Garbage Collector in Java 11](https://hub.packtpub.comgetting-started-with-z-garbage-collectorzgc-in-java-11-tutorial/)    
* [JVM 과 HotspotVM 차이](https://stackoverflow.com/questions/16568253/difference-between-jvm-and-hotspot)     
* [Garbage Collection in Java](https://www.freecodecamp.org/news/garbage-collection-in-java-what-is-gc-and-how-it-works-in-the-jvm/)     
