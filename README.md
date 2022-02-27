> 왜 주제를 자바8로 정하였는가?

자바 역사를 통틀어 가장 큰 변화가 자바8에서 일어났다. 자바 9에서도 중요한 변화가 있었지만, 8만큼 획기적이거나 생산성이 변한 것은 아니다. 자바 10에서는 형 추론과 관련해 약간의 변화만 일어났다.

> 자바8의 획기적인 변화란?

자바 8은 프로그램을 더 효과적이고 간결하게 구현할 수 있는 새로운 개념과 기능을 제공한다.

- 자바 8의 기본적인 변화 관점 1. 간결한 코드 2. 멀티코어 프로세서 사용
- 멀티코어 CPU 대중화로, 기존 하나의 코어만 사용했던 JAVA에서 나머지 코어 활용을 위해 많은 방법이 나왔다.
1버전 쓰레드와 락, 5버전 쓰레드 풀, 병렬실행 컬렉션, 7버전 병렬에 도움이 되는 포크/조인 프레임워크
*8버전 스트림 API (병렬 실행을 위한 가장 쉽고 단순한 방식)

멀티코어 병렬 연산을 지원하는 스트림 api덕분에 코드를 전달하는 기법(메서드참조, 람다), 인터페이스의 디폴트 메소드가 존재할 수 있다.

ex ) 간결한 코드 변화 예시

```java
//Collections.sort를 할 때,
Collections.sort(inventory, new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
			return a1.getWeight().compareTo(a2.getWeight());
	}
});

방법 1.메서드참조
inventory.sort(comparing(Apple::getWeight));

방법2.람다식(익명메소드 전달)
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compreTo(a2.getWeight));
```

ex ) 스트림을 통해 병렬처리하는 예시

```java
List<Transaction> transactions = Arrays.asList(
        new Transaction(brian, 2011, 300),
        new Transaction(raoul, 2012, 1000),
        new Transaction(raoul, 2011, 400),
        new Transaction(mario, 2012, 710),
        new Transaction(mario, 2012, 700),
        new Transaction(alan, 2012, 950)
    );

		순차처리
    // 질의 1: 2011년부터 발생한 모든 거래를 찾아 값으로 정렬(작은 값에서 큰 값).
    List<Transaction> tr2011 = transactions.**stream()**
        .filter(transaction -> transaction.getYear() == 2011)
        .sorted(comparing(Transaction::getValue))
        .collect(toList());
    System.out.println(tr2011);

		병렬처리
		List<Transaction> tr2011 = transactions.**parallelStream()**
        .filter(transaction -> transaction.getYear() == 2011)
        .sorted(comparing(Transaction::getValue))
        .collect(toList());
	  System.out.println(tr2011);
	
```

> 자바 8 설계의 밑바탕을 이루는 세 가지 프로그래밍 개념

### 1. 스트림처리

**스트림**이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다. 이론적으로 프로그램은 입력 스트림에서 데이터를 한 개씩 읽어들이며 마찬가지로 출력 스트림으로 데이터를 한 개씩 기록한다. 즉, **어떤 프로그램의 출력 스트림은 다른 프로그램의 입력 스트림이 될 수 있다. (파이프라인)**

```bash
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
--파일의 단어를 소문자로 바꾼 다음에 사전순으로 단어를 정렬했을 때,
-- 가장 마지막에 위치한 세 단어를 출력하는 프로그램
```

유닉스에서는 여러 명령(cat, tr, sort, tail) 을 병렬로 실행한다. cat이나  tr이 완료되기 전에 sort가 행을 처리하기 시작할 수 있다.

- 기존 자바는 한번에 한 항목을 처리했지만, 이제 파이프라인을 구성했던 것 처럼 스트림API가 제공하는 많은 메서드로 병렬처리를 한다.

- 스트림 파이프라인을 이용하여 여러 CPU 코어에 쉽게 할당할 수 있따는 부가적인 이득도 있다.(스레드라는 복잡한 작업 사용x, 공자로 병렬성을 얻을 수 있다)

