## 반박 가능성 (Refutability): 패턴이 매칭에 실패할지의 여부

패턴에는 반박 가능한 패턴과 반박 불가능한 패턴의 두 가지 형태가 있습니다.
넘겨진 모든 가능한 값에 대해 매칭되는 패턴을 *반박 불가능한* 패턴이라고 합니다.
일례로 `let x = 5;` 구문에서 `x`는 무엇이든 매칭되므로 매칭에 실패할 수
없습니다. 일부 가능한 값에 대해 매칭에 실패할 수 있는 패턴은 *반박 가능*입니다.
`if let Some(x) = a_value` 표현식의 `Some(x)`를 예로 들 수 있는데,
`a_value` 변수의 값이 `Some`이 아니라 `None`이면 `Some(x)` 패턴이
매치되지 않기 때문입니다.

함수 매개변수, `let` 구문 및 `for` 루프에는 반박 불가능한 패턴만
허용될 수 있는데, 이는 값이 매칭되지 않으면 프로그램이 의미 있는 작업을
수행할 수 없기 때문입니다. `if let`과 `while let` 표현식은 반박 가능한
패턴과 반박 불가능한 패턴을 허용하지만, 컴파일러는 반박 불가능한 패턴에
대해 경고를 주는데, 그 이유는 정의상 이 표현식들이 잠재적인 실패를
처리할 목적으로 만들어졌기 때문입니다: 조건문의 기능은 성공 또는 실패에
따라 다르게 수행하는 능력에 있으니까요.

일반적으로는 반박 가능한 패턴과 반박 불가능한 패턴을 구분할
필요는 없습니다; 하지만 반박 가능성의 개념에 익숙해야 에러
메시지에서 이것이 나왔을때 대응이 가능합니다. 이러한 경우
코드에서 의도한 동작에 따라 패턴 혹은 패턴을 사용하는 구문을
변경할 필요가 있을 것입니다.

러스트가 반박 불가능한 패턴을 요구하는 곳에서 반박 가능한 패턴의 사용을
시도하는 경우와 그 반대의 경우 어떤 일이 발생하는지 예제를 살펴봅시다.
Listing 18-8은 `let` 구문을 보여주는데, 이 패턴에는 반박 가능한 패턴인
`Some(x)`를 특정했습니다. 예상할 수 있듯 이 코드는 컴파일되지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-08/src/main.rs:here}}
```

<span class="caption">Listing 18-8: `let`에서 반박 가능한 패턴의
사용 시도하기</span>

`some_option_value`이 `None` 값이면 `Some(x)` 패턴과 매칭되지
않으므로 이 패턴은 반박 가능합니다. 그러나 `let` 구문은 `None`
값으로 코드가 수행할 수 있는 유효한 작업이 없기 때문에 반박 불가능한
패턴만 허용할 수 있습니다. 컴파일 타임에 러스트는 반박 불가능한 패턴이
필요한 곳에 반박 가능한 패턴을 사용하려고 시도했다고 불평할 것입니다:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-08/output.txt}}
```

패턴 `Some(x)`으로 모든 유효한 값을 포함하지 않았으므로 (정확히는
포함할 수도 없으므로) 러스트는 당연히 컴파일 에러를 냅니다.

반박 불가능한 패턴이 필요한 곳에서 반박 가능한 패턴을 사용한 경우, 패턴을
사용하는 코드를 변경하여 문제를 해결할 수 있습니다: `let`을 사용하는 대신
`if let`을 사용하면 됩니다. 그러면 패턴이 매칭되지 않는 경우 코드가 중괄호
안의 코드를 그냥 건너뛰어 유효하게 계속하는 방법을 제공합니다. Listing 18-9는
Listing 18-8의 코드를 수정하는 방법을 보여줍니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-09/src/main.rs:here}}
```

<span class="caption">Listing 18-9: `let` 대신 반증 가능한 패턴과 함께
`if let`과 코드 블록 사용하기</span>

코드에 탈출구를 만들어 주었습니다! 에러 없이 반박 가능한 패턴을 사용할
수 없다는 뜻이지만, 이 코드는 완벽하게 유효합니다. Listing 18-10에
나오는 `x`처럼 `if let`에 항상 매칭되는 패턴을 사용하면, 컴파일러는
경고를 줄 것입니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-10/src/main.rs:here}}
```

<span class="caption">Listing 18-10: `if let`에 반박 불가능한 패턴 사용
시도하기</span>

러스트는 반박 불가능한 패턴에 `if let`을 사용하는 것은 이치에 맞지
않는다고 불평합니다:

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-10/output.txt}}
```

이러한 이유로, 매치 갈래는 마지막 갈래를 제외하고 반박 가능한 패턴을
사용해야 하고, 마지막 갈래는 반박 불가능한 패턴으로 나머지 모든 값과
매칭되어야 합니다. 러스트에서는 하나의 갈래만 있는 `match`에 반박
불가능한 패턴 사용이 허용되지만, 이 문법은 특별히 유용하지 않으며 더
간단한 `let` 구문으로 대체할 수 있습니다.

이제 패턴을 사용하는 위치, 그리고 반박 가능한 패턴과 반박 불가능한
패턴간의 차이점을 알았으니, 패턴을 만드는 데 사용할 수 있는 모든 문법을
다뤄봅시다.