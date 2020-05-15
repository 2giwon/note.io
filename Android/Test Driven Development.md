## TDD 란 무엇인가?

TDD 란 개발자가 코드를 작성하기 전에 코드 조각에 대한 테스트를 작성하는 소프트웨어 방법론.

1. 예상되는 동작을 다루는 많은 테스트가 있음
2. 통과가 되는 코드를 작성할 때까지 반복

### TDD 이점

1. 어떤 기능에 대한 코드의 제약 조건을 모두 고려했기 때문에 구현이 빠름
2. 코드의 작동방식을 이미 정의 했기때문에 작성한 코드에 대한 파악이 쉬움
3. 오류에 대한 파악이 쉬움

### TDD 단점

viewModel 을 만들기 전에 단위 테스트 작성부터 고려해야됨.

![https://androidessence.com/assets/tdd/viewmodel_not_found.png](https://androidessence.com/assets/tdd/viewmodel_not_found.png)

실제 클래스가 존재하지 않기 때문에 구현을 먼저 해야하나

테스트 코드 작성을 먼저 하게 되어 테스트에 의한 구현이 발생.

## 효과적인 TDD 구현 방법

### 예제 구성 요소

Pokemon 모델의 일부 정보를 조건부로 보여주는 PokemonDetailViewModel을 작성

### 최소한의 구현으로 제작

컴파일 단계에서 실제값을 사용해서 작성할 수 있는 Kotlin TODO 메소드를 활용할 수 있지만

런타임에서 `NotImplementedError`를 발생

```kotlin
class PokemonDetailViewModel {

    fun setPokemon(pokemon: Pokemon) {
        TODO("This method is not yet implemented")
    }

    val pokemonName: String
        get() = TODO("This field is not yet implemented.")

    val firstType: String
        get() = TODO("This field is not yet implemented.")

    val secondType: String?
        get() = TODO("This field is not yet implemented.")

    val showSecondType: Boolean
        get() = TODO("This field is not yet implemented.")
}
```

### 테스트 계획 개발

ViewModel이 가져야할 제약 조건을 고려

1. 포켓몬이 공급되지 않을땐?
2. 유형이 없는 포켓몬을 공급한다면?
3. 포켓몬에 하나의 유형만 있다면?
4. 포켓몬에 두가지 유형이 제공되면?
5. 포켓몬에 두가지 이상의 유형이 제공되면?

위의 제약사항에 대한 접근법

1. 포켓몬이 제공되지 않으면 빈 문자열을 제공
2. 유효하지 않은 유형의 포켓몬은 예외 발생
3. 유효한 포켓몬은 이름과 유형정보가 노출

### 테스트 작성

```kotlin
class PokemonDetailViewModelTest {

    @Test
    fun uninitializedPokemonReturnsDefaultValues() {
        val viewModel = PokemonDetailViewModel()

        assertEquals("", viewModel.pokemonName)
        assertEquals("", viewModel.firstType)
        assertNull(viewModel.secondType)
        assertFalse(viewModel.showSecondType)
    }

    @Test
    fun pokemonWithOneTypeExposesOnlyThatType() {
        val viewModel = PokemonDetailViewModel()

        val testPokemon = Pokemon(
            name = "Squirtle",
            types = listOf("Water")
        )

        viewModel.setPokemon(testPokemon)
        assertEquals("Squirtle", viewModel.pokemonName)
        assertEquals("Water", viewModel.firstType)
        assertNull(viewModel.secondType)
        assertFalse(viewModel.showSecondType)
    }
}
```

위의 테스트 결과는 모두 실패

`NotImplementedError`:

![https://androidessence.com/assets/tdd/not_implemented_failure.png](https://androidessence.com/assets/tdd/not_implemented_failure.png)

### 테스트 통과 하기

- 테스트 통과를 위해 필수 필드를 구현
- 성공적인 루트부터 시작

```kotlin
class PokemonDetailViewModel {
    
    private var pokemon: Pokemon? = null

    fun setPokemon(pokemon: Pokemon) {
        this.pokemon = pokemon
    }

    val pokemonName: String
        get() = this.pokemon?.name.orEmpty()

    val firstType: String
        get() = this.pokemon?.types?.first().orEmpty()

    val secondType: String?
        get() = this.pokemon?.types?.getOrNull(1)

    val showSecondType: Boolean
        get() = this.secondType != null
}
```

입력이 올바르지 않은 경우 실패

![https://androidessence.com/assets/tdd/invalid_input_failure.png](https://androidessence.com/assets/tdd/invalid_input_failure.png)

setPokemon 메소드에 관련 예외 발생하도록 구현

```kotlin
class PokemonDetailViewModel {

    fun setPokemon(pokemon: Pokemon) {
        val typeCount = pokemon.types.size
        if (typeCount <= 0 || typeCount > 2) {
            throw IllegalArgumentException("Pokemon has an invalid number of types.")
        }
        
        this.pokemon = pokemon
    }
}
```

![https://androidessence.com/assets/tdd/tests_passing.png](https://androidessence.com/assets/tdd/tests_passing.png)

[https://gist.github.com/AdamMc331/c815f3ae7579409b01b0fbfd5c9984aa](https://gist.github.com/AdamMc331/c815f3ae7579409b01b0fbfd5c9984aa)

## 요약 및 팁

- TDD는 우리가 작성한 코드에 대해 신중하게 생각해야하기 때문에 중요합니다.
- 이 계획 단계 덕분에 코드를보다 빠르고 확실하게 작성할 수 있습니다.
Kotlin의 TODO 방법을 활용하여 코드 완성 및 기타 IDE 도구를 사용하여 테스트를 작성할 수있는 최소 구현을 만들 수 있습니다.
- 구축중인 구성 요소에 필요한 모든 기능을 포괄하는 테스트를 작성하십시오.
- 모든 TODO 호출을 실제 구현으로 대체하여 해당 테스트를 통과하십시오.

큰 규모에 전체 테스트 방식을 작성할 필요는 없습니다.

소규모 유틸 클래스 부터 시작하세요.

프로세스에 익숙해지면 더 큰규모로 작성해도 됩니다.
