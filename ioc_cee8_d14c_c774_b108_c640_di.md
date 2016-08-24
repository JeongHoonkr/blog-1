# IoC 컨테이너와 DI
**핵심 : Singleton 빈이 주인 스프링에서는 원칙적으로 싱글톤 보다 작은 lifecycle을 가지는 빈을 DI하는 것이 의미가 없고 DL을 사용해야 한다는 것이 DI의 원칙이자 자바언어의 기본 sematics이다.**<br>

기본적으로 스프링의 빈은 싱글톤으로 만들어진다. 애플리케이션 컨텍스트마다 빈의 오브젝트는 한 개만 만들어진다는 뜻이다. 사용자의 요청이 있을 때마다 매번 애플리케이션 로직을 담은 오브젝트를 새로 만드는 건 비효율적이기 때문이다. 하나의 빈 오브젝트에 동시에 여러 스레드가 접근하기 때문에 상태 값을 인스턴스 변수에 저장해두고 사용할 수 없다. 따라서 싱글톤의 필드에는 의존관계에 있는 빈에 대한 레퍼런스나 읽기전용 값만 저장해두고 오브젝트의 변하는 상태를 저장하는 인스턴스 변수는 두지 않는다.<br>

그런데 때로는 빈을 싱글톤이 아닌 다른 방법으로 만들어 사용해야 할 때가 있다. 빈 당 하나의 오브젝트만을 만드는 싱글톤 대신, 하나의 빈 설정으로 여러 개의 오브젝트를 만들어서 사용하는 경우다.<br>

cf) scope : 존재할 수 있는 범위를 가리키는 말이다. 빈의 스코프는 빈 오브젝트가 만들어져 존재할 수 있는 범위다. 빈 오브젝트의 생명주기는 스프링 컨테이너가 관리하기 때문에 대부분 정해진 범위(스코프)의 끝까지 존재한다. 싱글톤 스코프는 컨테이너 스코프라고 하기도 한다. 단일 컨테이너 구조에서는 컨테이너가 존재하는 범위와 싱글톤이 존재하는 범위가 일치하기 때문이다. 요청(request) 스코프는 하나의 요청이 끝날 때까지만 존재한다.<br>

싱글톤 스코프는 컨텍스트당 한 개의 빈 오브젝트만 만들어지게 한다. 따라서 하나의 빈을 여러 개의 빈에서 DI 하더라도 매번 동일한 오브젝트가 주입된다. DI 설정으로 자동주입하는 것 말고 컨테이너에 getBean() 메서드를 사용해 DL 하더라도 매번 같은 오브젝트가 리턴됨이 보장된다.<br>

##프로토타입 스코프
프로토타입 스코프는 컨테이너에게 빈을 요청할 때마다 매번 새로운 오브젝트를 생성해준다. DI, DL 상관없이 매번 새로운 오브젝트가 만들어진다.<br>

###프로토타입 빈의 생명주기와 종속성
IoC의 기본 개념은 애플리케이션을 구성하는 핵심 오브젝트를 코드가 아니라 컨테이너가 관리한다는 것이다. 그래서 스프링이 관리하는 오브젝트인 빈은 그 생성과 다른 빈에 대한 의존관계 주입, 초기화, DI와 DL을 통한 사용, 제거에 이르기까지 모든 오브젝트의 생명주기를 컨테이너가 관리한다. 빈에 대한 정보와 오브젝트에 대한 레퍼런스는 컨테이너가 계속 갖고 있고 필요할 때마다 요청해서 빈 오브젝트를 얻을 수 있다.<br>

그런데 프로토타입 빈은 독특하게 이 IoC의 기본 원칙을 따르지 않는다. 프로토타입 스코프를 갖는 빈은 요청이 있을 때마다 컨테이너가 생성하고 초기화하고 DI까지 해주기도 하지만 일단 빈을 제공하고 나면 컨테이너는 더 이상 빈 오브젝트를 관리하지 않는다. 따라서 프로토타입 빈 오브젝트는 한번 DL이나 DI를 통해 컨테이너 밖으로 전달 되면 그 후부터는 더 이상 스프링이 관리하는 빈이 아니게 된다. 이때부터는 DL을 통해서 오브젝트를 가져간 코드나 DI로 주입받은 다른 빈이 사실상 컨테이너가 제공한 빈 오브젝트를 관리하게 된다.한번 만들어진 프로토타입 빈 오브젝트는 다시 컨테이너를 통해 가져올 방법이 없고, 빈이 제거되기 전에 빈이 사용한 리소스를 정리하기 위해 호출하는 메서드도 이요할 수 없다.<br>

