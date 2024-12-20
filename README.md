# 동시성 제어 분석 보고서

## 1. 서론

### 1.1. 동시성 제어란?

&ensp;동시성 제어는 여러 프로세스 또는 스레드가 동시에 공유 자원에 접근하는 것을 제어하는 것을 말한다.
데이터 접근을 필요로 하는 다수의 동시 요청이 들어올 때 데이터 접근을 동시적으로 모두 허용해주면 데이터베이스의 일관성과 무결성이 깨지게됩니다. 그렇기에 동시성 제어를 하여 데이터 불일치, 경쟁 조건, 데드락 등의 문제를 해결하고 데이터 접근을 안전하게 제어하는 것이 필요합니다.

### 1.2. 목표

&ensp;본 보고서는 주요 동시성 제어 방식들의 개념, 특징, 장단점을 상세히 분석하는 것과 포인트 시스템에 적합한 동시성 제어 방식을 선택하는 것을 목표로 합니다.

## 2. 본론

### 2.1. 동시성 제어 방식 종류

-   **비관적 락 (Pessimistic Lock)**

    -   **특징**
        -   데이터 충돌이 **빈번하게 발생할 것이라고 가정**하고, 데이터를 읽기 전에 미리 락을 획득하는 방식입니다.
        -   데이터에 접근하는 즉시 락을 걸어 다른 트랜잭션의 접근을 차단합니다.
        -   주로 데이터 변경 충돌이 자주 발생하고, **데이터 무결성이 매우 중요한 환경**에서 사용됩니다.
        -   데이터베이스의 SELECT ... FOR UPDATE 문이나 파일 시스템의 파일 잠금 등이 대표적인 예입니다.
    -   **장점**
        -   **높은 데이터 무결성**: 데이터에 접근하기 전에 락을 획득하여 다른 트랜잭션의 접근을 막으므로, 데이터 무결성을 강력하게 보장할 수 있습니다.
        -   **충돌 감소**: 락을 통해 충돌을 사전에 방지하므로, 잦은 롤백으로 인한 성능 저하를 줄일 수 있습니다.
    -   **단점**
        -   **낮은 동시성**: 락으로 인해 여러 트랜잭션이 동시에 데이터에 접근할 수 없으므로, 동시성이 떨어져 시스템 성능이 저하될 수 있습니다.
        -   **데드락 발생 가능성**: 여러 트랜잭션이 서로 다른 자원에 대한 락을 획득하려 할 때 데드락이 발생할 수 있습니다.

-   **낙관적 락 (Optimistic Lock)**

    -   **특징**
        -   데이터 충돌이 **드물게 발생할 것이라고 가정**하고, 데이터를 읽을 때는 락을 획득하지 않고, 데이터를 변경할 때 충돌 여부를 확인하는 방식입니다.
        -   트랜잭션이 완료될 때까지 충돌을 감지하지 않다가, 커밋 시점에 충돌 여부를 확인합니다.
        -   주로 읽기 작업이 많고, **데이터 변경 충돌이 적은 시스템**에서 사용됩니다.
        -   버전 번호를 비교하거나, 타임스탬프를 비교하여 데이터 변경을 허용하거나 트랜잭션을 롤백하는 방식으로 구현됩니다.
    -   **장점**
        -   **높은 동시성**: 데이터를 읽을 때 락을 사용하지 않으므로, 락 경쟁을 피할 수 있고, 여러 트랜잭션이 동시에 데이터를 읽을 수 있어 동시성을 높일 수 있습니다.
        -   **향상된 성능**: 락 획득에 대한 오버헤드가 없어 락 경쟁으로 인한 성능 저하를 줄일 수 있으며, 시스템 처리량을 높일 수 있습니다.
    -   **단점**
        -   **잦은 롤백**: 충돌이 자주 발생하면 트랜잭션이 롤백되어 재시도해야 하므로, 롤백에 대한 비용이 많이 발생하고 성능 저하를 유발할 수 있습니다.
        -   **구현 복잡성**: 충돌 감지 로직과 롤백 처리 로직을 구현하는 것이 복잡할 수 있습니다.
        -   **사용자 경험 저하**: 충돌로 인한 롤백이 자주 발생하면 사용자 경험이 좋지 않을 수 있습니다.

