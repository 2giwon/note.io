## 리스트 종류

1. 선형 컬렉션
2. 연관 컬렉션
3. 그래프 컬렉션

## 리스트의 여러 유형

1. 접근 유형 
    1. 인덱스 (임의 원소에 접근 가능)
    2. 한쪽 끝 (한쪽 끝 원소만 접근 가능)
    3. 양쪽 끝 (양쪽 끝 원소 접근 가능)
2. 접근 순서
    1. FIFO
    2. LIFO
    3. 기타
3. 구현
    - 접근 유형과 순서에 따라 구현방식이 달라진다.

## 리스트의 예상 성능

### 1. 시간 복잡도와 공간 복잡도 맞바꾸기

- 원소 정렬 순서에 대한 고민
- 삽입 시점에 정렬할 것인가? 읽을 때 가장 작은 원소를 검색할 것인가?
- 읽은 원소를 데이터 구조로부터 체계적으로 삭제 할 수 없다면  삽입 시점 정렬이 낫다.
- 이런 방식은 우선순위 큐

***그러나 여러종류의 정렬 순서를 원한다면? 하나는 삽입된 순서, 하나는 역순으로 제공해야 된다면?***

- 각각의 접근 방법이 다르면 접근 시간도 다르다
- 리스트를 2개 유지하면 메모리를 소모하지만 속도면에서 이점을 볼 수 있다.

### 2. 제자리 상태 변이 피하기

1. 제자리 갱신 
    - 데이터 구조의 상태를 변이해 데이터 구조를 이루는 원소들을 바꾸는 방식.
    - 단일 스레드에서 이루어졌을 때 좋은 방식
    - 교착 상태, 라이브락, 스레드 기아상태 등 갖가지 문제가 있음

해결책 : 
- 불변 데이터 사용
- 새로운 원소를 삽입한 새로운 리스트 데이터를 생성

이러한 문제가 가지는 잠재적 문제

1. 다른 스레드에서 조작하는 데이터는 낡은 데이터
2. 메모리 낭비 성능 저하

위의 주장은 틀렸다. 

이러한 구조는 가변 데이터 구조에서 일어나는 문제를 방지할 수 있다.

## 코틀린에서 사용할 수 있는 리스트 종류

영속화 : 함수형프로그래밍에서는 데이터 구조에서 이전 버전이 파괴되지 않기 때문에 데이터가 메모리 상에 계속 존재한다는 의미에서 '영속성'이라는 용어를 사용한다.

### 영속적인 데이터 구조 사용하기

- 원소를 추가하기 전에 복사본을 만드는 것은 성능 저하
- 데이터 공유를 사용해야 한다.
- 불변이면서 영속적인 데이터구조의 기법
- 복사가 일어나지 않고 데이터를 추가 삭제할 수 있음.

### 불변이며 영속적인 단일 연결리스트 구현

재귀적인 리스트 구조

1. 리스트의 첫번째 원소가 될 원소가 필요하다. 이를 head라고 부른다.
2. 리스트의 나머지 부분이 필요하다. 이 나머지 부분은 그 자체로도 독립적인 리스트이며, 이를 tail이라고 부른다.

```kotlin
open class Pair<A, B> (val first: A, val second: B)

class List<A> (val head: A, val tail: List<A>): Pair<A, List<A>>(head, tail)
```

재귀 정의를 종료하기 위한 조건

- 빈 리스트 또는
- 원소 하나와 다른 리스트 쌍

Nil을 List<Nothing> 타입으로 만들면 Nil을 어떤 리스트 타입으로 든 변환 할 수 있다.

```kotlin
open class List<A>

object Nil : List<Nothing>()

class Cons<A>(private val head: A, private val tail: List<A>): List<A>()
```

위 방식의 단점

- 확장이 쉽다.
- 리스트 구현의 일관성이 없어짐
- 노출 해서는 안되는 Nil, Cons 하위 클래스에 쉽게 접근이 가능
- 해법

```kotlin
sealed class List<A> {
	internal object Nil: List<Nothing>()
	internal class Cons<A>(private val head: A, private val tail: List<A>): List<A>()
}
```

예제