프로토타입 빈은 컨테이너가 초기 생성 시에만 관여하고 DI 한 후에는 더 이상 신경 쓰지 않기 때문에 빈 오브젝트의 관리는 전적으로 DI 받은 오브젝트에 달려 있다. 그래서 프로토타입 빈은 이 빈을 주입받은 오브젝트에 종속적일 수밖에 없다. 프로토타입 빈을 주입받은 빈이 싱글톤이라면, 이 빈에 주입된 프로토타입 빈도 역시 싱글톤 생명 주기를 따라서 컨테이너가 종료될 때까지 유지될 것이다. 프로토타입 빈을 DI 받은 빈의 스코프가 더 작아서 일찍 제거돼야 한다면, DI 된 프로토타입 빈도 함께 제거될 것이다. 만약 DL 방식으로 직접 컨테이너에 getBean() 메서드를 통해서 프로토타입 빈을 요청했다면, 그 요청한 코드가 유지시켜주는 만큼 빈 오브젝트가 존재할 것이다. 메서드 안에서 사용하고 따로 저장해두지 않는다면, 메서드가 끝나면서 프로토타입 빈 오브젝트도 함께 제거된다.

###프로토타입 빈의 용도
사용자의 요청에 따라 매번 독립적인 오브젝트를 만들어야 하는데, 매번 새롭게 만들어지는 오브젝트가 컨테이너 내의 빈을 사용해야 하는 경우가 있다. DI가 필요한 오브젝트라는 뜻이다. 오브젝트에 DI를 적용하려면 컨테이너가 오브젝트를 만들게 해야 한다. 바로 이런 경우에 프로토타입 빈이 유용하다. 프로토타입 빈은 오브젝트의 생성과 DI 작업까지 마친 후에 컨테이너가 돌려준다.<br>

콜센터에서 고객의 A/S 신청을 받아서 접수하는 기능을 만든다고 생각해보자. 이때 등록 폼에서 고객번호를 입력받는다. 이렇게 입력받은 고객번호는 다른 입력 필드와 함께 폼 정보를 담는 오브젝트에 담겨서 서비스 계층으로 전달되어 A/S 신청 접수 기능에서 사용될 것이다.
```
A/S 신청 폼 클래스

public class ServiceRequest {
  String customerNo;
  String productNo;
  String description;
}
```
ServiceRequest의 오브젝트는 매번 신청을 받을 때마다 새롭게 만들어지고, 폼의 정보를 담아서 서비스 계층으로 전달될 것이다. 웹 요청을 받아 처리하는 웹 컨트롤러에서는 아래와 같이 매번 new 연산자로 ServiceRequest 클래스의 오브젝트를 생성하고, 폼 요청 정보를 넣은 뒤 서비스 계층으로 전달해줘야 한다.
```
ServiceRequest 웹 컨트롤러 

public void serviceRequestFormSubmit (HttpServletRequest request) {
    ServiceRequest serviceRequest = new ServiceRequest(); // 매 요청마다 새로운 객체를 생성한다.
    serviceRequest.setCustomerNo(request.getParameter("custno"));
    ...
    this.serviceRequestService.addNewServiceRequest(serviceRequest);
    ...
}
```
이 웹 컨트롤러는 매우 단순하고 원시적이다. 스프링의 웹 프레임워크를 사용하면 훨씬 세련되고 깔끔하게 만들 수 있지만, 일단은 어떤 식으로 동작하는지 설명하기 위한 코드라고 생각하고 보자. 일단 여기까지는 아무런 문제가 없다. 폼으로부터 요청이 있을 때마다 새로운 오브젝트를 만들고 폼의 필드에 입력된 고객번호를 저장하는 것은 자연스러운 일이다. <br>

