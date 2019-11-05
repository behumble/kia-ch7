---
marp: true
title: Operator overloading and other conventions
description: Chapter 7 of the book 'Kotlin in Action'
theme: uncover
backgroundColor: lightgreen
paginate: true
---
# Operator overloading
and other conventions

---
# Java - Interface specific language features

---
# `Iterable`
#### [foreach](https://www.baeldung.com/java-loops#foreach) in Java 5+
```java
// language support for Iterable
List<String> stringList = Arrays.asList("One", "Two", "Three", "Drink");
assert stringList instanceof Iterable;
for (String s : stringList) {
    System.out.println(s);
}
```
---
# `AutoCloseable`
#### [try-with-resources](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) in Java 7+
```java
// java 7 syntax for AutoClosable (try-with-resources)
try (FileInputStream fis = new FileInputStream(".")) {
    assert fis instanceof AutoCloseable;
} catch (IOException ioe) {
    ioe.printStackTrace();
}
```

---
# Kotlin - Conventions
extension by set of predefined method **name**s

---
![height:600px](./assets/img/binaryArithmetic.png)

---
# `+` (infix) operator
```kotlin
class Point(val x:Int, val y:Int) {
    operator fun plus(other: Point) = Point(x+other.x, y+other.y)
}

val p1 = Point(1,1)
val p2 = Point(2,3)

// >>> (p1+p2).x
// kotlin.Int = 3
```
`operator` modifier를 빼면?

---
```
>>> p1+p2
error: 'operator' modifier is required on 'plus' in 'Line_2.Point'
p1+p2
  ^
```
Extension function + data class 조합으로
```kotlin
data class Point(val x:Int, val y:Int)
operator fun Point.plus(val o:Point) = Point(this.x+o.x, this.y+o.y)

>>> val p1 = Point(1,1)
>>> val p2 = Point(2,3)
>>> p1+p2
res6: Line_0.Point = Point(x=3, y=4)
```

---
# Double.plus() 동작 변경하기
```kotlin
operator fun Double.plus(other:Double):Double {
    println("Double.plus"+other)
    return this + other
}

>>> 1.0+2.0
res19: kotlin.Double = 3.0
```
오류도 없이 안됨 - [이유](https://kotlinlang.org/docs/reference/extensions.html#extensions-are-resolved-statically)
**member always wins**

---
# 함수 이름 기반 convention
- operand 타입이 this와 달라도 됨
```kotlin
operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}
>>> val p = Point(10, 20)
>>> println(p * 1.5) Point(x=15, y=30)
```

---
# commutativity는 셀프

`1.5 * p`와 `p * 1.5`는 다른 이야기다. 같아야 한다면 같게 만들어야 함
```kotlin
operator fun Double.times(p: Point):Point {
    return p * this
}

```

---
# 다른 type을 return 해도 된다

당연한 이야기니 속지말자

(`Char`에서 `String`을 return 하는 extension function 예제)
```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}
>>> println('a' * 3)
aaa
```
---
# bitwise operator가 없다

![height:480px](./assets/img/bitwise.png)

---
## Compound assignment operators

- `plus()`만 정의하여도 `+=` 은 자동으로 동작한다
```kotlin
data class Point(val x:Int, val y:Int)

operator fun Point.plus(v2: Point):Point {
    return Point(x+v2.x, y+v2.y)
}

val p1 = Point(1,1)
p1 += Point(2,3) // ??
```

---
# `plusAssign`
```kotlin
val l = ArrayList<Int>()
l += 3   // what do we expect? 
```

l.append(3) 처럼 동작하길 바란다
이럴 경우 `plus()` 대신 `plusAssign()`을 정의하면 된다.
```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    this.add(element)
}
```

---
# 어려운 문제

![height:200px](./assets/img/ambiguity.png)

```kotlin
var list = arrayListOf(1,2,3)
list += 4 // ?? 해보자
```

끝이 아니다
```kotlin
val vList = list
vList += 4 // ??
```
---
# Unary operators
특별한 거 없다. 인자만 없다.
```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

>>> val p = Point(10, 20)
>>> println(-p)
Point(x=-10, y=-20)
```
- `unaryPlus()`, `unaryMinus()`, `not()`
- `inc()`, `dec()`

---
# inc() & dec()
```kotlin
operator fun Point.inc() = Point(x + 1, y + 1)

>> var p = Point(4, 2)
>> println(p++)  // postfix
>> println(p)
Point(x=4, y=2)
Point(x=5, y=3)
>> println(++p)  // prefix
Point(x=6, y=4)
```

operator function내부에서 prefix인지 postfix인지 구분은 안됨

---
# Comparison operators
![height:120px](./assets/img/equals.png)
- a가 null이어도 된다
    - [Intrinsics.areEquals()](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/jvm/runtime/kotlin/jvm/internal/Intrinsics.java#L161)를 통해 호출된다
- 특이하게도 `override`가 붙는다
    - `Any`에 정의되어있기 때문 (`operator`는 붙이지 않는다)
- extension function으로 정의할 수 없다
- `!=`은 덤으로 따라옴

---
# `===` 도 재정의 해 보자
> Note that the === operator **can’t** be overloaded.

그만 하기로 하자

---
# Ordering operators
- `<`, `>`, `<=`, `>=`
- Java의 `Comparable::compareTo`를 그대로 사용한다
    - (`operator` keyword를 사용하지 않는다)
- tip) kotlin엔 [compareValuesBy()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/compare-values-by.html) 가 제공된다. 2차, 3차 sort등을 구현할 때 유용
    - 고객정보를 국가별(Alphabetical)로 정렬하는데 같은 국가 안에서는 First name순으로

---
# Collections & Ranges
---
- index operator (a.k.a. square bracket) : `a[b]`
- Point의 property를 map처럼 접근하게 하려면? `operator` + `get`
```kotlin
data class Point(val x:Int, val y:Int)
operator fun Point.get(k:String) = when(k) {
    "x" -> x
    "y" -> y
    else -> throw IllegalArgumentException("Who are you?")
}

>>> val p = Point(1,2)
>>> p["x"]   // I know 'p.x' is a bit efficient :)
res21: kotlin.Int = 1
```
---
# 다차원도 OK

![](./assets/img/multi-dimen.png)

## mutable collection이라면 `set`도
![](./assets/img/mutable-collection.png)

---
# `in` convention
![](./assets/img/in-contains.png)

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)
operator fun Rectangle.contains(p: Point): Boolean {
    return p.x in upperLeft.x until lowerRight.x &&
    p.y in upperLeft.y until lowerRight.y
}

