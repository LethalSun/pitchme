# std::string

---
#### std::string이란?
* C++ 표준 라이브러리에서 문자열을 다루는 객체
* 포함 헤더파일
```cpp
#include <string>
```
---
#### string 객체의 생성
* 객체이므로 생성자를 이용해서 생성한다.
```{.cpp}
string publisher("ButterworthHeinemann");//문자열 포인터를 이용해서 생성
string bookName("Classical Field Theory");
string copiedBookName(bookName);// 복사 생성자
string author;//빈 스트링 객체
string bookStore[3];// 스트링 객체의 배열
```
---
#### string 객체의 입,출력
* 스트림객체인 cin,cout등을 이용해서 출력가능
* 즉 다른 스트림 객체를 이용해서도 가능 파일쓰기라던지
* 만약에 printf같은 c의 함수를 사용한다면?
* printf가 출력하는 것은 문자배열의 포인터를 이용하므로 포인터를 얻어오는 c_str()메소드를 사용하면 된다.
```cpp
printf("%s", bookName.c_str());
```
---
#### 문자열 삽입 삭제
*  특정위치 바로 뒤에 문자열을 삽입한다.
```cpp
bookName.insert(1,"3rd addition");//위치, 문자열
```
* 문자열 삭제 특정 위치부터 몇게
```cpp
bookName.erase(1, 13);//시작위치 갯수
```
---
#### 연산자를 이용한 문자열 추가
* +를 이용해서 두문자열을 연결한다.
```c++
string publisher("ButterworthHeinemann");//문자열 포인터를 이용해서 생성
string bookName("Classical Field Theory");
bookName += publisher;
```
---
#### 길이
* 두 메소드는 같은 역할을 한다.
```c++
string bookName = "hello world!";
bookName.length();
bookName.size();
```
---
#### 메모리 확인
* 첫번째는 재할당하지 않고 넣을수 있는 최대 용량
* 두번째는 메모리를 다 사용했을때 넣을수 있는 최대 용량
```c++
string bookName = "hello world!";
bookName.capacity();
bookName.max_size();
```
---
#### 특정위치의 문자 확인
* 특정위치의 문자를 반환한다.
```c++
string bookName = "hello world!";
bookName.at(0); // 'h'
bookName.at(1); // 'e'
```
---
#### 특정 문자의 검색
* 특정 문자열이 발견된 첫번째 위치를 반환한다.
* 없다면 string::npos 를 반환
```c++
string base = "hello world!";
base.find("world!");
```
---
#### 문자열 비교
* compare 메소드 사용
```c++
string a = "I am string one!";
string b = "string ";
if (a.compare(b) == 0)
{
  // 두 문자열이 같을 때
}
else if (a.compare(b) < 0)
{
  // a가 b보다 사전순으로 앞일 때
}
else if (a.compare(b) > 0)
{
  // a가 b보다 사전순으로 뒤일 때
}
```
---
#### 원하는 문자열을 특정 문자열로 대체
* replace 메소드 사용
```c++
std::string replaceString(std::string subject, const std::string &search, const std::string &replace)
 {
    size_t pos = 0;
    while ((pos = subject.find(search, pos)) != std::string::npos)
    {
      subject.replace(pos, search.length(), replace);
       pos += replace.length();
    }
    return subject;
}
```
---
#### 타입 변환
* C++11부터 생김
```c++
// int ---> string
string s;
int i = 10;
s = std::to_string(i);
// string ---> int
string s = "123";
int i;
i = std::stoi(s);
```
---
#### 부분 문자열 추출

* substr 메소드 사용
```c++
std::string str="We think in generalities, but we live in details.";
std::string str2 = str.substr (3,5);// "think"
std::size_t pos = str.find("live");// position of "live" in str
std::string str3 = str.substr (pos);// get from "live" to the end
```