```kotlin
sealed class List<A> {                  // sealed 클래스는 암묵적으로 추상클래스 이며, 생성자는 암묵적으로 비공개이다.
    abstract fun isEmpty(): Boolean     // 각 확장 클래스는 추상 isEmpty 함수를 다르게 구현한다.

    private object Nil: List<Nothing>() {
        override fun isEmpty(): Boolean = true
        override fun toString(): String = "[Nil]"
    }

    private class Cons<A> (internal val head: A,
                           internal val tail: List<A>): List<A>() {     // 비어 있지 않은 리스트를 표현하는 Cons확장 클래스
        override fun isEmpty(): Boolean = false
        override fun toString(): String = "[${toString("", this)}NIL]"
        private tailrec fun toString(acc: String, list: List<A>): String =
                when (list) {               // 공재귀 함수로 toString을 구현한다.
                    is Nil -> acc
                    is Cons -> toString("$acc${list.head}, ", list.tail)
                }
    }
    
    companion object {
        operator 
        fun <A> invoke(vararg az: A): List<A> = // operator 키워드를 사용해 선언한 invoke 함수는 클래스 이름()처럼 호출이 가능하다.
                // foldRight 함수의 첫 번째 인자로 쓰이는 Nil을 List<A>로 명시적으로 타입을 변환한다.
                az.foldRight(Nil as List<A>) { a: A, list: List<A> -> Cons(a, list)}  
        
        
    }

}
```

- **봉인된 클래스**는 하위 클래스 타입을 제한하기 때문에 **추상적 데이터 타입**을 정의 할 수 있다.
- Cons 두 인자를 내부(internal) 가시성으로 선언해 List클래스가 선언된 모듈이나 파일 안에서만 볼 수 있음
- invoke를 통해서만 리스트를 만들게 강제함
- [NIL]을 마지막에 나오도록 함

## 리스트 연산에서 데이터 공유하기

연습문제 5-1

```kotlin
fun cons (a: A): List<A> = Cons(a, this)
```

연습문제 5-2

```kotlin
fun setHead(a: A): List<A> = (this as Cons).tail.cons(a)
```

이런 식으로 tail에 있는 새로운 리스트를 생성하여 그 리스트에 head 값에 새로운 값을 넣고 그 값 뒤에 리스트를 붙인다.

```kotlin
fun setHead(a: A): List<A> = when(this) {
        is Nil -> throw IllegalStateException("setHead called on an empty list")
        is Cons -> tail.cons(a)
    }
```

그러나 리스트가 빈 리스트일 수도 있기 때문에 예외 처리를 추가해준다.

## 다른 리스트 연산들

연습문제 5-3

n개의 원소를 제거하는 drop함수를 만들자.

```kotlin
abstract fun drop(n: Int): List<A>
...

private object Nil: List<Nothing>() {
  ...      
	override fun drop(n: Int): List<Nothing> = this
}

private class Cons<A> (internal val head: A, 
											internal val tail: List<A>): List<A>() {
	...
	override fun drop(n: Int): List<A> {
		return if (n == 0) this else tail.drop(n - 1)
  }
}
```

위와 같이 구현을 하면 n이 커지면 스택 오버 플로우가 발생한다.

공재귀로 바꾸어야 한다.

```kotlin
override fun drop(n: Int): List<A> {
	tailrec fun drop(n: Int, list: List<A>): List<A> =
			if (n <= 0) list else when (list) {
				is Cons -> drop(n - 1, list.tail)
				is Nil -> list
			}
	return drop(n, this)
}
```

### 객체 표기법의 이점 살리기

drop 함수를 클래스 외부로 빼기

```kotlin
class List<A> {
...
}

fun <A> drop(aList: List<A>, n: Int): List<A> {
	tailrec fun drop_(list: List<A>, n: Int): List<A> = when (list) {
			List.Nil -> list
			is List.Cons -> if (n <= 0) list else drop_(list.tail, n - 1)
	}
	return drop_(aList, n)
}
```

도우미 함수 제거

```kotlin
tailrec fun drop(list: List<A>, n: Int): List<A> = when (list) {
		List.Nil -> list
		is List.Cons -> if (n <= 0) list else drop(list.tail, n - 1)
}
```