-   **공유적 락 (Shared Lock)**

    -   **특징**
        -   데이터를 읽을 때 사용하며, 다른 트랜잭션의 **읽기 접근은 허용하지만 쓰기 접근은 차단**하는 락입니다.
        -   여러 트랜잭션이 동시에 데이터를 읽을 수 있으므로, **읽기 작업이 많은 환경**에서 성능을 높이는 데 사용됩니다.
        -   데이터베이스 시스템에서 주로 사용되며, SELECT ... LOCK IN SHARE MODE (PostgreSQL), SELECT ... FOR SHARE (Oracle) 등의 구문을 사용하여 획득합니다.
    -   **장점**
        -   **동시 읽기 성능 향상**: 여러 트랜잭션이 동시에 데이터를 읽을 수 있으므로, 시스템 전체의 읽기 처리량을 향상시킬 수 있습니다.
    -   **단점**
        -   **쓰기 작업 불가능**: 공유 락이 설정된 동안에는 데이터에 대한 쓰기 작업이 불가능하며, 쓰기 작업을 하려면 공유 락을 해제하고 배타적 락을 획득해야 합니다.
        -   **데이터 변경 시 배타적 락 필요**: 데이터를 변경해야 할 경우에는 공유 락을 배타적 락으로 전환해야 하므로, 락 전환 과정에서 성능 저하가 발생할 수 있습니다.

-   **배타적 락 (Exclusive Lock)**

    -   **특징**
        -   데이터를 쓸 때 사용하는 락으로, **다른 모든 트랜잭션의 접근을 차단**합니다.
        -   한 번에 하나의 트랜잭션만 락을 획득할 수 있으며, 해당 트랜잭션이 데이터를 읽거나 수정하는 동안 다른 트랜잭션은 대기해야 합니다.
        -   데이터의 무결성을 보장해야 하는 상황에서 사용되며, 데이터베이스에서 SELECT ... FOR UPDATE 구문을 사용하여 획득합니다.
    -   **장점**
        -   **데이터 무결성 보장**: 데이터에 대한 독점적인 접근을 보장하므로, 여러 트랜잭션이 동시에 데이터를 변경하는 것을 방지하여 데이터의 무결성을 보장할 수 있습니다.
    -   **단점**
        -   **락 경쟁으로 인한 성능 저하**: 여러 트랜잭션이 동시에 배타적 락을 획득하려고 하면 락 경쟁이 발생하여 대기 시간이 증가하고, 시스템 성능이 저하될 수 있습니다.
        -   **데드락 발생 가능성**: 여러 트랜잭션이 서로 다른 자원에 대한 배타적 락을 획득하려고 할 때 데드락이 발생할 수 있습니다.

-   **스핀락 (Spin Lock)**

    -   **특징**
        -   락을 획득하려고 시도할 때, **락을 얻을 때까지 계속해서 락 획득을 시도하는 락**입니다.
        -   락을 얻지 못하면 CPU를 계속해서 소비하며, 대기하는 동안 다른 작업을 수행하지 않습니다.
        -   주로 **짧은 시간 동안만 락을 사용하는 상황**에서 효율적입니다.
        -   멀티 프로세서 환경에서 주로 사용되며, 커널 수준의 동기화에 사용되기도 합니다.
    -   **장점**
        -   **컨텍스트 스위칭 오버헤드 감소**: 락을 얻기 위해 대기할 때, 스레드를 중지하고 다른 스레드로 컨텍스트 스위칭하는 비용을 줄일 수 있습니다. 락을 얻을 때까지 계속해서 CPU를 사용하며 기다리므로, 컨텍스트 스위칭으로 인한 오버헤드가 발생하지 않습니다.
        -   **빠른 락 획득**: 락을 짧은 시간 동안만 사용하는 경우, 컨텍스트 스위칭 없이 바로 락을 얻을 수 있어 성능 향상에 도움이 됩니다.
    -   **단점**
        -   **CPU 자원 낭비**: 락을 얻지 못하면 락을 얻을 때까지 계속해서 CPU를 사용하므로, CPU 자원을 낭비합니다. 만약 락을 오랫동안 획득하지 못하면 CPU 자원을 과도하게 소모하여 전체 시스템 성능 저하를 유발할 수 있습니다.
        -   **기아 상태**: 특정 스레드가 락을 계속해서 얻지 못하는 상황이 발생할 수 있으며, 이 경우 해당 스레드는 기아 상태에 빠져 영원히 실행되지 못할 수도 있습니다.
        -   **불필요한 락 획득 시도**: 락 경쟁이 심하거나, 락을 획득하는 데 오래 걸리는 경우, 불필요한 락 획득 시도가 계속해서 발생하여 오히려 성능 저하를 유발할 수 있습니다.
        -   **단일 프로세서 환경에 부적합**: 단일 프로세서 환경에서는 스핀락을 사용하면 락을 획득하지 못한 스레드가 CPU를 독점하기 때문에, 다른 스레드가 실행되지 못하여 시스템 전체가 멈출 수 있습니다.

