# const
const의 큰 매력은 변경을 불가능하게 하는 **의미적인 제약**을 코드 수준에서 사용하여 컴파일러에서 이 제약을 지켜줄 수 있다는 점이다. 

# iterator
iterator는 포인터를 본뜬 것이기 때문에 기본적인 동작 원리가 `T*` 포인터와 흡사하다. iterator를 const로 선언한다는 것은 포인터를 상수로 선언하는 것(T* const pointer)과 같다. iterator는 변경하지 못하며 그 대상의 값은 변경이 가능하다.

```c++
const std::vector<int>::iterator it = v.begin();
*it = 10; // 대상의 값은 변경이 가능하다.
++it; // iterator it을 변경하는 것은 불가능하다.
```

# const_iterator를 사용해야 한다면 사용하자
const_iterator는 포인터로 비교해보자면 `const T*`와 같다. 즉, iterator가 가리키는 대상을 수정하지 못하게 막을 수 있어 immutable을 보장해준다. 만약 iterator가 가리키는 대상을 수정할 필요가 없고, 수정을 원치않는다면 const_iterator를 사용하는 것이 더 안정적인 코드가 될 수 있다.

## How to use const_iteartor since c++11
container의 const의 여부와 상관없이 `.cbegin()`과 `.cend()`의 container의 멤버함수를 이용하여 conster_iterator를 사용할 수 있다.

## iterator vs. const_iterator
```c++
std::vector<std::string> kakaoFriends{"라이언", "어피치", "춘식", "무지", "죠르디"};

using iter = std::vector<std::string>::iterator;
using citer = std::vector<std::string>::const_iterator;

for (iter it = kakaoFriends.begin(); it != kakaoFriends.end(); it++) {
	std::cout << *it << "\n";
	*it = "춘식";
}

for (citer cit = kakaoFriends.cbegin(); cit != kakaoFriends.cend(); cit++) {
	std::cout << *cit << "\n";
	*cit = "라이언" // 컴파일 에러 발생
}
```

## 비멤버함수 `cbegin()`, `cend()`
template 등에서 container 및 built-in type의 배열 혹은 third-party-library의 container 타입에 대해 최대한 일반적으로 동작하는 코드를 작성하고 싶을 수 있다. 이 경우에 멤버함수 `.begin()`, `.end()`를 사용하는 것보다 비멤버함수 `begin()`, `end()`를 사용하는 것이 좋다. 사용하는 container의 멤버함수로 항상 `.begin()`, `.end()`를 포함하고 있다는 것이 보장되지 않고, 일반성을 극대화한 코드에서는 특정 멤버 함수의 존재를 요구하기 보다는 그 멤버 함수에 상응되는 비멤버 함수를 사용한다.

```c++
template<typename C, typename V>
void findAndInsert(C& container,
                    const V& targetVal,
                    const V& insertVal)
{
    using std::cbegin;
    using std::cend;

    auto it = std::find(cbegin(container), cend(container), targetVal);

    container.insert(it, insertVal);
}
```

위 코드는 c++14에만 동작하는데, 그 이유는 c++11에서는 멤버함수가 아닌 `cbegin()`, `cend()`, `rbegin()`, `rend()`, `crbegin()`, `crend()`가 표준에 추가되지 않았다. c++11에서 위와 같은 방식으로 동작하는 코드를 구현하고 싶다면 아래 코드를 추가하면 된다.

```c++
template <class C>
auto cbegin(const C& container) -> decltype(std::begin(container))
{
    return std::begin(container);
}
```

컨테이너 타입이 C일 때 const C& 타입 컨테이너의 iterator를 반환하였으므로, 그 결과값은 const_iterator가 된다.