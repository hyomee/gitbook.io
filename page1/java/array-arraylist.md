# 자바 - Array와 ArrayList 차이점

## 1. Array와 ArrayList 차이점

일반적으로 Array 또는 ArrayList는 동일한 유형(Type)의 데이터 모음을 묶어서 그룹으로 사용하고 싶을 때 사용됩니다.

배열(Array)은 선언할 때 데이터가 들어갈 공간의 개수(길이)를 지정해야 합니다. 또한 크기가 지정되면 크기를 확장 또는 축소할 수 없습니다. 이것은 크기의 제약이 있는 것으로 주의가 필요합니다

#### 1.1. Array

Array는 크기를 가지고 있는 동일한 유형의 값을 저장하는 곳으로 int, long등과 같은 기본 테이터 유형과 코드에서 작성하는 데이터 객체를 요소로 가질 수 있지만 크기가 지정되어 있어서 축소나 확장을 하지 못하는 특성을 가지고 있습니다. 다음은 배열을 만드는 예제입니다.

코드 1.1-1  : 배열 만들기

| 1. String 객체로 배열 만들기                                                                                                  |                                                                                                                                                                                            |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="https://blog.kakaocdn.net/dn/CyqxY/btsdeeTLE9U/JB6WBQbKYQOwaLavFTxuQ0/img.png" alt="" data-size="original"> | <p>변수 products는<br>1.  크기가 3인 String 객체로 생성 초기값은 null로 할당됨<br>2.  배열의 요소는 0부터 시작하므로 각각 값 설정<br>3. 배열은 참조형으로 products를 출력 시 주소 값이 출력됨<br>4. 배열의 값을 표시하기 위해 Arrays.toString() 메서드 사용해야 함</p> |
| 2. 중괄호를 사용해 배열 만들기                                                                                                    |                                                                                                                                                                                            |
| <img src="https://blog.kakaocdn.net/dn/mQhon/btsdelrLqkX/szHzLCo8iP7hWkYsgwnvn0/img.png" alt="" data-size="original"> | <p>변수 products는 중괄호를 사용해서 배열을 생성 한 예제로 중괄호 안의 요소는 모든 같은 유형(타입)으로 설정해야 합니다.<br> </p>                                                                                                        |

#### 1.2. ArrayList

ArrayList는 Array와 같이 동일한 유형의 값을 저장하는 곳이지만 크기를 언제나 변경할 수 있으나  int, long등과 같은 기본 테이터 유형은 사용할 수 없고 기본 데이터 유형에 대응되는 래퍼 클래스를 사용해야 합니다. 다음은 ArrayList를  만드는 예제입니다.

코드 1.1-2  : ArrayList 만들기

| 1. ArrayList 객체 사용                                                                                                                       |                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| <p><br></p><p><img src="https://blog.kakaocdn.net/dn/eMR8kp/btsdhBVbyhT/HS1hPNR4BE8Ny6HrOLkddK/img.png" alt="" data-size="original"></p> | Collection 객체 중 하나인 ArrayList를 사용한 것으로 ArrayList 인스턴스 화 하여 변수 products 변수에 설정한 후 메소드를 사용하여 값 설정 |
| 2. 생성과 동시에 초기값 설정                                                                                                                        |                                                                                                 |
| <p><br></p><p><img src="https://blog.kakaocdn.net/dn/bhrlvH/btsdeHOXkpF/JBQx5kPSwFzF4Jz0tLEHk0/img.png" alt="" data-size="original"></p> | <p>변수 products는 중괄호를 사용해서 배열을 생성 한 예제로 중괄호 안의 요소는 모든 같은 유형(타입)으로 설정해야 합니다.<br> </p>             |

#### 1.3. Array와와 ArrayList 차이점

ArrayList는 Array와 같이 동일한 유형의 값을 저장하는 것이지만 실제 사용 시에는 많은 차이가 있습니다. 기본적으로 Array는 크기가 고정되어 있는 자바의 기본 데이터 유형을 요소로 가질 수 있지만 ArrayList는 크기에 대해서는 유연성을 주고 있지만 자바의 기본 데이터 유형을 사용할 수 없습니다. 다음은 Array와 ArrayList차이점 입니다.

