# Exchange-Topic

**라우팅 키가 일치하는 Queue로 메시지를 전달하는 방식으로 라우팅 키는 점(.)으로 구분된 단어를 조합해서 정의하며, 와일드 카드(\*)와(#)을 사용하여 패턴을 지정할 수 있습니다**

* 라우팅 키는 점으로 구분된 0개 이상의 단어로 구성되어야 합니다.
* 라우팅 패턴은 , **`.`** 및 **`#`**만 허용되는 정규식과 같습니다.
  * 기호 별(**\***)은 정확히 한 단어가 허용됨을 의미
  * 기호 해시(**#**)는 허용되는 단어 수가 0개 이상임을 의미
  * 기호 점(**.**)은 단어 구분 기호를 의미(여러 키 용어는 점 구분 기호로 구분)
  *   라우팅키 패턴:\


      | 턴                                        | 유효한 라우팅 키                                                | 잘못된 라우팅 키                                                      |
      | ---------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------- |
      | email**.\*** – 첫 번째 단어 다음에 한 단어가 옵니다.    | <p>email.naver,<br>email.iabacus,<br>email.daum</p>      | <p>email, <br>email.naver.anything, email.comnpany.iabacus</p> |
      | **#.sms.\*** – 0개 이상의 단어, 그 뒤에 정확히 한 단어. | <p>sms.featurephone<br>sms.sms.sms,</p><p>sm.sms<br></p> | <p>sms,<br>company.sms,<br>anything.sms.anything.xyz</p>       |
      | **#.sns**– 0개 이상의 단어 뒤에 단어가 옵니다.         | <p>kakao.sns,</p><p>sns.sns,<br>sns</p>                  | <p>sns.company,<br>anything.sns.anything</p>                   |

      \

*   Topic  Exchange 흐\


    <figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

    * 큐는 라우팅 키 패턴을 사용하여 Exchange애 바인딩 됩니다.
    * Publisher가 메시지를 게시 할 때 라우팅 key를 포함해서 Exchange에 보냅니다.
    * Exchange는  라우팅 키패턴과 라우팅 key가 일치하는 큐로 메시지를 전달합니다.



