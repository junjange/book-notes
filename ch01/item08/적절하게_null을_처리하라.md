## 💡 아이템 8: 적절하게 null을 처리하라

null은 '값이 부족하다(lack of value)'는 것을 나타낸다.
프로퍼티가 null이 라는 것은 값이 설정되지 않았거나, 제거되었다는 것을 나타낸다.
함수가 null을 리턴하는 것은 함수에 따라서 여러 의미를 가질 수 있다.

- String.toIntOrNull()은 String을 Int로 적절하게 변환할 수 없을 경우 null을 리턴한다.
- Iterable<T>.firstOrNull(() → Boolean)은 주어진 조건에 맞는 요소가 없을 경우 null을 리턴한다.

이처럼 null은 최대한 명확한 의미를 갖는 것이 좋다.
이는 nullable 값을 처리해야 하기 때문인데, 이를 처리하는 사람은 API 사용자(API 요소를 사용하는 개발자)이다.

기본적으로 nullable 타입은 세 가지 방법으로 처리한다.

- ?. 스마트캐스팅, Elvis 연산자 등을 활용해서 안전하게 처리한다.
- 오류를 throw한다.
- 함수 혹은 프로퍼티를 리팩터링 해서 nullable 타입이 나오지 않게 바꾼다.

### 📌 null을 안전하게 처리하기

null을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 안전 호출(safe call)과 스마트 캐스팅(smart casting)이 있다.

 ```kotlin
printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅
```

두 가지 모두 printer가 null이 아닐 때 print 함수를 호출한다.
애플리케이션 사용자의 관점에서 가장 안전한 방법이다.
사실 개발자에게도 정말 편리하여, nullable 값을 처리할 대 이 방법을 가장 많이 활용한다.

코틀린은 nullable 변수와 관련된 처리를 굉장히 광범위하게 지원한다.
대표적으로 인기 있는 다른 방법은 Elvis 연산자를 사용하는 것이다.
Elvis 연산자는 오른쪽에 return 또는 throw을 포함한 모든 표현식이 허용된다.

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

많은 객체가 nullable과 관련된 처리를 지원한다.
예를 들어 컬렉션 처리를 할 때 무언가 없다는 것을 나타낼 때는 null이 아닌 빈 컬렉션을 사용하는 것이 일반적이다.
따라서 Collection<T>?.orEmpty() 확장 함수를 사용하면 nullable이 아닌 List<T>를 리턴 받는다.

스마트 캐스팅은 코틀린의 규약 기능(contracts feature)을 지원한다.
이 기능을 사용하면 다음 코드처럼 스마트 캐스팅할 수 있다.

```kotlin
println("What is your name?")
val name = Leadline()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()}")
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    bews.forEach { notifyUser(it) }
}

```

지금까지 살펴묜 null을 적절하게 처리하기 위한 방법들은 반드시 알아 두어야 한다.

방어적 프로그래밍(defensive programming): 코드가 프로덕션 환경으로 들어갔을 때 발생할 수 있는 수많은 것들로부터 프로그램을 방어해서 안정성을 높이는 방법을 나타내는 굉장히 포괄적인 용어
공경적 프로그래밍(offensive programming): 예상하지 못한 상화이 발생했을 때, 이러한 문제를 개발자에게 알려서 수정하게 만드는 방법

### 📌 오류 throw 하기

이전에 살펴보았던 코드에서는 printer가 null일 때, 이를 개발자에게 알리지 않고 코드가 그대로 진행된다.
하지만 printer가 null이 되리라 예상하지 못했다면, print 메서드가 호출되지 않아서 이상할 것이다.
이는 개발자가 오류를 찾기 어렵게 만든다.
따라서 다른 개발자가 어떤 코드를 보고 선입견처럼 '당연히 그럴 것이다.'라고 생각하게 되는 부분이 있고, 그 부분에서 문제가 발생할 경우에는 개발자가 오류를 강제로 발생시켜주는 것이 좋다.
오류를 강제로 발생시킬 때는 throw, !!, requireNotNull, checkNotNull 등을 활용한다.

### 📌 not-null assertion (!!) 과 관련된 문제

nullabl을 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 것이다.
그런데 !!를 사용하면 자바에서 nullable을 처리할 때 발생할 수 있는 문제가 똑같이 발생한다.
어떤 대상이 null이 아니라고 생각하고 다루면, NPE 예외가 발생한다.
!!은 사용하기 쉽지만, 좋은 해결 방법은 아니다.