- 자바 8에서는 java.util.stream 패키지에 스트림 API가 추가되었다.  Stream<T>는 T 형식으로 구성된 일련의 항목을 의미한다.

### 2. 동작 파라미터화로 메서드에 코드 전달하기(메서드 참조, 람다)

동작파라미터화 = 

메서드가 다양한 동작을 받아서 내부적으로 다양한 동작을 수행할 수 있게 해주는 것.

- 기존 자바 8에서는 정해진 동작(method)에 인자(숫자, 문자 등) 만 전달할 수 있었다. (객체를 생성하여 전달할 수 있지만 너무 복잡하고 기존 동작을 단순하게 재활용)
- 자바 8에서는 메서드(우리 코드) 를 다른 메서드의 인수로 넘겨주는 기능을 제공한다.

왜 동작 파라미터화가 중요한가?

스트림 API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초하기 때문이다. (위의 Transaction 예시처럼)

```java
사과농장의 주인에게 사과 분류에 대한 여러 요청이 들어온 경우
**//기존의 방법**

**1.녹색 사과 필터링**
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (**"green".equals(apple.getColor())**) {
        result.add(apple);
      }
    }
    return result;
  }

단단히 응집되어 빨강을 비교하려면 같은 메서드를 하나 추가해야한다.

**2.색을 파라미터화**
public static List<Apple> filterGreenApples(List<Apple> inventory. **Color color**) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (**color**.**equals(apple.getColor())**) {
        result.add(apple);
      }
    }
    return result;
  }

**3.가능한 모든 속성으로 필터링**
public static List<Apple> filterApplesByWeight(List<Apple> inventory, **int weight**, 
																																			**Color color)** {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (**apple.getWeight() > weight  || 
					color**.**equals(apple.getColor())** {
        result.add(apple);
      }
    }
    return result;
  }

**//자바8에서의 방법**

**4.추상적 조건으로 필터링**
public static List<Apple> filterApplesByWeight(List<Apple> inventory, **ApplePredicate p**) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (**p.test**) {
        result.add(apple);
      }
    }
    return result;
  }

//동작 파라미터화시

interface ApplePredicate {

    boolean test(Apple a);

  }

  static class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }

  }

  static class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }

  }

//사용법
    List<Apple> greenApples = filter(inventory, new AppleColorPredicate());
  
    List<Apple> heavyApples = filter(inventory, new AppleWeightPredicate());

**//간소화 단계

5.익명클래스 사용**

List<Apple> redApples2 = filter(inventory, **new ApplePredicate() {
      @Override
      public boolean test(Apple a) {
        return a.getColor() == Color.RED;
      }
    });**
    System.out.println(redApples2);
  }

**6.람다 표현식 사용**

List<Apple> redApples2 = filter(inventory, **(Apple apple) -> RED.equals(apple.getColor()));**
  

**7.리스트 형식으로 추상화**

//자바에서 제공
public interface Predicate<T> {
	boolean test(T t);
	}

//형식 파라미터 T
public static <T> List<T> filter(List<T> list, Predicate<T> p){
	List<T> result = new ArrayList<>();
	for(T e : list){
		if(p.test(e)){
			result.add(e);
	}
}
return result;
}

//이제 오렌지, 바나나, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다.

List<Apple> redApples = 
									filter(inventory, (Apple apple) -> RED.equals(apple.getColor());

List<Integer> evenNumbers = 
									filter(numbers, (Integer i) -> i % 2 == 0);

한결 간결하고 유연한 코드가 되었다.

동작 파라미터화 패턴은 동작을 한 조각의 코드로 캡슐화한 다음에 메서드로 전달해서
메서드의 동작을 파라미터화한다. (ex 사과의 다양한 프레디케이트).

** 자바API의 많은 메서드를 다양한 동작으로 파라미터화할 수 있다. 그리고 많은 익명 클래스와 
자주사용하곤 한다. (ex Runnable, GUI 이벤트 처리하기)

ex) GUI 이벤트 처리하기
Button button = new Button("Send");
button.setOnAction(new EventHandler<ActionEvent>() {
	public void handle(ActionEvent event){
		label.setText("Sent!!");
	}
});

즉, EventHandler는 setOnAction 메서드의 동작을 파라미터화한다. 람다식으로 표현하면

button.setOnAction((ActionEvent event) -> label.setTExt("Sent!!")); 이런식
```