-   **분산락 (Distributed Lock)**

    -   **특징**
        -   **여러 서버 인스턴스(분산 환경)에서 공유 자원에 대한 접근을 제어하는 락**입니다.
        -   일반적인 락은 단일 서버 환경에서만 동작하지만, 분산락은 여러 서버 환경에서 데이터의 일관성을 보장해야 할 때 사용됩니다.
        -   외부 시스템 (Redis, Zookeeper 등)을 사용하여 락을 구현합니다.
    -   **장점**
        -   **분산 환경에서 일관성 보장**: 여러 서버에서 동시에 공유 자원에 접근하는 상황에서 데이터의 일관성을 보장할 수 있습니다.
        -   **고가용성**: 외부 시스템을 사용하여 락을 구현하면, 단일 서버 장애가 발생하더라도 락 기능이 유지될 수 있습니다.
        -   **확장성**: 분산 환경에 맞춰 시스템을 확장할 수 있도록 도와줍니다.
    -   **단점**
        -   **복잡한 구현**: 분산락을 구현하기 위해서는 외부 시스템을 설정하고 관리해야 하므로, 구현이 복잡해질 수 있습니다.
        -   **네트워크 오버헤드**: 분산락을 사용하기 위해서는 네트워크 통신이 필요하며, 이로 인해 추가적인 지연 시간과 오버헤드가 발생할 수 있습니다.
        -   **단일 실패점**: 외부 시스템에 문제가 발생하면 분산락 기능 전체가 동작하지 않을 수 있습니다. 따라서 외부 시스템을 안정적으로 관리해야 합니다.
        -   **성능 이슈**: 네트워크 통신으로 인해 일반적인 락에 비해 락 획득 및 해제에 대한 성능이 떨어질 수 있습니다.
        -   **일관성 보장 어려움**: 분산 환경에서 일관성을 완전히 보장하는 것은 어려우며, 시스템 설계에 따라 일관성 수준이 달라질 수 있습니다.

-   **세마포어 (Semaphore)**

    -   **특징**
        -   **동시에 접근할 수 있는 스레드/프로세스 수를 제한하는 동기화 도구**입니다.
        -   내부적으로 카운터를 가지고 있으며, 이 카운터 값을 조작하여 공유 자원에 대한 접근을 제어합니다.
        -   카운터 값이 0보다 크면 스레드/프로세스가 자원에 접근할 수 있으며, 0이면 대기해야 합니다.
        -   주로 자원 풀링, 스레드 풀링과 같이 **자원 접근을 제한해야 하는 상황**에서 사용됩니다.
    -   **장점**
        -   **자원 접근 제어**: 동시에 접근할 수 있는 스레드/프로세스 수를 제한하여 시스템 과부하를 방지하고 안정성을 높일 수 있습니다.
        -   **유연성**: 다양한 동시성 제어 시나리오에 적용 가능하며, 카운터 값을 조절하여 접근 가능한 스레드/프로세스 수를 쉽게 변경할 수 있습니다.
        -   **공평성**: 스레드/프로세스가 자원을 요청한 순서대로 접근 권한을 부여하여 자원 접근의 공평성을 보장할 수 있습니다.
    -   **단점**
        -   **과도한 동시성 제어**: 동시에 접근할 수 있는 스레드/프로세스 수를 너무 제한하면 성능 저하가 발생할 수 있습니다.
        -   **구현 복잡성**: 세마포어의 초기화, acquire(자원 획득), release(자원 반환) 로직을 정확하게 구현해야 하며, 잘못 구현하면 데드락과 같은 문제가 발생할 수 있습니다.
        -   **오버헤드**: 세마포어를 사용하기 위한 acquire, release 등의 연산은 운영체제의 시스템 콜을 호출해야 하므로 오버헤드가 발생할 수 있습니다.