표 1.1-1  : Array와 ArrayList이점

<table data-header-hidden><thead><tr><th width="149"></th><th></th><th></th></tr></thead><tbody><tr><td>초기화와 선언</td><td><img src="https://blog.kakaocdn.net/dn/cYSpRs/btsdsSn1kBu/hyJ3LGwmD9euDp9W86JSjk/img.png" alt="" data-size="original"></td><td>Array은 고정된 크기를 가지므로 크기를 지정해야 하고 ArrayList는 동적 크기를 가지므로 크기를 지정하지 않습니다.</td></tr><tr><td>저장할 내용</td><td><img src="https://blog.kakaocdn.net/dn/cwJrJX/btsdemRKwU0/vhEmsIo40QN5iksQPw7fJ1/img.png" alt="" data-size="original"></td><td>Array는 int, long등과 같은 기본 테이터 유형과 코드에서 작성하는 데이터 객체를 저장할 수 있지만 ArrayList는  int, long와 같은 기본 테이터 유형은 가질 수 없습니다. 기본형은 사용하기 위해서는 기본형의 래퍼 클래스 (Integer, Long..)를 사용해야 합니다.</td></tr><tr><td>내용 읽기</td><td><p><br></p><p><img src="https://blog.kakaocdn.net/dn/ekVnQ0/btsdlrR4vPs/F0PGZIDUX539Biur6GEJyk/img.png" alt="" data-size="original"></p></td><td>Array는 대괄호안에 인덱스를 지정하여 사용하고 ArrayList는 get() 메소드 안에 인덱스를 지정하여 사용합니다. 인덱스는 Array나 ArrayList는 모두 0 부터 시작합니다</td></tr><tr><td>길이 구하기</td><td></td><td>Array는 length이고 ArrayList는 size 입니다</td></tr><tr><td>축소 및 확장</td><td></td><td>Array는 고정된 크기를 가지므로 요소를 축소하거나 확장할 수 없지만 ArrayList는 add, remove등 메소드를 사용하여 자유롭게 할 수 있습니다</td></tr><tr><td>요소 변경</td><td><p><br></p><p><img src="https://blog.kakaocdn.net/dn/b5Yo7E/btsdwutCLzW/kiLhUyF56I2gBxwxznJKb1/img.png" alt="" data-size="original"></p></td><td>Array는 인덱스를 사용하여 직접 변경, ArrayList는 set 메서드 사용합니다.</td></tr><tr><td>요소 삭제</td><td><img src="https://blog.kakaocdn.net/dn/WaLyM/btsdpyDlK90/rzQYxKno3jOf9cXREVzkE1/img.png" alt="" data-size="original"></td><td>Array는 고정 길이 이므로 삭제를 할 수 없고 ArrayList는 remove 메서드를 사용하여 삭제할 수 있습니다. 삭제 후 크기는 변하게 됩니다.</td></tr><tr><td>출력</td><td><img src="https://blog.kakaocdn.net/dn/bOJQRY/btsdemqFuVo/U7kEVJc8IifNYn4v0rJe2K/img.png" alt="" data-size="original"></td><td>Array는 배열일 저장된 주소를 출력하지만 ArrayList는 배열의 값을 출력합니다.</td></tr></tbody></table>

<figure><img src="https://blog.kakaocdn.net/dn/bFFK0C/btsdem5migk/HoZKFhXiC9mOMQBajnNS0k/img.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://blog.kakaocdn.net/dn/lMA77/btsdwu1uwKF/6wEO8hclIRUdM33QzfKKdK/img.png" alt=""><figcaption></figcaption></figure>

ArrayList는 Array의 래퍼 클래스로 Array를 사용할 때보다는 오버헤드가 발생 하나 Collection 객체의 다양한 기능을 사용해서 유연성을 확보할 수 있으며 스트림 객체와 결합하여 다양한 기능을 쉽게 작성할 수 있습니다.