클래스 동반 객체 정의

```kotlin
companion object {
		tailrec fun drop(list: List<A>, n: Int): List<A> = when (list) {
				List.Nil -> list
				is List.Cons -> if (n <= 0) list else drop(list.tail, n - 1)
		}
}
```

맨 앞 원소 2개 제거후 결과로 얻은 객체의 첫번째 원소 0으로 바꾸기

```kotlin
val newList = setHead(drop(list, 2), 0)
```

연습문제 5-4

```kotlin
private tailrec fun <A> dropWhile(list: List<A>, p: (A) -> Boolean): List<A> = when (list) {
		Nil -> list
		is Cons -> if (p(list.head)) dropWhile(list.tail, p) else list
}
```

### 리스트 연결

list 1을 list2에 연결하려면 

- list1의 맨마지막 리스트를 list2의 맨 앞에 추가하고
- list1의 맨 마지막에서 두번째 원소를 다시 추가한다.

concat함수를 정의하는 법

- list1이 비어있다면 list2를 반환
- list1이 비어있지 않다면 list1의 꼬리(list1.tail)에 list2를 연결한 리스트 앞에 list1의 첫번째 원소(list1.head)를 추가한 새 리스트를 반환

```kotlin
fun <A> concat(list1: List<A>, list2: List<A>): List<A> = when(list1) {
		Nil -> list2
		is Cons -> concat(list1.tail, list2).cons(list1.head)
}
```

this를 인자로 넣어 companion object에 있는 인스턴스 함수를 list에 추가할 수 있다.

```kotlin
fun concat(list: List<A>): List<A> = concat(this, list)
```

이 해법을 이해하려 하지 마라!

이 코드는 수학적 정의를 옮긴 것 뿐

위의 방법은 길이의 제약이 있으나 공재귀로 바꾸는 것은 불가능하다. 

추상화 연산으로 재사용성을 높일 수 있다.

### 리스트 끝에서부터 원소 제거하기

연습문제 5-5

마지막 원소를 제거하는 함수를 작성하라.

```kotlin
fun init() : List<A>
```

도우미 함수 생성

```kotlin
tailrec fun <A> reverse(acc: List<A>, list: List<A>): List<A> = 
		when (list) {
				Nil -> acc
				is Cons -> reverse(acc.cons(list.head), list.tail)
		}
```

```kotlin
fun reverse(): List<A> = reverse(List.invoke(), this)
```

Companion Object라서 List()라는 구문을 쓸 수 없어서 invoke()를 쓴다.

```kotlin
fun init(): List<A> = reverse().drop(1).reverse()
```

### 재귀와 고차 함수로 리스트 접기

연습문제 5-6

재귀를 사용해 정수 원소로 이뤄진 영속적 리스트의 모든 원소 합계를 구하는 함수를 작성하라.

- 빈 리스트의 합계는 0
- 원소가 있으면 head + (sum(tail))

```kotlin
fun sum(ints: List<Int>): Int = when (ints) {
		Nil -> 0
		is Cons -> ints.head + sum(ints.tail)
}
```

### 변성 사용하기

- List<Nothing> 을 List<Int>로 변환 할 수 없다.
- List를 A에 대해 공변이 되게 만들어야 한다.
- List<out A> 정의

```kotlin
sealed class List<out A> {
...
}
```

위의 코드는 아래 코드에서 오류를 발생시킨다.

```kotlin
fun cons(a: A): List<A> = Cons(a, this)
```

- List<A>타입을 파라미터로 받는 함수를 포함할 수 없다
- 함수는 A를 반환 타입에만 쓸 수 있다.

### 변성 이해하기

사과 바구니

- 사과를 바구니에 넣는다.
- 사과를 바구니에서 꺼낸다.

Apple 은 Fruit (참)

Gala는 Apple (참)

Fruit은 Apple이 아니다.

Apple은 Gala가 아니다.

Gala를 Apple바구니 안에(`in`) 넣을 수 있다.

Fruit는 Apple바구니 안에 넣을 수 없다.

반대로 Fruit이 필요할 때 Apple 바구니에서 꺼낼 수 있다.