-   **뮤텍스 (Mutex, Mutual Exclusion)**
    -   **특징**
        -   **한 번에 하나의 스레드/프로세스만 접근할 수 있는 락**입니다.
        -   임계 영역(critical section)에 대한 접근을 상호 배제(mutual exclusion)하여, 데이터 경쟁을 방지합니다.
        -   **상호 배제**를 위한 락으로, 주로 공유 자원에 대한 배타적인 접근을 보장해야 하는 상황에 사용됩니다.
    -   **장점**
        -   **상호 배제**: 한 번에 하나의 스레드/프로세스만 접근할 수 있도록 하여 데이터 경쟁 상황을 방지하고, 데이터 무결성을 보장합니다.
        -   **간단한 사용법**: 락을 획득(acquire)하고 해제(release)하는 비교적 간단한 인터페이스를 제공하여 사용하기 쉽습니다.
        -   **동기화**: 여러 스레드/프로세스 간의 동기화를 손쉽게 구현할 수 있습니다.
    -   **단점**
        -   **락 경쟁으로 인한 성능 저하**: 여러 스레드/프로세스가 동시에 뮤텍스를 획득하려고 할 때 락 경쟁이 발생하여 대기 시간이 증가하고, 성능 저하가 발생할 수 있습니다.
        -   **데드락 발생 가능성**: 여러 뮤텍스를 사용할 때, 데드락 상황이 발생할 가능성이 있습니다.
        -   **과도한 동기화**: 임계 영역이 너무 크거나, 뮤텍스 사용이 너무 잦으면 성능 저하가 발생할 수 있습니다.
        -   **오버헤드**: 뮤텍스를 획득하고 해제하는 연산은 운영체제의 시스템 콜을 호출해야 하므로 오버헤드가 발생할 수 있습니다.
-   **다중 버전 동시성 제어 (MVCC, Multi-Version Concurrency Control)**
    -   **특징**
        -   **데이터를 여러 버전으로 관리하여 동시성 문제를 해결**하는 방식입니다.
        -   데이터를 변경할 때 **기존 데이터를 덮어쓰지 않고 새로운 버전을 생성**하며, 각 버전은 고유한 타임스탬프 또는 트랜잭션 ID를 가집니다.
        -   읽기 작업은 트랜잭션 시작 시점의 데이터 버전을 읽어 일관성을 유지합니다.
        -   주로 데이터베이스 시스템(PostgreSQL, Oracle, MySQL InnoDB 등)에서 사용됩니다.
    -   **장점**
        -   **높은 동시성**: 읽기 작업은 락을 획득하지 않고 특정 버전의 데이터를 읽기 때문에 락 경쟁으로 인한 성능 저하를 피할 수 있습니다.
        -   **읽기 일관성**: 트랜잭션은 시작 시점의 데이터 스냅샷을 읽기 때문에 항상 일관된 데이터 상태를 볼 수 있습니다.
        -   **락 오버헤드 감소**: 락 기반 제어 방식에 비해 락 획득 및 해제에 대한 오버헤드가 적어 시스템 성능을 향상시킬 수 있습니다.
    -   **단점**
        -   **추가적인 저장 공간 필요**: 여러 버전의 데이터를 관리해야 하므로, 추가적인 저장 공간이 필요합니다.
        -   **데이터 관리 복잡성 증가**: 오래된 버전의 데이터를 정리하는 작업(가비지 컬렉션)이 필요하며, 시스템의 복잡도가 증가합니다.
        -   **성능 저하 가능성**: 데이터 버전 관리가 잘못되면 읽기 성능이 저하될 수 있으며, 가비지 컬렉션 작업이 시스템 부하를 증가시킬 수 있습니다.
