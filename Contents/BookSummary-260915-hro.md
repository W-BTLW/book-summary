# Chapter 11. 자주쓰는 서버 구조와 설계 패턴

### MVC 패턴
- Model-View-Controller
  - Model : 비즈니스 영역의 로직 처리
  - View : 사용자가 보게 될 결과를 생성해서 응답
  - Controller : 사용자 입출력 처리 및 흐름제어
- MVC 패턴의 핵심
  - 비즈니스 로직을 처리하는 모델 / 결과를 생성하는 뷰 를 분리
  - 흐름 제어나 사용자 요청 처리는 컨트롤러에 집중
- 뷰와 모델은 컨트롤러에 의존하지 않는다. 컨트롤러만 모델과 뷰를 의존한다. -> 유지보수 용이

<br/>

### 계층형 아키텍처
- Layered Architecture
- 각 계층마다 특정 역할을 수행하고, 상위 계층에서 하위 계층으로의 의존만 허용한다
- 표현,UI(컨트롤러, 뷰) -> 응용(서비스) -> 도메인/모델(상태변경, 제약조건) -> 인프라/영속(DB연동)
- 구조가 단순하고 규칙이 명확해서 코드 실행 흐름을 추적하기 쉽다

<br/>

### DDD와 전술패턴
- Domain-Driven Design
- DDD 구성요소
  - Entity : 고유 식별자
  - Value : 개념적인 값 (ex. 금액, 배송 주소)
  - Aggregate : 관련된 객체를 묶어 하나의 개념적인 단위를 표현
  - Repository : 도메인 객체를 저장하고 조회할 때 사용되는 인터페이스
  - Domain Service : 특정한 Aggregate에 속하지 않은 로직 구현
  - Domain Event : 도메인 내에서 발생한 이벤트를 표현
- 도메인 로직을 Aggregate 단위로 묶어서 관리하여 복잡도를 낮추고, 로직 응집도를 높인다
``` java
public class CancelOrderService {
  private OrderRepository orderRepository;

  @Transactional
  public void cancel(OrderNumber orderNum) {
    // 응용 서비스는 도메인 모델을 사용해서 사용자 요청을 처리한다.
    Optional<Order> orderOpt = orderRepository.findById(orderNum);
    Order order = orderOpt.orElseThrow(() -> new NoOrderException());
    // 주문 취소 로직은 Order 애그리거트에 위치한다.
    order.cancle();
  }
}
```

<br/>

### 마이크로서비스 아키텍처
- 더 작은 단위로 서비스를 분리하고 각 서비스가 연동되는 구조.
- 독립적 배포가 가장 중요하다
- 각 마이크로서비스 간 결합도를 최대한 낮춰야

<br/>

||모놀리식|마이크로서비스
|---|---|---|
|장점| 배포 단순 | 독립적인 배포와 지속적인 배포가 용이 |
|| 코드 관리 쉬움 | 성능 확장 용이 |
|| 성능을 높이기 위해 복잡한 구조를 가질 필요가 없음 | 기술에 대한 유연성을 가질 수 있음 |
|| 테스트와 디버깅 쉬움 | 개발자 만족도 높음 |
|단점| 구모가 커질수록 개발 속도가 느려질 수 있음 | 테스트와 디버깅 어려움 |
| | 한 기능의 문제가 전체에 영향을 줄 수 있음 | 모놀리식 대비 인프라가 복잡해짐 |
| | 구현 기술 변경이 어려움 | 소통에 따른 부하 증가 |
| | 작은 변경도 전체 재배포 필요 | 분산 모놀리식이 될 수 있음 |

<br/>

### 이벤트 기반 아키텍처
- 두 시스템 간에 통신할 때 이벤트를 사용하는 구조
- 구성 요소
  - 이벤트 생산자
  - 이벤트 소비자
  - 이벤트 브로커(or 라우터)
- 생산자와 소비자 직접 연결되지 않고 브로커를 통해 간접적으로 연결되어, 독립적인 배포가 가능하다

<br/>

### CQRS 패턴
- Command Query Responsibility Segregation
- 명령(상태변경)을 위한 모델과 조회를 위한 모델을 분리하는 패턴
- 조회 성능을 향상시킬 수 있다 (캐시 적용, 조회전용 DB 확장)
- But, 각 기능마다 모델을 따로 만들어야 하므로 작업해야 할 코드가 늘어난다.
- 구현 기술이 늘어날 수 있다. 