>>> val rect = Rectangle(Point(10, 20), Point(50, 50))
>>> println(Point(20, 30) in rect)
true
>>> println(Point(5, 5) in rect)
false
```
---
# `rangeTo` convention
![](./assets/img/range-to.png)

- `rangeTo()` 호출로 변환됨
- 단, `Comparable`하면 `rangeTo()`를 구현하지 않아도 된다
    - Kotlin standard library인 [rangeTo()](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/ranges/Ranges.kt#L87) 덕분이다.

---
### `iterator` convention in `for` loop

- `for(x in list)` 에서의 `in`은 '속하냐?'의 in과 다르다
- `for...in..` 문은 `list.iterator()`호출로 변환됨
- 잠시 Java 랑 비교
```java
for(char c : "Hello") { ... }
```
- 사실은 위의 자바코드는 [컴파일 안됨](https://gist.github.com/behumble/c21744303aa31a18952e88a74893f62f) 아래처럼 수정해야 함
```java
    for(char c : "Hello".toCharArray()) { ... }
```

---
- Kotlin standard library에서 [java.lang.CharSequence](https://docs.oracle.com/javase/8/docs/api/java/lang/CharSequence.html)에 [iterator()를 추가](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/text/Strings.kt#L273)한다
- 그래서 Kotlin에선 이것도 된다
```kotlin
for(c in "Hello") {
    println(c)
}
```

---
# Destructuring
- 하나의 composite value를 여러개의 별도 변수로 풀어 담기
- 괄호가 등장한다
    ```kotlin
    val p = Point(1,2)
    val (x,y) = p  // this is destructuring
    ```

- `componentN` convention

![height:100px](./assets/img/componentN.png)

---
# componentN convention

- data class는 primary constructor의 property 순서 기준으로 자동으로 생성
    ```kotlin
    class Point(val x: Int, val y: Int) {
        operator fun component1() = x
        operator fun component2() = y
    }
    ```
- array와 collection은 `component5()` 까지 제공된다
![](./assets/img/destructuring-6.png)
- 너무 귀찮으면 `Pair`와 `Triple` 을 애용해 주세요

---
# Delegated Properties

- 기본적인 Kotlin의 property는 단순히 backing field를 두고 getter, setter를 제공
- 고급기능이 필요하면 property getter,setter를 구현하면 된다
- standard delegated properties
    - lazy initialization
    - property listener
- #unique #powerful #advanced #난이도상

---
# Lazy property (무식ver.)

- email을 얻어오는건 I/O가 있어서 느리다 가정하자
    ```kotlin
    class Person(val name:String) {
        private var _emails:List<Email>? = null
        val emails: List<Email>
            get() {
                if(_emails==null) {
                    _emails = loadEmails(this) // slow
                }
                return _emails!! // not-null assertion
            }
    }
    val p = Person("Alan Goo") // quick
    println(p.emails) // slow
    ```
- 이런 property가 5개라면?

---
# `by lazy {}` 로 kotlin 스럽게
```kotlin
class Person(val name:String) {
    val emails by lazy { loadEmails(this) }
}
```
- `by`는 keyword. `lazy`는 function
- `loadEmails()` 가 한번만 호출 됨을 보장함 (synchronized 처리 포함)

---
## Property 변경 감지 in Kotlin way
```kotlin
class Person(val name:String, age:Int, salary:Int) {
    private val observer = {
        prop:KProperty<*>, oldValue:Int, newValue:Int
                    -> println("$prop:$oldValue->$newValue")
    }
    var age: Int by Delegates.observable(age, observer)
    var salary: Int by Delegates.observable(salary, observer)
}