하지만 Gala가 필요할 때 Apple 바구니에서 꺼낼 수 없다.

빈바구니에는 다 넣을 수 있다.

Any가 빈바구니

하지만 빈바구니에서 꺼낸 것이 어떤 종류의 물건인지 모른다.

Nothing타입은 모든 타입의 하위타입

A 타입 파라미터를 out으로 선언함으로 List<Gala>가 List<Apple>로 할 수 있다.

빈 리스트를 List<Nothing>로 선언하면 된다. 

Nothing 은 부재(아무것도 없음)으로 이해 할 수 있다.

### 변성에 어긋나는 활용 방지

```kotlin
sealed class List<out A> {
	fun cons(a: A): List<A> = Cons(a, this) {
		...
	}

	internal class Cons<out A>(internal val head: A, internal val tail: List<A>): List<A>()
	internal object Nil: List<Nothing>()
}
```

문제 없어 보이지만 컴파일 오류

```kotlin
sealed class List<out A> {
	abstract fun cons(a: A): List<A> = Cons(a, this) {
		...
	}

	internal class Cons<out A>(internal val head: A, internal val tail: List<A>): List<A>()
	internal object Nil: List<Nothing>() {
		override fun cons(a: Nothing): List<Nothing> = Cons(a, this) // 오류
	}
}
```

'도달 할 수 없는 코드' 컴파일 오류

Nil 클래스에 있는 this는 List를 가리키며, Nil.cons(1) 은 1을 Nothing으로 타입 변환 하기 때문.

코틀린의 타입 추론 과정

- List<Nothing> 인 this를 안전하게 List<A>로 변환 할 수 있다.
- override fun cons(a: Nothing)
- 함수에 전달될 때 타입이 변환되기 때문에 오류가 발생

해결 방법

- @UnsafeVariance 어노테이션 사용

```kotlin
fun cons(a: @UnsafeVariance A): List<A>
```

- 구현을 부모 클래스 안에 넣어서 A가 하위 타입으로 캐스팅 되지 않도록 함.

```kotlin
sealed class List<out A> {
	fun cons(a: @UnsafeVariance A): List<A> = Cons(a, this)
	...
}
```

setHead, concat도 동일하게 적용

```kotlin
fun cons(a: @UnsafeVariance A): List<A> = Cons(a, this)
fun setHead(a: @UnsafeVariance A): List<A> = when(this) {
    is Nil -> throw IllegalStateException("setHead called on an empty list")
    is Cons -> Cons(a, this.tail)
}

fun concat(list: List<@UnsafeVariance A>): List<A> = concat(this, list)
```

- 안전하지 못한 타입 변환이 실패하지 않음을 확인 해야됨!

- 빈 리스트 구현 방식

```kotlin
abstract fun concat(list: List<A>): List<A>

abstract class Empty<A>: List<A>() {
    override fun concat(list: List<A>): List<A> = list
}

private object Nil : Empty<Nothing>()

private class Cons<A> (internal val head: A,
                           internal val tail: List<A>): List<A>() {
	override fun concat(list: List<A>): List<A> = Cons(this.head, list.concat(this.tail))
}
```

연습 문제 5-7 

double의 리스트에 들어 있는 모든 원소의 곱을 계산하는 product함수를 작성하라.

```kotlin
fun product(nums: List<Double>): Double = when (nums) {
      List.Nil -> 1.0
      is List.Cons -> nums.head * product(nums.tail)
}
```

곱셈은 흡수 원소를 고려해야 된다.

```kotlin
a * 흡수 원소 = 흡수 원소 * a = 흡수 원소
```

- 영원소 : 어떤 연산의 흡수 원소
- 영원소를 이용한 코드

```kotlin
fun product(nums: List<Double>): Double = when (nums) {
    List.Nil -> 1.0
    is List.Cons -> if (nums.head == 0.0) 
                        0.0
                    else
                        nums.head * product(nums.tail)
}
```

sum 함수

```kotlin
fun sum(nums: List<Int>): Int = when (nums) {
      List.Nil -> 0
      is List.Cons -> nums.head + sum(nums.tail)

}
```

