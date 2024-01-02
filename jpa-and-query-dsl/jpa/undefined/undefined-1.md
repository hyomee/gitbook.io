# 양방향 매핑

객체 관계를 양쪽 방향으로 연돤 관계를 설정하는 것으로 꼭 필요할 때만 사용해여 한다.

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption><p>양방향 관계</p></figcaption></figure>

객체는 양방향이 없으므로 사용자 관점, 조직 관점을 고려해서 주인 객체를 지정해야 한다. 즉 객체 관계를 만들때 **주인이 되는 객체를 우선 선정**하여 코드 작성을 해야 한다.

* <mark style="color:purple;">주인 객체만 외래키 관리(등록, 수정, 삭제)를 할 수 있다.</mark>
* <mark style="color:purple;">주인이 아닌 객체는 읽기만 가능</mark>
* <mark style="color:purple;">주인 객체 : @JoinColumn 사용</mark>
* <mark style="color:purple;">주인이 아닌 객체 : mappedBy 속성 사용</mark>
* <mark style="color:purple;">주인 객체는 일반적으로 DB에서 FK가 있는 객체임</mark>

<pre class="language-java"><code class="lang-java">@Entity
public class 사용자 {
    private String 사용자ID;
    private String 사용자이름;
    @ManyToOne
    @JoinColumn(name=“조직ID”)
<strong>    private 조직 조직;
</strong>}

@Entity
public class 조직 {
    @Id
    @Column(name=“조직ID”)
    private String 조직ID;
    private String 조직이름;
    
    @OneToMany(mappedBy = “조직”)
    private List&#x3C;사용자> 사용자s = new ArrayList&#x3C;사용자>{}
}
</code></pre>