이번엔 서비스 계층의 구현을 살펴보자. 콜 센터의 업무를 담당하는 서비스 오브젝트에서는 새로운 A/S 요청이 접수되면 접수된 내용을 DB에 저장하고 신청한 고객에게 이메일로 접수 안내 메일을 보내주도록 되어 있다. 폼에서는 단지 문자열로 된 고객번호를 받았을 뿐이지만 CustomerDao에게 요청하면 고객정보를 모두 가져올 수 있다. CustomerDao에서 가져온 고객정보는 Customer 오브젝트에 담겨 있을 것이고, 이를 이용해 이메일을 발송할 수도 있다. 서비스 계층의 ServiceRequestService 클래스에는 아래와 같은 코드가 만들어질 것이다.
```
ServiceRequest 서비스 계층
public void addNewServiceRequest(ServiceRequest serviceRequest) {
  Customer customer = this.customerDao.findCustomerByNo(serviceRequest.getCustomberNo());
  ...
  this.serviceRequestDao.add(serviceRequest, customer);
  
  this.emailService.sendEmail(customer.getEmail(), 
    "A/S 접수가 정상적으로 처리되었습니다.");
}
```
이런 코드가 자연스럽게 느껴질지도 모르겠다. ServiceRequest를 단지 폼의 정보를 전달해주는 DTO와 같은 데이터 저장용 오브젝트로 취급하고, 그 정보를 이용해 실제 비지니스 로직을 처리할 때 필요한 정보는 다시 서비스 계층의 오브젝트가 직접 찾아오게 만드는 것이다.
![](오브젝트사용방식.jpg)
위 그림은 ServiceRequest가 폼의 정보를 담고 사용되는 구조를 나타낸다. 코드에서 new로 생성하는 ServiceRequest를 제외한 나머지 오브젝트는 스프링이 관리하는 싱글톤 빈이다. 이 방식의 장점은 처음 설계하고 만들기는 편하다는 것이다. 웹 페이지의 등록 폼에서 어떤 식으로 사용자 정보가 입력될지를 미리 정해두고, 그 입력 방식에 따라서 컨트롤러와 서비스 오브젝트까지 만들면 된다. 서비스 오브젝트는 폼에서 문자열로 입력된 고객번호가 ServiceRequest 오브젝트에 담겨 전달된다는 사실을 미리 알고 있다.<br>

문제는 폼의 고객정보 입력 방법이 모든 계층의 코드와 강하게 결합되어 있다는 점이다. 만약 고객정보를 텍스트로 입력받는 대신 AJAX를 써서 이름을 이용한 자동완성 기능을 이용한 후에 Customer 테이블의 id를 폼에서 전달하는 식으로 바뀌면 어떻게 될까? ServiceRequest의 필드와 이를 처리하는 컨트롤러는 물론이고, A/S 서비스 신청을 처리하는 서비스 오브젝트인 ServiceRequestService의 코드도 다음과 같이 id 값을 이용해 Customer 오브젝트를 가져오는 방법으로 수정돼야 할 것이다.
```
Customer customer =  this.customerDao.getCustomer(serviceRequest.getCustomerId());
```
이는 전형적인 데이터 중심의 아키텍처가 만들어내는 구조다. 비록 ServiceRequest 오브젝트에 폼 정보가 담겨 있긴 하지만, 도메인 모델을 반영하고 있다고 보기 힘들다. 모델 관점으로 보자면 서비스 요청 클래스인 ServiceRequest는 Customer라는 고객 클래스와 연결되어 있어야지, 폼에서 어떻게 입력받는지에 따라 달라지는 customerNo나 customerId 같은 값에 의존하고 있으면 안된다.<br>

