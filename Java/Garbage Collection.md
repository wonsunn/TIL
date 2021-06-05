# Garbage Collection

## Garbage Collector란

- '더이상 참조되지 않는 메모리'인 가비지를 정리해주는 JVM 내 실행 엔진의 한 요소.
- Heap 영역 위주로 탐색하여 메모리를 정리해준다.

## Garbage Collection의 과정

Java에서는 개발자가 프로그램 코드로 메모리를 명시적으로 해제하지 않기 때문에 가비지 컬렉터(Garbage Colletor)가 더이상 필요없는 객체를 찾아 지우는 작업을 한다. 이 가비지 컬렉터는 두 가지 가설(전제 조건) 하에 만들어졌다. 이를 '**weak generational hypothesis**'라 한다.

- 대부분의 객체는 금방 접근 불가능한 상태(unreachable)가 된다.
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

이 가설의 장점을 최대한 살리기 위해 HotSpot VM에서는 크게 2개로 물리적 공간을 나누었다. 둘로 나눈 공간이 Young 영역과 Old 영역이다.

- Young 영역 : 새롭게 생성한 객체들이 위치한다. 대부분의 객체는 금방 접근 불가능한 상태가 되기 때문에, 많은 객체가 이 영역에서 생성되었다가 사라진다.
- Old 영역 : Young 영역에서 계속 사용되어 살아남은 객체가 복사되는 영역이다. Young 영역보다 크게 할당되며, 더 적은 GC가 발생한다.

**그렇다면, 두 번째 가설과 관련 있는 Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우가 있을 때에는 어떻게 처리될까?**
![image](https://user-images.githubusercontent.com/47625368/120897622-110b9b00-c662-11eb-876f-33c93eb24713.png)
이러한 경우를 처리하기 위해,

- 위의 그림과 같이 Old 영역에 512바이트의 덩어리(chunk)로 되어 있는 카드 테이블이 존재한다.
- 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 정보가 표시된다.
- Young 영역의 GC를 실행할 때 Old 영역에 있는 모든 객체의 참조를 확인하지 않고, 이 카드 테이블만 뒤져서 GC 대상인지 식별한다.

### Young 영역에서의 가비지 컬렉션(GC) - Minor GC

먼저 Young 영역의 구성을 보자면,

- Eden 영역
- Survivor 영역 2개

GC 작동 과정

1. 새로운 객체가 Eden 영역에 생성된다.
2. Eden 영역에 GC가 동작하고, 그 중에서 살아남은 객체가 Survivor0으로 이동한다.
3. 2번의 동작이 반복되어 Survivor0이 꽉차게 된다.
4. Survivor0 영역에 GC가 동작하고, 살아남은 객체들은 Survivor1로 이동하며, Survivor0을 비우게 된다.(이때 2개의 Survivor 영역 중 1개는 반드시 비어있어야 한다.)
5. 위의 동작들이 반복되어 특정 횟수만큼 살아남은 객체는 Old 영역으로 이동한다.

![image](https://user-images.githubusercontent.com/47625368/120887174-58773480-c62c-11eb-86a3-80368bfac6a5.png)

### Old 영역에서의 가비지 컬렉션(GC) - Major GC

Old 영역은 기본적으로 데이터가 가득 차면 GC를 실행한다. GC 방식에 따라서 처리 절차가 달라진다. GC 방식은 JDK7을 기준으로 5가지 방식이 있다.

- **Serial GC**
  - mark-sweep-compact 알고리즘을 사용한다.
  - Old 영역에서 살아있는 객체를 식별(Mark)하고, 살아있는 객체만을 남긴다.(Sweep) 이후에 객체들을 앞부분부터 채워 객체가 존재하는 부분과 존재하지 않는 부분으로 나눈다.(Compaction)
- **Parallel GC**
  - 기본적인 알고리즘은 Serial GC와 같지만 여러 쓰레드를 이용하여 GC를 처리한다.
- **Parallel Old GC(Parallel Compacting GC)**
  - Serial GC의 Sweep 알고리즘 대신 Summary를 사용한다.
  - Summary 단계는 앞서 GC를 수행한 영역에 대해서 별도로 살아있는 객체를 식별하며, Sweep보다 조금 더 복잡하다.
- **Concurrent Mark & Sweep GC(CMS)**
  ![image](https://user-images.githubusercontent.com/47625368/120899577-57b1c300-c66b-11eb-955c-f202e07e694c.png)
  - 초기 **Initial Mark 단계**에서는 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝내기 때문에, 멈추는 시간(Stop-the-World)이 짧다.
  - 그리고 **Concurrent Mark 단계**에서는 방금 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인하다.(다른 쓰레드가 실행 중인 상태에서 동시에 진행됨)
  - 다음 **Remark 단계**에서는 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
  - 마지막으로, **Concurrent Sweep 단계**에서 가비지 객체를 정리하는 작업을 실행한다.
  - **Stop-the-World 시간이 매우 짧기 때문에 모든 애플리케이션의 응답 속도가 중요할 때 사용된다.**
- **G1(Garbage First) GC**
  - 바둑판의 각 영역에 객체를 할당하고 GC를 실행한다.
  - Young 영역과 Old 영역에 대한 개념을 사용하지 않고, 객체를 할당한다.
  - 가장 성능이 좋은 방식
