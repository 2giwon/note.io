[Kotlin Collection Functions Cheat Sheet](https://medium.com/@elye.project/kotlin-collection-functions-cheat-sheet-975371a96c4b)

코틀린 컬렉션 유틸 함수의 5가지 카테고리

1. Creation - 컬렉션을 만드는 기능 (ex : listOf)
2. Convert - 다른 유형으로 캐스트하는 기능 (ex: asMap)
3. Change - 내용을 변환하는 기능 (ex: map)
4. Choose - 항목중 하나에 액세스하는 기능 가져오기
5. Conclude - 항목에서 무언가를 생성하는 기능(ex: sum)

![https://miro.medium.com/max/1400/1*0Cm0aYBWChlOhEpbOPuWSA.png](https://miro.medium.com/max/1400/1*0Cm0aYBWChlOhEpbOPuWSA.png)

## Example 1

목록의 모든 정수 항목 값의 곱을 수행하는 함수를 찾으려면,

![https://miro.medium.com/max/231/1*0pSsPYTlaiPbyTYdT5bbhg.png](https://miro.medium.com/max/231/1*0pSsPYTlaiPbyTYdT5bbhg.png)

결과 값이 하나로 합쳐지므로 Conclude 입니다.

결과를 보면 `reduce`와 동일

```kotlin
list.reduce{ result, item -> result * item }
```

## Example 2

함수를 찾으려면 list를 더 작은 고정 크기의 여러 하위 list로 나눕니다.

![https://miro.medium.com/max/982/1*UrZ6f7IxZ4zudtdlBNR9vw.png](https://miro.medium.com/max/982/1*UrZ6f7IxZ4zudtdlBNR9vw.png)

결과 값이 다른 형식의 컬렉션 이므로 Change 입니다.

`Chunk`로 대체할 수 있습니다.

```kotlin
list.chunked(3)
```

## 카테고리 간의 관계

1. 첫번째는 Creation
2. Change, Convert는 중간
3. Conclude, Choose는 최종 단계

예를 들면

```kotlin
listOf(1, 2, 3)   // Creation
.map { it * 2 }   // Change
.sum()            // Conclude
```

첫번째 단계는 중간 을 건너뛰고 최종으로 갈 수 있음.

```kotlin
listOf(1, 2, 3)   // Creation
.sum()            // Conclude
```

![https://miro.medium.com/max/1282/1*Qm5bvp517vrSGodjHHi1GQ.png](https://miro.medium.com/max/1282/1*Qm5bvp517vrSGodjHHi1GQ.png)

Note : 일부 함수에는 By, To 및 With가 추가 된 변형 도우미 기능이 있습니다.

[Kotlin Collection's appended To, By, and With](https://proandroiddev.com/kotlin-collections-appended-to-by-and-with-205d9540208)

[Understand Kotlin Past Participle Named Collection Function](https://medium.com/@elye.project/understand-kotlin-collection-function-past-tense-59f592af9436)

# 카테고리 생성

하위 카테고리로 세분화하여 리스트 검색

1. Creation Compose - 새 컬렉션을 인스턴스화
2. Creation Copy - 컬렉션 복제
3. Creation Catch - try-catch 와 같은 방식으로 생성

## 1. Creation Compose - 새 컬렉션을 인스턴스 화

컬렉션을 새로운 인스턴스로 생성하는 데 도움이 되는 기능

```kotlin
// Empty Collection
emptyList, emptyMap, emptySet
// Read-only Collection
listOf, mapOf, setOf
// Mutable Collection
mutableListOf, mutableMapOf, mutableSetOf, arrayListOf
// Build Collection from mix sources
buildList, buildMap, buildSet
// Linked Collection
linkedMapOf, linkedSetOf (more in stackOverflow)
// Sorted Collection
sortedMapOf, sortedSetOf (more in stackOverflow)
// Hash Collection
hashMapOf, hashSetOf (more in stackOverflow)
// Programmatically create Collection
List, MutableList, Iterable
```

## 2. 컬렉션 복제

```kotlin
copyInto     // Can into array
copyOfRange  // Partially copy
copyOf       // Copy fully
toCollection // Copy into collection
```

## 3. Creation Catch - try-catch 와 같은 방식으로 생성

```kotlin
ifEmpty         // if empty give a default
orEmpty         // change null into empty
requireNoNulls  // crash if any element is null
listOfNotNull   // make single element list or null
```

# 카테고리 변환

컬렉션 유형을 다른 유형으로 변경

1. Conversion Copy - 다른 유형의 새 컬렉션으로 변환
2. Conversion Cite - 원본을 참조하여 다른 유형으로 전환

ex) toIntArray 및 asIntArray(Cite)

![https://miro.medium.com/max/1400/1*kA1S5zVE7wkqS2M_FTnpOQ.png](https://miro.medium.com/max/1400/1*kA1S5zVE7wkqS2M_FTnpOQ.png)

```kotlin
// toIntArray example (a new copy)
val uIntArray = UIntArray(3) { 1U }
val toIntArray = uIntArray.toIntArray()
toIntArray[1] = 2
println(toIntArray.toList())  // [1, 2, 1]
println(uIntArray.toList())   // [1, 1, 1]
// asIntArray example (a reference copy)
val uIntArray = UIntArray(3) { 1U }
val asIntArray = uIntArray.asIntArray()
asIntArray[1] = 2
println(asIntArray.toList())  // [1, 2, 1]
println(uIntArray.toList())   // [1, 2, 1]
```

## 1. Conversion Copy

```kotlin
// to array type
toBooleanArray, toByteArray, toCharArray, toDoubleArray, 
toFloatArray, toIntArray, toLongArray, toShortArray, 
toTypedArray, toUByteArray, toUIntArray, toULongArray, toUShortArray
// to read-only collection
toList, toMap, toSet
// to mutable collection
toMutableList, toMutableMap, toMutableSet, toHashSet
// to sorted collection
toSortedMap, toSortedSet
// Convert Entries to Pair
toPair       // More illustration below
// Convert Map to Properties
toProperties // More illustration below
```

```kotlin
map.entries.map { it.toPair() }
// This is essentially 
map.toList()
// Underlying `toList` of Map, it is using `toPair()`
// to do all the conversion of entries to Pair
```

`toProperties` 로 속성을 map으로 변환(각 원소를 반환)

```kotlin
val map = mapOf("x" to "value A", "y" to "value B")
val props = map.toProperties()
println(props.getProperty("x"))         // value A
println(props.getProperty("z"))         // null
println(props.getProperty("y", "fail")) // value B
println(props.getProperty("z", "fail")) // fail
println(map.get("x"))                   // value A
println(map.get("z"))                   // null
println(map.getOrDefault("y", "fail"))  // value B
println(map.getOrDefault("z", "fail"))  // fail
```

## 2. Conversion Cite - 원본 변환

```kotlin
// as array type
asByteArray, asIntArray, asLongArray, asShortArray, 
asUByteArray, asUIntArray, asULongArray, asUShortArray,
// as collection type. For list vs sequestion, check this blog
asIterable, asList, asSequence
// Convert to indexed iterator
withIndex    // More illustration below
// Convert to Map with customized default
withDefault  // More illustration below
```

`withIndex`를 사용하면 원본 참조의 indexedValue iterable로 변환

```kotlin
val list = listOf("A", "B", "C")
val indexed = list.withIndex()
println(list)  // [A, B, C]
println(indexed.toList())
// [IndexedValue(index=0, value=A), 
//  IndexedValue(index=1, value=B), 
//  IndexedValue(index=2, value=C)]
```

`withDefault`는 커스텀 DefaultValue와 함께 map으로 변환

```kotlin
val map = mutableMapOf(1 to 1)
// customize default value return x 2 of key
val default = map.withDefault { k -> k * 2 }
println(default.getValue(10)) // return 20
println(map.getValue(10))     // crash as no key 10
```

# 카테고리 변경

- 컬렉션 내용을 변경
- 다른구조로 새로운 컬렉션 생성

1. Change Content - 원본은 그대로 유지하고 타입도 그대로 (int → int) 유지하며 내용만 변경된 새로운 컬렉션으로 변경 (ex) filter
2. Change Contour - 컬렉션 타입을 변경하거나 결과 컬렉션의 타입이 변경 (int → List<Int>) (ex) map, groupBy

![https://miro.medium.com/max/1400/1*ebtnr1q3jcKRT5QCmJxznQ.png](https://miro.medium.com/max/1400/1*ebtnr1q3jcKRT5QCmJxznQ.png)

## 1. Change Content

- 내용을 변경한 새 리스트 반환
- 원본의 내용을 변경, 반환하지는 않음

ex) add, plus

```kotlin
val list = listOf(1)
val mutableList = mutableListOf(1)

println(list)          // [1]
println(mutableList)   // [1]

val newList = list.plus(2)
mutableList.add(2)

println(list)          // [1]
println(newList)       // [1, 2]
println(mutableList)   // [1, 2]
```

![https://miro.medium.com/max/1124/1*faXcTXsC1LIpVtV32KbXNA.png](https://miro.medium.com/max/1124/1*faXcTXsC1LIpVtV32KbXNA.png)

1. mutable한 italic 처리를 하는 함수 생성(add)
2. non-italic 한 처리를 하여 새로운 리스트를 반환하는 함수 생성 (plus

```kotlin
// Changing content
set, setValue //(more info in stackoverflow)

// Adding content
plus, plusElement, //(more info in stackoverflow)
plusAssign, //(more info in stackoverflow)
add, addAll, put, putAll

// Removing content
minus, minusElement, //(more info in stackoverflow)
minusAssign, //(more info in stackoverflow)
remove

// Remove away from the front or behind of the collection
drop, dropLast, dropLastWhile, dropWhile,
removeFirst, removeFirstOrNull, removeLast, removeLastOrNull,
```

![https://miro.medium.com/max/1400/1*zIYC8g4cBpptpgxi8at6sA.png](https://miro.medium.com/max/1400/1*zIYC8g4cBpptpgxi8at6sA.png)

```kotlin
// Pick from the front or behind part of the collection
take, takeLastWhile, takeLast, takeWhile,
```

![https://miro.medium.com/max/1400/1*Konpc1nmrDSP3RtzPGJWsw.png](https://miro.medium.com/max/1400/1*Konpc1nmrDSP3RtzPGJWsw.png)

```kotlin
// Pick from the center section of the collection
slice, sliceArray
```

![https://miro.medium.com/max/1400/1*j9Y235Xag019VNolGSVsKA.png](https://miro.medium.com/max/1400/1*j9Y235Xag019VNolGSVsKA.png)

```kotlin
// Get distinct (unique) value only
distinct, distinctBy
```

![https://miro.medium.com/max/1400/1*JBovyymvIxzxMe7laYab8g.png](https://miro.medium.com/max/1400/1*JBovyymvIxzxMe7laYab8g.png)

```kotlin
// Perform venn diagram operation given two collections
union, intersect retainAll, subtract removeAll
```

![https://miro.medium.com/max/2000/1*5vh8i58njkCfPGp67vv5HA.png](https://miro.medium.com/max/2000/1*5vh8i58njkCfPGp67vv5HA.png)

```kotlin
// Transform the content to another value
map, mapTo, mapIndexed, mapIndexedTo, mapKeys, mapKeysTo, mapValues, 
mapValuesTo, replaceAll, fill

// Transform the content and remove all null
// to have non-nullable result of content
mapNotNull, mapNotNullTo, mapIndexedNotNull, mapIndexedNotNullTo
```

![https://miro.medium.com/max/1400/1*3rURnAcwo4cHrLIiV7GY2Q.png](https://miro.medium.com/max/1400/1*3rURnAcwo4cHrLIiV7GY2Q.png)

> *Note: `map` function can also change the list to another form or different type (i.e. Change Contour subcategory)*

```kotlin
// Filter some content away
filter, filterIndexed, filterIndexedTo, filterIsInstance, 
filterIsInstanceTo, filterKeys, filterNot, filterNotNull, 
filterNotNullTo, filterNotTo, filterTo, filterValues
```

![https://miro.medium.com/max/1400/1*23-S4LdzpEYlch1H9mNVvw.png](https://miro.medium.com/max/1400/1*23-S4LdzpEYlch1H9mNVvw.png)

```kotlin
// Reverse the content value 
// (Checkout the article for their differences)
reversed, reversedArray, reverse, asReversed
```

![https://miro.medium.com/max/1322/1*3RtoE0n6gMvWFOcHhG8TPw.png](https://miro.medium.com/max/1322/1*3RtoE0n6gMvWFOcHhG8TPw.png)

```kotlin
// Sort the content value
sorted, sortedArray, sortedArrayDescending, sortedArrayWith, 
sortedBy, sortedByDescending, sortedDescending, sortedWith sort, 
sortBy, sortByDescending, sortDescending, sortWith

// Randomize the content sequence
shuffle, shuffled
```

![https://miro.medium.com/max/1362/1*U5dFuiUZVO9JxL5N2jvtZg.png](https://miro.medium.com/max/1362/1*U5dFuiUZVO9JxL5N2jvtZg.png)

```kotlin
// Like fold or reduce, but done on each elements incrementally
scan, scanIndexed, scanReduce, scanReduceIndexed
```

![https://miro.medium.com/max/1366/1*MaVpcuHyInEP-04_EZ2_wQ.png](https://miro.medium.com/max/1366/1*MaVpcuHyInEP-04_EZ2_wQ.png)

## 2. Change Contour

타입 변경된 새로운 리스트 반환

```kotlin
// Form a map of aggregated items
aggregate, aggregateTo //(require groupingBy)

// Example
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9)
val aggregated = numbers.groupingBy { it % 3 }
    .aggregate { key, accumulator: Int?, element, first ->
        if (first) element else accumulator?.plus(element)
    }
println(aggregated) // {1=12, 2=15, 0=18}
```

![https://miro.medium.com/max/1182/1*xO-SoeHMEi_bPS4NcaLUug.png](https://miro.medium.com/max/1182/1*xO-SoeHMEi_bPS4NcaLUug.png)

```kotlin
// Form a map of number of related items
eachCount, eachCountTo //(require groupingBy)

// Example Code
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9)
val eachCount = numbers.groupingBy { it % 2 }.eachCount()
println(eachCount) // {1=5, 0=4}
```

![https://miro.medium.com/max/982/1*qGJE4I2lNRSVBCTwRn2osg.png](https://miro.medium.com/max/982/1*qGJE4I2lNRSVBCTwRn2osg.png)

```kotlin
// Link each item with a map key
associate, associateBy, associateByTo, associateTo, 
associateWith, associateWithTo //(check this article for their differences)

// Example code
val list = listOf(1, 2, 3, 4, 5)
val associate = list.associateWith { it * it }
println(associate) // {1=1, 2=4, 3=9, 4=16, 5=25}
```

![https://miro.medium.com/max/1182/1*HoPU5ZEt7u9d4fKG2PwHdQ.png](https://miro.medium.com/max/1182/1*HoPU5ZEt7u9d4fKG2PwHdQ.png)

```kotlin
// Group related items together into map of List
groupBy, groupByTo

// Example code
val list = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9)
val groupBy = list.groupBy{it % 3}
println(groupBy)
```

![https://miro.medium.com/max/1182/1*tBg4zj4a4sgQCn08jMP97g.png](https://miro.medium.com/max/1182/1*tBg4zj4a4sgQCn08jMP97g.png)

```kotlin
// Flatten into single list.
flatMap, flatMapTo, flatten

// Example code
val map = mapOf(1 to listOf(1, 2), 2 to listOf(2, 4))
val flatMap = map.flatMap { it.value }
println(flatMap)  // [1, 2, 2, 4]

// Example code
val list = listOf(listOf(1, 2), listOf(2, 4))
val flatten = list.flatten()
println(flatten)  // [1, 2, 2, 4]
```

![https://miro.medium.com/max/1400/1*wSzSKduUEkEiggPMBk9Zhg.png](https://miro.medium.com/max/1400/1*wSzSKduUEkEiggPMBk9Zhg.png)

```kotlin
// Transform the content to another type value
map, mapTo, mapIndexed, mapIndexedTo, mapKeys, mapKeysTo, mapValues, mapValuesTo

// Transform the content and remove all null
// to have non-nullable result of content
mapNotNull, mapNotNullTo, mapIndexedNotNull, mapIndexedNotNullTo

// Example code
val days = listOf("Monday", "Tuesday", "Wednesday", "Thursday")
val daysLength = days.map { it.length }
println(daysLength)
```

![https://miro.medium.com/max/1400/1*2SPQFXh1RYYw7ZxuCqWIaw.png](https://miro.medium.com/max/1400/1*2SPQFXh1RYYw7ZxuCqWIaw.png)

```kotlin
// Categorize items into different list
chunked, partition, windowed

// Example code
val list = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9)
val chunked = list.chunked(4)
val windowed = list.windowed(4, 3, true)
val partition = list.partition { it % 2 == 0 }
println(chunked)  // [[1, 2, 3, 4], [5, 6, 7, 8], [9]]
println(windowed) // [[1, 2, 3, 4], [4, 5, 6, 7], [7, 8, 9]]
println(partition)// ([2, 4, 6, 8], [1, 3, 5, 7, 9])
```

![https://miro.medium.com/max/2000/1*Niwhnm6gDu4H0eePkirrig.png](https://miro.medium.com/max/2000/1*Niwhnm6gDu4H0eePkirrig.png)

```kotlin
// Join two item together (or unjoin)
zip, zipWithNext, unzip

// Example code
val list = listOf(1, 2, 3, 4, 5)
val list2 = listOf(6, 7, 8, 9)
val zip = list.zip(list2)
println(zip)

// Example code
val list = listOf(1, 2, 3, 4)
val zipWithNext = list.zipWithNext()
println(zipWithNext)

// Example code
val list = listOf(1 to 2, 3 to 4, 5 to 6)
val unzip = list.unzip()
println(unzip)
```

![https://miro.medium.com/max/1400/1*X6o1FjllyVv05uB6wVqQpA.png](https://miro.medium.com/max/1400/1*X6o1FjllyVv05uB6wVqQpA.png)
