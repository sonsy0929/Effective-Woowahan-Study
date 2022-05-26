# iterator보다 const_iterator를 선호하라

## const
const의 큰 매력은 변경을 불가능하게 하는 **의미적인 제약**을 코드 수준에서 사용하여 컴파일러에서 이 제약을 지켜줄 수 있다는 점이다. 

## iterator
iterator는 클래스 템플릿 타입의 객체로 원소들의 범위를 참조하여 container에 저장되어 있는 원소들을 **공통적인 방법으로** 하나씩 접근할 수 있게 해준다. 각 타입에 `::iterator` 또는 `::const_iterator`를 뒤에 붙여주면 사용이 가능하다.

```c++
// vector<int> container의 iterator
std::vector<int>::iterator it;
// vetor<int> container의 const_iterator
std::vector<int>::const_iterator cit;
```
iterator는 포인터를 본뜬 것이기 때문에 기본적인 동작 원리가 `T*` 포인터와 흡사하다. iterator를 const로 선언한다는 것은 포인터를 상수로 선언하는 것(T* const pointer)과 같다. iterator는 변경하지 못하며 그 대상의 값은 변경이 가능하다.

```c++
const std::vector<int>::iterator it = v.begin();
*it = 10; // 대상의 값은 변경이 가능하다.
++it; // iterator it을 변경하는 것은 불가능하다.
```

### Bidirectional Iterator와 Random Access Iterator

container마다 지원하는 반복자의 종류가 다르며, 사용할 수 있는 연산 또한 다르다. 

- Bidirectional Iterator
  - Read, Write가 가능하다.
  - 산술연산 `++`, `--` 가능하다.
  - 비교연산 `==`, `!=` 가능하다.
  - 대표적인 container로 `std::list`, `std::set`, `std::map`이 해당 반복자를 지원한다.

- Random Access Iterator
  - Read, Write가 가능하다.
  - 산술연산 `++`, `--`, `+`, `-`, `+=`, `-=` 가능하다.
  - 비교연산 `==`, `!=`, `>`, `<`, `>=`, `<=` 가능하다.
  - `[]` 연산자 사용이 가능하다.
  - 대표적인 container로 `std::vector`, `std::deque`가 있다.
  
## const_iterator를 사용해야 한다면 사용하자
const_iterator는 포인터로 비교해보자면 `const T*`와 같다. 즉, iterator가 가리키는 대상을 수정하지 못하게 막을 수 있어 immutable을 보장해준다. 만약 iterator가 가리키는 대상을 수정할 필요가 없고, 수정을 원치않는다면 const_iterator를 사용하는 것이 더 안정적인 코드가 될 수 있다.

### How to use const_iteartor since c++11
container의 const의 여부와 상관없이 `.cbegin()`과 `.cend()`의 container의 멤버함수를 이용하여 conster_iterator를 사용할 수 있다.

### iterator vs. const_iterator
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

### 비멤버함수 `cbegin()`, `cend()`
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

## Range based for loop
c++98에서 for loop를 사용할 때는 아래의 코드와 같이 초기값, 조건식, 변화값을 주어야 했다.

```c++
std::vector<std::string> kakaoFriends{"라이언", "어피치", "춘식", "무지", "죠르디"};
for (int i = 0; i < kakaoFriends.size(); i++) {
  std::cout << kakaoFriends[i] << " ";
}
```
c++11에서는 range based for loop라는 새로운 유형의 반복문을 제공하여 더 간단하고 안전하게 STL의 Container의 모든 요소를 반복하는 방법을 제공한다.

### How to use range based for loop

```c++
for (range-declaration : range-expression) {
  loop-statement
}
```
- `range-expression` : `array`, `vector`와 같은 순회가 가능한 container를 명시해주면 된다.
- `range-declaration` : container가 가지고 있는 elment의 type과 변수 이름을 명시해주면 된다.

```c++
std::vector<std::string> kakaoFriends{"라이언", "어피치", "춘식", "무지", "죠르디"};
for (std::string kakaoFriend : kakaoFriends) {
  std::cout << kakaoFriend << " ";
}
for (auto kakaoFriend : kakaoFriends) {
  std::cout << kakaoFriend << " ";
}
```

### Copy and Performance issues

매번 loop를 반복할 때마다 container의 element들이 `range-declaration`에 선언된 변수에 복사가 된다. 따라서 위와 같은 range based for문 내부에서는 container의 element들을 변경할 수 없다. 또한, 복사 비용이 큰 object인 경우에는 성능 상에 이슈가 생길 수 있다.

참고) https://codeforces.com/contest/1326/problem/A

이러한 단점을 보완하기 위해서 c++의 reference를 이용하면 된다.

```c++
std::vector<std::string> kakaoFriends{"라이언", "어피치", "춘식", "무지", "죠르디"};
for (std::string& kakaoFriend : kakaoFriends) {
  /*
  1. 복사 비용이 발생하지 않는다.
  2. container의 element를 변경할 수 있다.
  */
  std::cout << kakaoFriend << " ";
}
for (const auto& kakaoFriend : kakaoFriends) {
  /*
  1. 복사 비용이 발생하지 않는다.
  2. container의 element를 변경하지 않는 것을 보장한다.
  */
  std::cout << kakaoFriend << " ";
}
```