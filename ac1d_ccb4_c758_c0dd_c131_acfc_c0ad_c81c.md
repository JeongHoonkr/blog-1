# 객체의 생성과 삭제
객체를 만들어야하는 시점과 그 방법, 객체 생성을 피해야 하는 경우와 그 방법, 적절한 순간에 객체가 삭제되도록 보장하는 방법, 그리고 삭제 전에 반드시 이루어져야 하는 청소 작업들을 관리하는 방법을 살펴본다. 

###규칙1 : 생성자 대신 정적 팩터리 메서드를 사용할 수 없는지 생각해보라

클래스를 통해 객체를 만드는 일반적인 방법(public 생성자 이용)말고 또 다른 방법이 있다. 바로 **public static factory method**를 만드는 것이다. 
```
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

**첫 번째 장점은, 생성자와는 달리 정적 팩토리 메서드에는 이름(name)이 있다.**<br>
생성자에 전달되는 인자(parameter)들은 어떤 객체가 생성되는지를 설명하지 못하지만, 정적 팩토리 메서드는 이름을 잘 짓기만 한다면 사용하기 쉽고, 클라이언트 코드의 가독성도 높아진다. 예를들어, 소수일 가능성이 높은 BigInteger 객체를 생성하는 생성자 BigInteger(int, int, Random)는 BigInteger.probablePrime과 같은 이름의 정적 팩토리 메서드로 표현했으면 더 이해하기 쉬웠을 것이다.(이 메서드는 JDK 1.4 버전에 결국 추가됨) 

같은 시그너처(메서드의 형태가 같은)를 갖는 생성자를 여러 개 정의할 필요가 있을 때는 그 생성자들을 정적 팩토리 메서드로 바꾸고, 메서드 이름을 보면 차이가 명확히 드러나도록 작명에 신경쓰자. 

**두 번째 장점은, 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요가 없다.**<br><br>
**세 번째 장점은, 생성자와는 달리 반환값 자료형의 하위 자료형 객체를 반환할 수 있다**<br>


JDBC와 같은 서비스 제공자 프레임워크의 근간을 이루는 것이 바로 유연한 정적 팩토리 메서드들이다.
![](serviceproviderframework.PNG)
서비스 제공자 프레임워크는 세 가지의 핵심 컴포넌트로 구성된다. 
1. 서비스 인터페이스 : Connection (서비스 제공자가 구현한다.)
2. 제공자 등록 API : DriverManager.registerDriver (구현체를 시스템에 등록하여 클라이언트가 쓸수 있도록 한다.)
3. 서비스 접근 API : DriverManager.getConnection(클라이언트에게 실제 서비스 구현체를 제공한다.)
4. 서비스 제공자 인터페이스(옵션) : Driver (서비스 제공자가 구현하고 서비스 구현체의 객체를 생성하기 위한 것이다. 서비스 제공자 인터페이스가 없는 경우 구현체는 클래스 이름으로 등록되며 자바의 리플렉션 기능을 통해 객체로 만들어진다.)


```
Connection conn = null;  

try{
    String url = "jdbc:mysql://localhost:3306/jdbcTest";
    String id = "testid";                                   
    String pw = "testpw";                                               
    Class.forName("com.mysql.jdbc.Driver");

    conn=DriverManager.getConnection(url,id,pw); 
    
    ...
```

```
//서비스 제공자 인터페이스의 대략적인 모습

//서비스 인터페이스 ex)Connection
public interface Service{
    ... //서비스에 고유한 메서드들이 이 자리에 온다. 
}

//서비스 제공자 인터페이스 ex) Driver (서비스 구현체의 객체를 생성하기 위한것) 
public interface Provider{
    Service newService();
}

//서비스 등록과 접근에 사용되는 객체 생성 불가능 클래스 ex) DriverManager
public class Services{
    private Services() {}
    
    //서비스 이름과 서비스 간 대응관계 보관
    private static final Map<String, Provider> providers = 
        new ConcurrentHashMap<String, Provider>();
    public static final String DEFAULT_PROVIDER_NAME = "<def>";
    
    //제공자 등록 API ex) DriverManager.registerDriver
    public static void registerDefaultProvider(Provider p){
        registerProvider(DEFAULT_PROVIDER_NAME, p);
    }
    
    public static void registerProvider(String name, Provider p){
        providers.put(name,p);
    }
    
    //서비스 접근 API ex) DriverManager.getConnection
    public static Service newInstance(){
        return newInstance(DEFAULT_PROVIDER_NAME);
    }
    
    public static Service newInstance(String name){
        Provider p = providers.get(name);
        if(p == null){
            throw new IllegalArgumentException("No provider registered with name: "+name);
        }
        return p.newService(); 
    }
}
```

**네 번째 장점은, 형인자 자료형(parameterized type)객체를 만들 때 편하다는 점이다.**<br>
```
Map<String, List<String>> m = new HashMap<String, List<String>>();
```
이처럼 자료형 명세를 중복하면, 형인자가 늘어남에 따라 길고 복잡한 코드가 만들어진다. 하지만 정적 팩토리 메서드를 사용하면 컴파일러가 형인자를 스스로 알아내도록 할 수 있다. 이런 기법을 자료형 유추라고 부른다.
```
public static <K, V> HashMap<K, v> newInstance(){
    return new HashMap<K,V>();
}

//이런 메서드가 있으면 좀 더 간결하게 작성할 수 있다. 
Map<String, List<String>> m = HashMap.newInstance(); 
```