val p = Person("Alan Goo", 26, 100 )
p.age = 36 // var Person.age: kotlin.Int:26->36
p.salary = 105 // var Person.salary: kotlin.Int:100->105
```

[star projection](https://kotlinlang.org/docs/reference/generics.html#star-projections)

---
# Delegated properties, again
```kotlin
class LoggingDelegate {
  operator fun getValue
    (thisRef: Any?, property: KProperty<*>):Long {
      println("LoggingDelegate.getValue : $thisRef . $property")
      return System.currentTimeMillis()
  }

  operator fun setValue
    (thisRef: Any?, property: KProperty<*>, v: Long) {
    println("LoggingDelegate.setValue : $thisRef . $property <- $v")
  }
}
```

---
```kotlin
class Person(age:Long) {
    var age: Long by LoggingDelegate()
}

val p = Person(26)
println("p.age : "+p.age)
// LoggingDelegate.getValue : Person@1d56ce6a . var Person.age: kotlin.Long
// p.age : 1571470775827
p.age = 100
LoggingDelegate.setValue : Person@1d56ce6a . var Person.age: kotlin.Long <- 100
```

---
# Delegated property
- delegated property가 설정되면 (with `by` keyword) backing field를 만들지 않고 hidden property로 delegate을 둔다 (`age$delegate`)
- delegate은 convention에 의해 `getValue()`(mutable property의 경우 : `setValue()` 까지)를 구현해야 한다

![height:150px](./assets/img/delegatedprop.png)
[](https://www.notion.so/aaeda5d82bc0430fbaac3925f0603dda#38b3439639ed4db89cf1f53c5abaad61)

---
# Map에서 property 읽어오기

```kotlin
var alanMap = mutableMapOf<String, String>("koreanName" to "구건")

class Person() {
    var koreanName: String by alanMap
    var englishName: String by alanMap
}

val p = Person()
println("koreanName : "+p.koreanName) // koreanName : 구건
p.koreanName = "건님"
println("koreanName : "+p.koreanName) // koreanName : 건님
println("English name : "+p.englishName)
// Exception in thread "main" java.util.NoSuchElementException: Key englishName is missing in the map.
//	at kotlin.collections.MapsKt__MapWithDefaultKt.getOrImplicitDefaultNullable(MapWithDefault.kt:24)
//	at Person.getEnglishName(Hello.kt)
//	at HelloKt.main(Hello.kt:20)
//	at HelloKt.main(Hello.kt)
```

---
# DB framework도 같은 방식으로

    object Users : IdTable() {
    	val name = varchar("name", length=50).index()
    	val age = integer("age")
    }
    
    class User(id:EntityID): Entity(id) {
    	var name:String by Users.name
    	var age:Int by Users.age
    }

- `Users`는 table을 나타내는 singleton
- `User`의 property들은 `Users.name`에게 넘긴다. `varchar()` 따위가 Delegate의 convention을 구현하는 object를 넘겨주겠지