### 3.병렬성과 공유 가변 데이터

 **자바 8**은 **'병렬성을 공짜로 얻을 수 있다'**라는 말에서 시작된다. 병렬성을 얻은 대신, 스트림 메서드로 전달하는 코드의 동작 방식을 조금 바꿔야 한다.  스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 **안전하게 실행**될 수 있어야 하기 때문이다.

 **안전하게 실행**할 수 있는 코드는 **'공유된 가변 데이터'**에 접근하지 않아야 한다. 이를 위해 기존 자바는 synchronized를 이용해 사용했지만, 높은 난이도, 시스템 성능에 악영향 등의 단점이 있었다. 스트림 API는 수학적인 함수처럼 함수가 정해진 기능만 수행하여 다른 부작용은 일으키지 않는다.

 공유되지 않는 가변데이터, 메서드와 함수 코드를 다른 메서드로 전달하는 두 가지 기능은 **함수형 프로그래밍**으로의 변화에서 핵심적인 사항이다. ****

함수

프로그래밍 언어에서 함수(function)은 보통 메서드와 같은 의미로 사용된다.

자바에서는 이에 더해 수학적인 함수처럼 사용되며 부작용을 일으키지 않는 함수를 의미한다.

자바 8에서는 함수를 새로운 값의 형식으로 추가했다. 멀티코어 병렬 프로그래밍을 활용할 수 있는 Stream과 연계될 수 있도록 만들었다. 

### 함수를 값처럼 사용하다

 자바에서 조작할 수 있는 값은 첫 번째로 42(int), 3.14(dobule) 같은 기본값이 있다.  두 번째로 객체 클래스의 인스턴스(엄밀히 말하자면 객체의 참조 또는 라이브러리 함수를 이용해서 객체의 값을 얻는 방식) ex) "abc"(String 형식), new HashMap<Integer, String>이 있다.

 프로그래밍의 핵심은 값을 바꾸는 것이다. 값이 변하는 것들은 **'일급값'** 이라고 한다.

변하지 않는 구조체 (메서드, 클래스) 등은 '**이급 시민'** 이라고 부른다. 이런 이급 시민인 메서드를 런타임시에 전달할 수 있다면 프로그래밍에 유용하게 사용할 수 있다.

	
	
	
	###############################
	
	
	
	
	
	
	
	
	
	
> 람다

## 람다란?

메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

- 익명

    보통 메서드와 달리 이름이 없어 **익명**

- 함수

    람다는 메서드처럼 특정 '클래스'에 종속되지 않아 함수라고 칭한다. (but, 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함)

- 전달

    람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.

- 간결성

    익명 클래스처럼 자질구레한 코드를 구현할 필요가 없다.

람다를 사용하여 코드를 전달하는 과정에서 판에 박힌 코드를 구현할 필요가 없게 만들어준다. (동작 파라미터 형식의 코드를 더 쉽게 구현할 수 있다) 결과적으로 코드를 더 간결하고 유연하게 만들어준다.

```java
ex)
//기존 
Comparator<Apple> byWeight = new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}

//Comparetor 객체를 byWeight란 이름으로 만들고 Comparator가 필요한 위치에 삽입하는 방식

//람다

Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeigth().compareTo(a2.getWeight());
```

```java
기본적인 람다 표현식

(Apple a1 . Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

(람다 파라미터) -> 람다 바디;
```

- 람다 파라미터 리스트

    Comparator의  compare 메서드 파라미터 (사과 두 개)