-   **타임스탬프 기반 동시성 제어 (Timestamp-Based Concurrency Control)**
    -   **특징**
        -   각 트랜잭션에 **고유한 타임스탬프를 부여**하여 트랜잭션의 작업 순서를 결정합니다.
        -   트랜잭션의 **읽기 및 쓰기 연산은 부여된 타임스탬프를 기준**으로 처리됩니다.
        -   타임스탬프 **불일치가 발견되면 트랜잭션을 중단(abort)하거나 롤백**합니다.
        -   락을 사용하지 않고 동시성을 제어하는 방식입니다.
    -   **장점**
        -   **데드락 방지**: 락을 사용하지 않기 때문에 락 기반 동시성 제어 방식에서 발생할 수 있는 데드락 문제를 해결할 수 있습니다.
        -   **낮은 오버헤드**: 락을 사용하지 않으므로, 락 획득 및 해제에 대한 오버헤드를 줄일 수 있습니다.
        -   **단순한 구현**: 락 기반 방식에 비해 구현이 비교적 단순하고, 락 관리 로직이 필요하지 않습니다.
        -   **공평한 처리**: 타임스탬프 순서대로 트랜잭션을 처리하므로, 트랜잭션이 불공평하게 처리될 가능성을 줄일 수 있습니다.
    -   **단점**
        -   **트랜잭션 중단 및 재시도**: 타임스탬프 불일치가 발생하면 트랜잭션을 중단하고 롤백해야 하며, 이러한 과정이 빈번하게 발생하면 시스템 성능이 저하될 수 있습니다.
            -   특히 쓰기 작업이 많은 환경에서는 트랜잭션 중단 및 재시도가 자주 발생할 가능성이 높습니다.
        -   **타임스탬프 관리 오버헤드**: 각 트랜잭션마다 타임스탬프를 관리해야 하고, 데이터 접근 시마다 타임스탬프를 검사해야 하므로 추가적인 오버헤드가 발생할 수 있습니다.
        -   **기아 상태**: 특정 트랜잭션이 계속해서 타임스탬프 불일치로 인해 중단되고 재시도되는 상황이 반복될 수 있으며, 이 경우 해당 트랜잭션은 기아 상태에 빠져서 계속 처리가 지연될 수 있습니다.
        -   **복잡한 충돌 처리**: 타임스탬프 불일치 감지 및 트랜잭션 중단/재시도 로직을 구현하는 것이 복잡할 수 있습니다.

### 2.2. 포인트 시스템에 적합한 동시성 제어 방식

&ensp;본 시스템에서는 **Mutex**를 사용하여 동시성 제어를 구현하였습니다. Mutex는 한 번에 하나의 스레드/프로세스만 공유 자원에 접근할 수 있도록 보장하므로, 포인트 잔액과 같은 민감한 데이터를 안전하게 관리할 수 있습니다. 또한, 구현이 비교적 간단하고 필요한 동기화 기능을 제공하므로, 시스템 복잡도를 줄이는 데 도움이 됩니다.

#### 초기 고민

&ensp;처음 동시성 제어에 대해 고민할 때, 모든 사용자 요청에 대한 동시성 제어를 고려해야 하는 것은 아닌지 생각했습니다. 즉, 동일한 사용자의 요청뿐만 아니라, 다른 사용자의 동시 요청까지 모두 고려해야 하는 것이 아닌가 하는 고민이 있었습니다. 하지만, 동시성 제어는 동일한 사용자의 데이터에 대한 접근에서 발생하는 문제를 해결하는 데 집중해야 한다는 것을 알게 되었습니다. 이에 따라, 본 시스템에서는 동일한 사용자의 요청에 대해서만 동시성 제어를 처리하는 것으로 구현 범위를 좁히게 되었습니다.

&ensp;또한, 이론적으로 완벽하게 동시에 여러 요청이 서버에 도착할 수 있지 않을까 하는 고민도 있었습니다. 하지만, 네트워크 지연, 서버 처리 시간 등 현실적인 여러 요소들을 고려했을 때, 수억 분의 1의 확률로 완벽하게 동시에 요청이 도착하는 경우는 사실상 불가능하다고 판단했습니다. 그럼에도 불구하고, 극히 낮은 확률로 완벽한 동시 요청이 발생할 수 있는 가능성을 배제할 수는 없기에, 이러한 상황에 대비하여 큐를 사용하여 포인트 관련 메소드 간의 우선순위를 정하여 처리하는 방안을 고려할 수 있다고 생각합니다.