그렇다면 이 구조를 좀 더 오브젝트 중심의 구조로 만들고, 좀 더 객체지향적으로 바꾸려면 어떻게 해야 할까? 일단 웹 컨트롤러는 같은 웹 프레젠테이션 계층의 뷰에서 만들어주는 폼과 밀접하게 연결되어 있는 것이 자연스럽고 별문제가 되지 않는다. 대신 서비스 계층의 ServiceRequestService는 ServiceRequest 오브젝트에 담긴 서비스 요청 내역과 함께 서비스를 신청한 고객정보를 Customer 오브젝트로 전달받아야 한다. 그래야만 프레젠테이션 계층의 입력 방식에 따라서 비지니스 로직을 담당하는 코드가 휘둘리지 않고 독립적으로 존재할 수 있다. 따라서 ServiceRequest를 다음과 같이 변경해야 한다.
```
public class ServiceRequest {
  Customer customer;
  String productNo;
  String description;
  ...
}
```
ServiceRequest는 customerNo 값 대신 Customer 오브젝트 자체를 참조하게 한다. ServiceRequest가 좀 더 도메인 모델에 가깝게 만들어졌으니, 서비스 계층의 코드는 다음과 같이 바꿀 수 있다.
```
수정된 서비스 계층 코드

public void addNewServiceRequest(ServiceRequest serviceRequest) {
  this.serviceRequestDao.add(serviceRequest);
  this.emailService.sendEmail(serviceRequest.getCustomer().getEmail(),
    "A/S 접수가 정상적으로 처리되었습니다.");
}
```
폼에서 입력받은 고객번호로 고객을 찾아오는 번거로운 작업을 생략할 수 있게 됐다. serviceRequestDao에도 ServiceRequest 타입의 오브젝트만 전달하면 된다. DAO가 A/S 신청정보를 저장할 때 필요한 id와 같은 고객정보는 ServiceRequest의 customer 필드를 통해 가져올 수 있다. DAO는 물론이고 서비스 오브젝트도 폼의 입력방식에서 완전히 자유로워졌다.<br>

그러나 아직 해결해야 할 가장 큰 문제가 남아 있다. 폼에서는 문자열로 된 고객번호를 입력받을 텐데 그것을 어떻게 Customer 오브젝트로 바꿔서 ServiceRequest에 넣어 줄 수 있을까? 답은 간단하다. customerNo를 가지고 CustomerDao에 요청해서 Customer 오브젝트를 찾아오면 된다. 이전에는 그것을 ServiceRequestService의 메서드에서 처리했는데, 이제는 어디서 해야 할까? 일단 생각해볼 수 있는 건, 웹 컨트롤러에서 CustomerDao를 사용해 Customer를 찾은 뒤에 이를 ServiceRequest에 전달하는 것이다. 이것도 그리 나쁜 방법은 아니다. 하지만 그보다 나은 방법은 ServiceRequest 자신이 처리하는 것이다.<br>

만약 ServiceRequest가 CustomerDao에 접근할 수 있다면 어떨까? 그렇다면 다음과 같이 ServiceRequest 코드를 만들 수 있다.
```
Customer를 검색할 수 있는 기능을 가진 ServiceRequest

public class ServiceRequest {
  Customer customer;
  ...
  @Autowired
  CustomerDao customerDao; 
  
  public void setCustomerByCustomerNo(String customerNo) {
    this.customer = customerDao.findCustomerByNo(customerNo);
  }
}
```
ServiceRequest가 CustomerDao를 DI 받아서 사용할 수 있다면 문제는 간단해진다. 폼에서 고객번호를 입력받았다면 웹 컨트롤러에서는 setCustomerByCustomerNo() 메서드를 통해 ServiceRequest 오브젝트에 전달해주기만 하면 된다. 이렇게 하면 ServiceRequestService는 ServiceRequest의 customer 오브젝트가 어떻게 만들어졌는지에 대해서는 전혀 신경쓰지 않아도 된다. 단지 A/S 신청정보에는 그것을 신청한 고객정보가 도메인 모델을 따르는 오브젝트로 만들어져 있으리라 기대하고 사용할 뿐이다.<br>

폼에서 입력받는 것이 고객번호가 아니라 고객검색 팝업이나 AJAX를 통해 구한 고객의 ID라면, 다음과 같은 메서드를 ServiceRequest에 추가해주고 컨트롤러를 통해 id 값을 넣어주게만 하면 그만이다.
```
public void setCustomerByCustomerId(int customerId) {
  this.customer = this.customerDao.getCustomer(customerId);
}
```
폼에서 고객정보를 입력받는 방법을 어떻게 변경하든 ServiceRequest를 사용하는 서비스 계층이나 DAO의 코드는 전혀 영향을 받지 않는다. 이제 남은 문제는 컨트롤러에서 new 키워드로 직접 생성하는 ServiceRequest 오브젝트에 어떻게 DI를 적용해서 Customer