일반적으로 !! 연산자 사용을 피해야한다.
이러한 제안은 코틀린 커뮤니티 전체에서 널리 승인되고 있는 제안이다.
대부분의 팁이 !! 연산자를 아예 사용하지 못하게 하는 정책을 갖고 있다.
Detekt와 같은 정적 분석 도구는 !! 연산자를 사용하면, 아예 오류를 발생하도록 설정되어 있다.
필자의 경우, 오류까지 발생시키는 것은 조금 극단적이라고 생각하기는 하지만, 문제가 발생할 수 있는 코드를 짚어 줄 수 있다는 것에 동의한다.
!! 연산자를 보면 반드시 조심하고, 무언가가 잘못되어 있을 가능성을 생각하자.

### 📌 의미없는 nullability 피하기

nullability는 어떻게든 적절하게 처리해야 하므로, 추가 비용이 발생한다.
따라서 필요한 경우가 아니라면, nullability 자체를 피하는 것이 좋다.
null은 중요한 메시지를 전달하는 데 사용될 수 있다.
따라서 다른 개발자가 보기에 의미가 없을 때는 null을 사용하지 않는 것이 좋다.
만약 이유 없이 null을 사용했다면, 다른 개발자들이 코드를 작성할 때, 위험한 !! 연산자를 사용하게 되고, 의미 없이 코드를 더럽히는 예외 처리를 해야할 것이다.
nullalbility를 피할 때 사용할 수 있는 몇 가지 방법을 소개하겠다.

- 클래스에서 nullability에 따라 여러 함수를 만들어서 제공할 수도 있다.
- 어떤 값이 클래스 생성 이후에 확실하게 설정된다는 보장이 있다면, lateinit이나 notNull 델리게이트 사용하자.
- 빈 컬렉션 대신 null을 리턴하지 말라
- nullable enum 대신에 None enum 으로 처리하라

### 📌 lateinit 프로퍼티와 notNull 델리게이트

클래스가 클래스 생성 중에 초기화할 수 없는 프로퍼티를 가지는 것은 드문 일은 아니지만 분명 존재하는 일이다.
이러한 프로퍼티는 사용 전에 반드시 초기화해서 사용해야 한다.

프로퍼티를 사용할 때마다 nullalbe에서 null이 아닌 것으로 타입 변환하는 것은 바람직하지 않다.
이러한 값은 테스트 전에 설정될 거라는 것이 명확하므로, 의미 없는 코드가 사용된다고 할 수 있다.
이러한 코드에 대한 바람직한 해결책은 나중에 속성을 초기화할 수 있는, lateinit 한정자를 사용하는 것이다.
lateinit 한정자는 프로퍼티가 이후에 설정될 것임을 명시하는 한정자이다.

물론 lateinit을 사용할 경우에도 비용이 발생한다.
만약 초기화 전에 값을 사용하려고 하면 예외가 발생한다.
무섭게 들릴 수도 있겠지만, 무서워 할 필요가 없다.
처음 사용하기 전에 반드시 초기화가 되어 있을 경우에만 lateinit을 붙이는 것이다.
만약 그런 값이 사용되어 예외가 발생한다면, 그 사실을 알아야 하므로 예외기 발생하는 것은 오히려 좋은 일이다.
lateinit는 nullable과 비교하면 다음과 같은 차이가 있따.

- lateinit을 쓰는 경우에 !! 연산자로 언팩하지 않아도 된다.
- 이후에 어떤 의미를 나타내기 위해서 null을 사용하고 싶을 때, nullable로 만들 수도 있따.
- 프로퍼티가 초기하된 이후에는 초기화되지 않은 상태로 돌아갈 수 없다.

반대로 lateinit을 사용할 수 없는 경우도 있다.
JVM에서 Int, Long, Double, Boolean과 같은 기본 타입과 연결된 타입으로 프로퍼티를 초기화해야하는 경우이다.
이런 경우에는 lateinit보다는 약간 느리지만, Delegetes.notNull을 사용한다.

```kotlin
class DoctorActivity : Activty() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean by Delegates.notNull()

    override onCreate(savedInstanceState: Bundle?)
    {
        super.onCreate(savedInstanceState)
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
    }
}

```

위의 코드처럼 onCreate 때 초기화하는 프로퍼티는 지연 초기화하는 형태로 다음과 같이 프로퍼티 위임(property delegation)을 사용할 수도 있따.

```kotlin
class DoctorActivity : Activty() {
    private var doctorId: Int by arg(DOCTOR_ID_ARG)
    private var fromNotification: Boolean by arg(FROM_NOTIFICATION_ARG)
}

```

프로퍼티 위임을 사용하는 패턴은 '아이템 21: 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라'에서 자세하게 다룰 예정이다.
프로퍼티 위임을 사용하면, nullability로 발생하는 여러 가지 문제를 안전하게 처리할 수 있다.