두 개의 함수를 합치기

```kotlin
fun <A> operation(list: List<A>, identity: A, operator: (A) -> (A) -> (A)): A = 
        when (list) {
            List.Nil -> identity
            is List.Cons -> operator (list.head) (operation(list.tail, identity, operator))
        }

```

결과 타입을 다른 타입으로 넘기기

```kotlin
fun <A, B> operation(list: List<A>, identity: B, operator: (A) -> (B) -> (B)): B =
        when (list) {
            List.Nil -> identity
            is List.Cons -> operator (list.head) (operation(list.tail, identity, operator))
        }
```

foldRight 구현과 foldRight를 사용한 sum과 product 구현

```kotlin
fun <A, B> foldRight(list: List<A>,             // A, B 타입을 나타낸다
                        identity: B,                // 접기 연산의 항등원
                        f: (A) -> (B) -> B): B =     // 연산자를 표현하는 함수로 커리한 형태
        when (list) {
            List.Nil -> identity
            is List.Cons -> f(list.head)(foldRight(list.tail, identity, f))
        }

fun sum (list: List<Int>): Int = 
        foldRight(list, 0 ) { x -> { acc -> x + acc } }

fun product(list: List<Double>): Double = 
        foldRight(list, 1.0) { x -> { acc -> x * acc }}
```

foldRight는 단일 연결리스트와 비슷하다.

```kotlin
Cons(1, Cons(2, Cons(3, Nil)))          // Nil이 항등원
```

```kotlin
foldRight(List(1, 2, 3), List()) { x: Int ->
		{ acc: List<Int> ->
				acc.cons(x)
		}
}
```

출력 결과 확인

```kotlin
println(foldRight(List(1, 2, 3), List()) { x: Int ->
		{ acc: List<Int> ->
				acc.cons(x)
		}
})
```

출력 결과

```kotlin
[1, 2, 3, NIL]
```

연습 문제 5-8

리스트의 길이를 계산하는 함수를 작성하라. foldRight를 사용해 작성하라.

```kotlin
fun length(): Int = foldRight(0) { { it + 1} }
```

연습 문제 5-9

공재귀 함수인 foldLeft를 만들어라

```kotlin
fun <B> foldLeft(identity: B, f: (B) -> (A) -> B): B
```

```kotlin
tailrec fun <A, B> foldLeft(acc: B, list: List<A>, f: (B) -> (A) -> B): B =    
        when (list) {
            List.Nil -> acc
            is List.Cons -> foldLeft(f(acc)(list.head), list.tail, f)
        }
```

연습 문제 5-10

foldLeft 함수를 이용해 스택을 안전하게 사용하는 sum, product, length함수를 작성하라.

```kotlin
fun sum2(list: List<Int>): Int = 
        foldLeft(0, list) { acc -> { y -> acc + y } }
    
fun product2(list: List<Double>): Double = 
        foldLeft(1.0, list) { acc -> { y -> acc * y } }

fun length2(): Int = foldLeft(0) { acc -> { _-> acc + 1 } }
```

프로덕션 코드에 사용해서는 안된다?

연습 문제 5-11

foldLeft 를 사용해 리스트를 뒤집는 함수를 작성하라.

항등원 타입이 없기 때문에 컴파일 단계에서 warning이 발생한다.

```kotlin
fun reverse2(): List<A> = foldLeft(Nil as List<A>) { acc -> { acc.cons(it) } }
```

여기서 Nil이 항등원이다.

```kotlin
fun reverse2(): List<A> = foldLeft(invoke()) { acc -> { acc.cons(it) } }
```

invoke()로 리스트를 생성한다.

연습 문제 5-12

foldLeft를 사용해 foldRight를 정의 하라.

```kotlin
fun <B> foldRightViaFoldLeft(identity: B, f: (A) -> (B) -> B): B =
            this.reverse2().foldLeft(identity) { acc -> { y -> f(y)(acc)} }
```

위의 코드는 reverse를 통해 미리 계산이 되어 지연 계산 시 이점이 없다.

## 스택을 안전하게 사용하는 foldRight 만들기

연습문제 5-13