- 화살표

    화살표 (→) 는 람다 파라미터 리스트와 바디를 구분한다.

- 람다 바디

    두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

    > 그래서 정확히 람다가 어디서 사용되는가?

    ## 함수형 인터페이스

     함수형 인터페이스란 정확히 **하나의 추상 메서드**를 지정하는 인터페이스이다.

    추상 메서드란 
    선언만 하고 내용은 선언하지 않은 메서드이다.

    ```java
    public interface Predicate<T>{
    	boolean test (T t);
    }

    ```

    인터페이스는 dafault 메서드를 포함할 수 있다.  default 메서드는 구현이 이미 되어있는 메서드이다. 이미 구현을 했으니 해당 인터페이스를 구현하는 클래스는 추가된 메서드의 구현을 할 필요가 없다. 

    자바 8에서는 대표적으로 List 인터페이스의 sort, Collection 메서드의 stream 메서드가 있다.

    ```java
    @FunctionalInterface
    public interface Predicate<T>{
    	boolean test (T t);
    	
    	default void sort(Comparator<? super E> c){
    	Collections.sort(this,c);
    	}	
    }

    이런 식으로 굳이 단 하나의 추상 메서드만 존재하지만 이미 구현된 default 메서드는
    인터페이스에 마음껏 존재하여도 된다.
    ```

    - Quiz

        다음 인터페이스중 함수형 인터페이스는?

        ```java
        //1

        public interface Adder {
        	int add(int a, int b);
        }

        public interface SmartAdder extends Adder {
        	int add(double a, dobule b);
        }

        public interface Nothing {
        }
        ```

        ```java
        //1

        public interface Adder {
        	int add(int a, int b);
        }

        public interface SmartAdder extends Adder {
        	int add(double a, dobule b);
        }

        public interface Nothing {
        }
        ```

    - 정답

        Adder만 함수형 인터페이스이다.

        SmartAdder는 두 추상 add 메서드(하나는 상속받음)를 포함하므로 아니다.

        Nothing은 없으므로 아니다.

    함수형 인터페이스로 뭘 할 수 있을까?

    람다 표현식으로 함수형 인터페이스의 추상메서드 구현을 직접 전달할 수 있으므로, 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

    ```java
    1. 람다 사용
    Runnable r1 = () -> System.out.println("Hello World 1");

    2. 익명 클래스 사용
    Runnable r2 = new Runnable() {
    	public void run() {
    		System.out.println("Hello World 2");
    	}
    }

    3.람다 직접 전달

    public static void process(Runnable r){
    	r.run();
    }

    process(r1);
    process(r2);
    **process(() -> System.out.println("Hello World 3");**
    ```

    > 함수 디스크립터

    함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.

    람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터**라고 부른다.

    즉, 위의 Runnable 인터페이스의 추상메서드 run은 인자가 없고, return값도 없다는 것이 함수 디스크립터이다. **() → void** 

     

    왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?

    언어 설계자들이 자바에 함수형식을 추가하는 방법도 대안으로 고려해봤는데, 언어를 더 복잡하게 만들지 않는 지금 방법을 선택했다. 기존의 자바 프로그래머가 하나의 추상 메서드를 가지는 인터페이스에 이미 익숙하다는 점을 고려했다.

    > 실용적 사용 정리

    ### 실행 어라운드 패턴

    자원 처리(예를 들면 데이터베이스의 파일 처리) 에 사용하는 순환 패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다.

    즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태이다.

    이를 동작 파라미터전달로(람다) 간소화할 수 있다.

    ```jsx
    // 1. 동작 파라미터화를 기억하라. 
    //자바 7에 추가된 try-with-resources라 자원을 닫아줄 필요가 없다.
    public static String processFile() throws IOException { 
    	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){ 
    			return br.readLine(); 
    	} 
    }

    // 2. 함수형 인터페이스를 이용해서 동작 전달 
    public interface BufferedReaderProcessor { 
    	String process(BufferedReader b) throws IOException; 
    }

    public static String processFile(BufferedReaderProcessor p) throws IOException { 
    ... 
    }

    // 3. 동작 실행 
    public static String processFile(BufferedReaderProcessor p) throws IOException {
     try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
    	 return p.process(br); 
    }
    ```

    - Quiz BufferedReaderProcessor 의 함수 디스크립션을 말해보시오

        BufferedReaderProcessor의 추상메서드는 process이며

        (BufferedReader) → String으로 표현할 수 있다.

    - Quiz 라인을 출력하려면 어떻게 해야 할까?

        ```jsx
        // 4. 람다 전달. 
        String oneLine = processFile((BufferedReader br) -> br.readLine()); 
        String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
        ```

   ![image](https://user-images.githubusercontent.com/37995817/125884759-0bbc7e6c-bda7-4712-815a-6eebfc6d99d8.png)

    ### 함수형 인터페이스 사용

    함수형 인터페이스는 오직 하나의 **'추상 메서드'**를 가진다고 했다. 함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다.  함수형 인터페이스의 추상 메서드 시그니처를 **'함수 디스크립터'** 라고 한다. 자바 API는 이미 제작된 다양한 함수형 인터페이스를 포함하고 있다.

![image](https://user-images.githubusercontent.com/37995817/125884733-26626954-1ea9-4440-9904-34652f43a22c.png)
    > 람다 동작 원리

    ### 형식 검사

    ```jsx
    List<Apple> HeavierThan150g = 
    		filter(inventory, (Apple apple) -> apple.getWeight() > 150);
    ```

1. filter 메서드의 선언을 확인한다.

    ```jsx
    filter(List<Apple>inventory, Predicate<Apple> p)
    ```

2. filter 메서드는 두 번째 파라미터로 Predicate<Apple>을 기대한다. T → boolean
3. Predicate<Apple>인터페이스의 추상 메서드는 

```jsx
boolean test(Apple apple)
```

4.Apple을 인수로 받아 boolean을 반환하는 test 메서드임을 확인한다.

5.함수 디스크립터가 Apple → boolean이므로 람다의 시그니처와 일치한다. 람다도 Apple을 인수로 받아 boolean을 반환하므로 코드 형식 검사가 성공적으로 완료된다.

### 형식 추론

자바 컴파일러가 람다 표현식이 사용된 콘텍스트(대상 형식)을 이용하여 람다 표현식과 관련된 함수형 인터페이스를 추론한다. 즉, **대상형식을 이용하여** 함수 디스크립터를 알 수 있으므로 람다 시그니처도 추론할 수 있다. 그래서 람다 문법에서 이를 생략할 수 있다.

```jsx
Comparator<Apple> c = 
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
Comparator<Apple> c = 
	(a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

> **메서드참조**

메서드 정의를 재활용하여 람다처럼 전달할 수 있다. 특정 메서드만을 호출하는 람다의 축약형이다. (즉, 람다표현식에서 하는 일이라고는 이미 존재하는 메서드를 호출하는게 저눕인 경우, 사용할 수 있다.)

명시적으로 메서드명을 참조함으로써 가독성을 높힐 수 있다.

![image](https://user-images.githubusercontent.com/37995817/125884704-f8ffac00-139d-4989-ace7-6ec8fc11d079.png)
1.정적 메소드 참조 예

Integer :: parseInt

2.다양한 형식의 인스턴스 메서드 예

![image](https://user-images.githubusercontent.com/37995817/125884686-c56721b2-362c-4a8d-8813-1faaa1a1e350.png)
3.기존 객체의 인스턴스 메서드 참조 예

![image](https://user-images.githubusercontent.com/37995817/125884673-c353ca6f-8bc3-4bfb-8149-ae3a3ca18c46.png)
- 1 번과 2번의 차이는 이런 느낌이라고 생각한다.

![image](https://user-images.githubusercontent.com/37995817/125884667-d0c7bfd8-bd3f-4005-806c-5054ad0120d4.png)
4. 생성자 참조

```java
List<Integer> weights = Arrays.asList(7, 3, 4, 10);
List<Apple> apples = map(weights, Apple::new);
List<Apple> apples = map(weights, ()->new Apple());

public List<Apple> map(List<Integer> list,
																Function<Integer, Apple> f){
	List<Apple> result = new ArrayList<>();
	for(Integer i : list){
		result.add(f.apply(i));
	}
	return result;
}

//<T,R>->R 디스크립션인 Function <Integer,Apple> -> Apple객체 생성
```

> 응용, 정리

람다를 1회성 행위를 위해서 사용한다면 훌륭하지만, 여러 곳에서 사용된다면 어떨까?

```java
class Person {
  public String firstName;
  public String lastName;
  public int age;
};
```

Person 객체를 SortedSet에 저정하거나 어떤 형태로든 리스트 내에서 정렬될 필요가 있을 때, Person 인스턴스가 정렬되기 위한 다른 여러가지 방식을 필요로 합니다. 예를 들어, 이름으로 정렬하고 싶을 때도 있고 성으로 정렬하고 싶을 때도 있습니다. 이 목적으로 Comparator<T>를 사용하는데 Comparator<T> 인스턴스를 전달함으로써 정렬 방식을 정의할 수 있게 됩니다.

```java
public interface Comparator<T>{
	int compare(T o1, T o2);
}//함수형 인터페이스 Comparator
```

람다로 정의하면 간단하게 코드를 작성할 수 있습니다.

```java
public static void main(String... args) {
  Person[] people = new Person[] {
    new Person("Ted", "Neward", 41),
    new Person("Charlotte", "Neward", 41),
    new Person("Michael", "Neward", 19),
    new Person("Matthew", "Neward", 13)
  };
  // Sort by first name
  Arrays.sort(people, (lhs, rhs) -> lhs.firstName.compareTo(rhs.firstName));
  for (Person p : people)
    System.out.println(p);
}
```

그러나 만약 Person 인스턴스를 이름에 의해 정렬하는 것을 여러번 수행한다면 저 람다 코드가 반복이 될 것이다. 그래서 Comparator 객체를 생성해준다.

```java
//Comparator를 Person 자체의 멤버변수로 선언해준다. 변하면 안되기에 final

class Person {
  public String firstName;
  public String lastName;
  public int age;

  public final static Comparator<Person> compareFirstName =
    (lhs, rhs) -> lhs.firstName.compareTo(rhs.firstName);

  public final static Comparator<Person> compareLastName =
    (lhs, rhs) -> lhs.lastName.compareTo(rhs.lastName);

  public Person(String f, String l, int a) {
    firstName = f; lastName = l; age = a;
  }

  public String toString() {
    return "[Person: firstName:" + firstName + " " +
      "lastName:" + lastName + " " + "age:" + age + "]";
  }
}
```

그 뒤에 다른 static 필드처럼 참조당할 수 있다.

```java
public static void main(String... args) {
  Person[] people = . . .;

  // Sort by first name
  Arrays.sort(people, Person.compareFirstName);
  for (Person p : people)
    System.out.println(p);
}
```

compareFirstName을 바로 메소드 참조를 이용하여 사용하는 방법이다.

```java

class Person {
  public String firstName;
  public String lastName;
  public int age;

  public static int compareFirstNames(Person lhs, Person rhs) {
    return lhs.firstName.compareTo(rhs.firstName);
  }

  // ...
}

public static void main(String... args) {
  Person[] people = . . .;
  // Sort by first name
  Arrays.sort(people, **Person::compareFirstNames**);
  for (Person p : people)
    System.out.println(p);
}

혹은 Comparator 객체를 생성하여 넣어주기

public static void main(String... args) {
  Person[] people = . . .;
	Comparator cf = Person::compareFirstNames;

  // Sort by first name
  Arrays.sort(people, cf);
  for (Person p : people)
    System.out.println(p);
}
```

Collections의 sort 메서드의 모습.

![image](https://user-images.githubusercontent.com/37995817/125884642-370fb88a-6a29-455d-8ca4-7e8fbccef07f.png)

가장 강력한 방법으로는 새로운 라이브러리를 사용하는 것이다. 

```java

Arrays.sort(people, comparing(Person::getFirstName));

```

![image](https://user-images.githubusercontent.com/37995817/125884856-2a00ba2e-f5a4-4f48-af11-9e6710a462b3.png)

comparing은 Function함수 인터페이스를 받아서 Comparator 함수 인터페이스를 return해주는 자바8의 추가된 java.util.Comparator 라이브러리이다.

![image](https://user-images.githubusercontent.com/37995817/125884832-8bba126b-eccb-4b8f-918f-edc7022ab7ac.png)
> 스트림

### Collection이란?

![image](https://user-images.githubusercontent.com/37995817/125884543-27c5b1a7-aea8-4808-b9ea-476853ce3181.png)

자바에서는 이 Collection으로 데이터를 그룹화하고 처리한다. 모든 요리의 칼로리 합이라던지, 칼로리가 50 이하인 음식을 고른다던지 하는 식이다.

데이터베이스에서는 선언형으로 SELECT name FROM dishes WHERE calorie 이런 식으로 구현한다.

위에서 봤듯, SQL 질의는 요리의 속성을 이용하여 어떻게 필터링할 것인지 구현할 필요가 없다. (자바처럼 반복자, 누적자 등을 이용하는 것)

자바 8에서는 이런 식으로 Collection을 처리하고 싶었다.  성능 좋게 멀티코어 아키텍처를 활용해서 병렬적으로. 그렇기에 스트림은 탄생했다.

> 스트림이란?

스트림을 이용하면 선언형 (즉, 데이터를 처리하는 임시 구현 코드 대신 질의로 표현)으로 Collection 데이터를 처리할 수 있다.  그리고 스트림을 사용하면 멀티스레드 구현을 하지 않더라도 데이터를 투명하게 병렬로 처리할 수 있다.

정의 : 스트림이란 '데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소'

### 구성 요소

- 데이터 처리 연산 : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원. EX) filter, map, reduce, find, match, sort 등으로 데이터 조작
- 소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. (정렬된 컬렉션으로 스트림을 생성하면 정렬이 유지된다.)
- 연속된 요소 : 컬렉션은 자료구조이므로 컬렉션에서는 ArrayList를 쓸지, LinkedList를 쓸지에 대한 고민과 같은 시간,공간복잡성 등 데이터 성질 주체이다. 스트림은 연속된 값 집합의 인터페이스를 제공하여 계산을 주체로 사용된다. *( 여기서 연속된이란 의미는 순차적으로 값에 접근하는 것이다)

### 두가지 특징

- 파이프라이닝 : 대부분의 스트림 연산은 스트림 연산끼리 연결해 스트림 자신을 반환한다.
- 내부 반복 : 반복자를 명시하여 반복하는 컬렉션과 다르게 내부 반복을 지원한다.
- 게으른 생성 : 필요할 때만 값을 계산한다.

예시 자바7

```java
public static List<String> getLowCaloricDishesNamesInJava7(List<Dish> dishes) {
    List<Dish> lowCaloricDishes = new ArrayList<>();
    for (Dish d : dishes) {
      if (d.getCalories() < 400) {
        lowCaloricDishes.add(d);
      }
    }
    List<String> lowCaloricDishesName = new ArrayList<>();
    Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
      @Override
      public int compare(Dish d1, Dish d2) {
        return Integer.compare(d1.getCalories(), d2.getCalories());
      }
    });
    for (Dish d : lowCaloricDishes) {
      lowCaloricDishesName.add(d.getName());
    }
    return lowCaloricDishesName;
  }