#### Mutex 선택 이유

&ensp;이번 시스템에서 **Mutex**를 선택하게 된 이유는 단지 한 번에 하나의 스레드/프로세스만 접근해야 하고 구현이 간단했기 때문만은 아닙니다. 실제 데이터베이스를 사용하는 것이 아닌 인메모리 형식의 데이터베이스를 사용하기 때문에, 애플리케이션 레벨에서 사용자 레벨의 동시성 제어를 처리해야 하는 필요성이 있었습니다. 이러한 요구 사항을 충족하면서, 성능 오버헤드를 최소화하고 코드 복잡성을 줄이기 위해 **Mutex**를 선택하게 되었습니다.

&ensp;물론, **Mutex**를 사용할 때 락 경쟁으로 인한 성능 저하 같은 단점도 고려해야 합니다. 하지만, 포인트 잔액과 같은 민감한 데이터를 안전하게 관리하는 것이 더 중요하다고 판단하여 **Mutex**를 사용하기로 결정했습니다.

## 3. 결론

&ensp;본 보고서는 주요 동시성 제어 방식들의 개념, 특징, 장단점을 상세히 분석하고, 특정 포인트 시스템에 적합한 동시성 제어 방식을 제시하는 것을 목표로 하였습니다. 다양한 동시성 제어 방식들을 분석한 결과, 각 방식은 고유한 장단점을 가지고 있으며, 시스템의 요구사항과 환경에 따라 적합한 방식이 다르다는 것을 알 수 있었습니다.

### 3.1. 포인트 시스템에 대한 시사점

&ensp;본 보고서에서 분석한 다양한 동시성 제어 방식들은 각각 고유한 장단점을 가지고 있으며, 시스템의 요구 사항에 따라 적합한 방식이 다릅니다. 따라서 특정 시스템에 최적화된 동시성 제어 방식을 선택하기 위해서는 다양한 요소를 고려해야 합니다. 예를 들어 데이터 일관성, 성능, 구현 복잡성, 시스템 환경 등을 종합적으로 검토해야 합니다.

&ensp;본 보고서에서 분석한 결과, 포인트 시스템의 경우 데이터 무결성을 확보하고 구현 복잡도를 최소화하는 것이 중요하다고 판단하였습니다. 따라서 **Mutex**를 사용하여 동시성 제어를 구현하기로 결정했습니다. **Mutex**는 공유 자원에 대한 배타적인 접근을 보장하여 데이터 불일치 문제를 방지하고, 구현이 비교적 간단하여 시스템 복잡도를 줄일 수 있다는 장점이 있습니다.

&ensp;하지만 락 기반 동시성 제어 방식은 락 경쟁으로 인한 성능 저하나 데드락 발생 가능성과 같은 단점을 내포하고 있다는 점을 명심해야 합니다. 따라서 시스템 설계 시 임계 영역을 최소화하고 데드락 발생을 방지하기 위한 추가적인 로직을 구현할 필요가 있습니다.

&ensp;또한, 락 기반 제어 외에도 다양한 동시성 제어 방식(MVCC, 타임스탬프 기반 동시성 제어 등)에 대한 분석을 통해, 향후 시스템을 확장하거나 다른 동시성 제어 방식을 적용해야 할 상황에 대한 준비도 갖춰야 합니다.

### 3.2. 앞으로의 방향

-   **분산 환경에서의 동시성 공부 필요**: 현재 시스템은 단일 서버 환경이지만, 향후 분산 환경으로 확장할 때를 대비하여 분산 환경에서의 Redis를 사용한 분산락과 같은 동시성 제어 기술에 대한 공부도 필요합니다.

### 3.3. 마무리

-   이번 프로젝트를 통해, 동시성 제어 방식에 대한 이해를 높이고 실제 시스템에 적용할 수 있는 지식을 습득할 수 있었습니다. 또한, 다양한 동시성 제어 방식들을 분석하며 각 방식의 장단점을 이해하고, 포인트 시스템에 적합한 동시성 제어 방식을 선택하는 데 도움이 되었습니다.