foldLeft를 명시적으로 제공하지 않고, 공재귀 foldRight 함수를 구현하라. 이 함수를 coFoldRight라고 부르자.

```kotlin
private tailrec fun <A, B> coFoldRight(acc: B, list: List<A>, identity: B, f: (A) -> (B) -> B): B =
            when (list) {
                List.Nil -> acc
                is List.Cons -> coFoldRight(f(list.head)(acc), list.tail, identity, f)
            }
```

```kotlin
fun <B> coFoldRight(identity: B, f: (A) -> (B) -> B): B =
        coFoldRight(identity, this.reverse2(), identity, f)
```

연습 문제 5- 14

concat을 foldLeft나 foldRight로 정의하라. 

```kotlin
fun <A> concatViaFoldRight(list1: List<A>, list2: List<A>): List<A> =
            foldRight(list1, list2) { x -> { acc -> Cons(x, acc) }}
```

```kotlin
fun <A> concatViaFoldLeft(list1: List<A>, list2: List<A>): List<A> =
            list1.reverse2().foldLeft(list2) { acc -> acc::cons }
```

연습 문제 5-15

리스트의 리스트를 받아서 내포된 리스트에 들어 있는 모든 원소를 펼친 평평한 리스트를 반환 하는 함수를 만들어라.

```kotlin
fun <A> flatten(list: List<List<A>>): List<A> = 
		list.foldRight(invoke()) { x -> { acc: List<A> -> x.concat(acc) } }
```

## 리스트 매핑과 필터링

연습 문제 5-16

정수 리스트를 받아서 각 원소에 3울 곱한 리스트를 반환하는 함수를 작성하라.

```kotlin
fun triple(list: List<Int>): List<Int> =
        List.foldRight(list, List.invoke()) {
            { acc : List<Int> ->
                acc.cons(it * 3)
            }
        }
```

연습 문제 5-17

List<Double> 의 모든 원소를 String으로 변환하는 함수를 작성하라.

```kotlin
fun doubleToString(list: List<Double>): List<String> =
    List.foldRight(list, List()) { head ->
        { acc: List<String> ->
            acc.cons(head.toString())
        }
    }
```

연습 문제 5-18

스택을 안전하게 사용하는 map 함수를 list클래스에 정의하라.

fun <B> map (f: (A) → B) : List<B>

```kotlin
fun <B> map(f: (A) -> B): List<B> =
        foldRight(invoke()) { head ->
            { acc: List<B> ->
                Cons(f(head), acc)
            }
        }
```

스택을 더 안전하게 쓰는법

```kotlin
fun <B> map2(f: (A) -> B): List<B> =
      coFoldRight(invoke()) { head ->
          { acc: List<B> ->
              Cons(f(head), acc)
          }
      }
```

연습 문제 5-19

주어진 술어를 만족하지 않으면 제거하는 filter함수를 작성하라.

`fun filter(p: (A) → Boolean): List<A>`

```kotlin
fun filter(p: (A) -> Boolean): List<A> =
        foldRight(invoke()) { head ->
            { acc: List<A> ->
                if ( p(head) ) Cons(head, acc) else acc
            }
        }
```

책에서는 Nil을 넣으라고 되어있는데 invoke()로 넣어야 실행이 된다.

연습 문제 5-20

List<A> 타입인 리스트의 각 원소에 대해 A를 List<B>로 변환하는 함수를 적용한 다음, 결과 리스트들을 모두 연결해 평평하게 만든 List<B>를 반환하는 flatMap함수를 작성하라.

```kotlin
fun <B> flatMap(f: (A) → List<B>) : List<B>
```

```kotlin
fun <B> flatMap(f: (A) -> List<B>) : List<B> = flatten(map(f))
```

연습 문제 5-21

flatmap으로 새로운 filter를 작성하라

```kotlin
fun filter2(p: (A) -> Boolean): List<A> = 
		flatMap { condition -> if (p(condition)) List(condition) else invoke() }
```

- map 은 리스트를 변환하여 리스트로 반환한다.
- flatten은 리스트의 모든 원소를 포함하는 리스트를 반환한다.
- flatmap은 map → flatten 과 동일하다.