```

예시 자바8

```java
public static List<String> getLowCaloricDishesNamesInJava8(List<Dish> dishes) {
    return dishes.stream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
  }
```

작동 순서
dishes 리스트에서 stream 메서드를 호출하여 스트림을 얻었다.
데이터 소스 : dishes (연속된 요소) 를 스트림에 제공한다.
filter, sort, map, collect 같은 데이터 처리 연산을 적용한다. 
*collect를 제외한 모든 연산은 서로 파이프라인으로 연결되도록 스트림을 반환한다.
마지막으로 collect로 스트림을 List로 변환하여 배출한다. 

**마지막 collect 호출 전까지는 무엇도 선택되지 않았고 출력 결과도 없다. (메서드 호출이 collect 이전까지 저장됨)

filter : 람다를 인수로 받아 스트림에서 특정 요소 제외

map : 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출

limit : 정해진 개수 이상으로 요소가 스트림에 저장되지 못하게 스트림 크기 축소

collect : 스트림을 다른 형식으로 변환한다.

![image](https://user-images.githubusercontent.com/37995817/125884525-c508efdd-a4c1-4ce3-b736-a78451b42a4c.png)
# 컬렉션 vs 스트림

컬렉션은 DVD이고, 스트림은 인터넷 스트리밍이다.

(DVD는 영화 전체가 로딩되고 나서야 재생 가능하고, 스트리밍은 들어오는 순서부터 바로 보여주기 시작한다.)

컬렉션은 팔기도 전에 창고를 가득 채우고 스트림은 데이터를 요청할 때만 값을 계산한다.

수학적으로 이는 굉장히 큰 값어치를 가진다. 

 예를 들어 무제한의 소수를 포함하는 스트림과 컬렉션이 있을 때, 컬렉션은 모든 소수를 다 구하고 값을 사용자에게 반환하려 할 것이다 그렇다면 무한 루프를 돌고 사용자는 평생 결과값을 받을 수 없을 것이다. 

 반면에 스트림은 소수가 하나하나 요청이 들어올 때 마다 결과를 반환하기 때문에 무제한의 수에도 연산이 가능하다. 

이를 DVD는 '적극적 생성' 이라 하고 스트림은 '게으른 생성' 이라고 칭한다.

- 스트림은 딱 한번만 탐색할 수 있다. (흩어짐)
- 컬렉션은 외부반복, 스트림은 내부반복이다.

```java
//기존 컬렉션 처리 방법
List<String> names = new ArrayList<>();
for(Dish dish:menu){
	names.add(dish.getName()); //<- 메뉴리스트를 명시적으로 순차 반복한다.
}

//스트림
List<String> name = menu.stream
										.map(Dish::getName)
										.collect(toList()); //파이프라인을 실행한다. 반복자는 필요 없다.
```

아이에게 장난감을 치우라고 지시할 때, 장난감이 있냐고 물어본 뒤, 그 장난감을 치우고 다음 장난감은 무엇이 있지? 그리고 치워를 반복하는 것과 바닥에 있는 모든 장난감을 치우렴의 차이이다.

- Quiz

    ```java
    다음 Collection코드를 Stream으로 바꿔보자.

    List<String> highCaloricDishes = new ArrayList<>();
    Iterator<String> iterator = menu.iterator();
    while(iterator.hasNext(){
    	Dish dish = iterator.next();
    	if(dish.getCalories() > 300) {
    			highCaloricDishes.add(d.getName());
    	}
    }
    ```

    - Ans

### 중간연산, 최종연산

스트림의 연산 중, 스트림을 반환하는 filter,map,limit 등은 중간연산이다.

collect같이 결과를 반환하는 것은 최종연산이다.

스트림의 게으른 특성 덕분에 중간 연산을 합친 다음 합쳐진 중간 연산을 최종 연산으로 한 번에 처리한다.

최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 List, Integer,void 등 스트림 이외의 결과

![image](https://user-images.githubusercontent.com/37995817/125884491-c03d057f-c027-4ed5-8ef0-dba6b1554f5d.png)
> 요약

스트림 이용 과정은 다음 세 가지로 요약 가능하다.

- 질의를 수행할 (컬렉션 같은) 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산
