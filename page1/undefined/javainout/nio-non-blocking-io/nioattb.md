# Selector

스레드는 운영 채제애 비용을 먾이 들이는 작업으로 하나의 스레드에서 여러 채널을 관리하며 비용 소모가 적게 들게 되니다. 이런 경우 Selector에 여러개의 스레드를 묶어서 하나의 스레드로 처리하게 됩니다.

<figure><img src="../../../../.gitbook/assets/image.png" alt="" width="306"><figcaption></figcaption></figure>

작업 순서는 다음과 같습니다.

1. Selector 생성
2. 선택 가능한 채널 등록



## 1. selector 생성&#x20;

```java
Selector selector = Selector.open();
```

## 2. 선택 가능한 채널 등록

선택기에는 채널을 사용하기 위해서는 register 메서드로 채널을 등록 해야 하는데 우선 비동기 모드로 설정 되어 있어야 됩니다. 이 말의 의미는 FileChannel을 사용 할 수 없다는 이야기 입니다. 왜냐 하면 FileChannel는 비동기 모드로 동작을 하지 않기 때문입니다.&#x20;

다음은  channel에 selector

register 의 시스니처 입니다.

```java
public final SelectionKey register(Selector sel, int ops, Object att)
        throws ClosedChannelException { ... } 
```

```java
channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

register 의 두번째 파라메터는 "interest set"으로 selector를 통해 채널에서 수신하려는 이벤트를 의미하며 다음과 같이 네가지의 이벤트가 있습니다.

* Connect : SelectionKey.OP\_CONNECT, 클라이언트가 서버에 연결을 시도&#x20;
* Accept : SelectionKey.OP\_ACCEPT, 서버가 클라이언트의 연결을 수락
* Read : SelectionKey.OP\_READ, 서버가 채널에서 읽을 준비가 되었을 때
* Write : SelectionKey.OP\_WRITE, 서버가 채널에 쓸 준비가 되었을 때

## 3.  _SelectionKey_ 개체의 속성을 검사하는 방법

&#x20;**SelectionKey 객체는** 채널에 등록된 데이터를 가지고 있는 객체로 다음과 같은 속성이 있습니다.

* Interest Set
* Ready Set
* Channel
* Selector
* Attaching Objects

### 3-1. Interest Set

_selectionKey_의 _interestOps_ 메서드에 의해서 반환되는 값으로 SelectionKey의 상수값으로 두 값을 AND 연산을 통해서 감지되고 있는지 여부를 알 수 있습니다.

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

### 3-2. Ready Set

Channel이 준비된 이벤트 세트으로 s_electionKey_의 _readyOps_ 메서드로 얻을 수 있습니다.

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWriteable();
```

### 3-3. Channel

selectionKey 객체에서  수신되는 channel

```java
Channel channel = selectionKey.channel();
```

### 3-4. selector

selectionKey 객체에서  정의된 selector

```java
Selector selector = selectionKey.selector();
```

## 4. 추가 객체 첨부

주어진 채널 또는 채널에 추가 정보를 첨부합니다. 예를 들면 채널과 함께 사용 중인 또는 더 많은 집계를 포함하는 데이터객체를 추가 합니다.

```java
key.attach(Object);
Object object = key.attachment();

// selector 생성시 
SelectionKey key = channel.register(
  selector, SelectionKey.OP_ACCEPT, object);
```

## 5. 선택기를 통한 채널 선택

selector에는 하나 또는 여러개의 채널을 연결할 수 있는데 selector의 _select_ 메소드를 사용하여 선택 합니다. select 메서드는 하나 이상의 채널이 작업을 수행할 준비( (Interest Set : connect, accept, read or write))가 될 때까지 차단됩니다. 반환되는 정수는 채널이 조작을 위해 준비된 키의 수를 나타내는 것으로 다음과 같은 방법으로 얻을 수 있습니다.

* int select() : 등록된 이벤트가 하나 이상의 채널이 준비 될 때까지 차단&#x20;
* int select(long timeout) : 최대 허용 시간 까지 차단단
* int selectNow() : 차단 하지 않습니다.

### 5-1. selectedKeys()

준비된 채널의 SelectionKey Set를 통해서 얻을 수 있습니다.&#x20;

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {
    
    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}
```



## 6. 전체 예제

{% code title="서버 " lineNumbers="true" %}
```java
package org.example;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class JavaNioSocketSever {

    public static void main(String[] args) {
        System.out.println("Hello world!");

        try {
            JavaNioSocketSever.selectorServerSocketChannel();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private static void selectorServerSocketChannel() throws IOException {
        // Select 오픈 ( 생성 )
        Selector selector = Selector.open();

        System.out.println("Selector open: " + selector.isOpen());


        // 서버 소켓 채널 오픈 ( 생성 )
        ServerSocketChannel serverSocket = ServerSocketChannel.open();
        InetSocketAddress hostAddress = new InetSocketAddress("localhost", 5454);
        serverSocket.bind(hostAddress);

        // 비동기 설정
        serverSocket.configureBlocking(false);
        
        // 해당 채널을 연결 수락을 식별하는 작업 SET
        int ops = serverSocket.validOps();

        // 서버 소켓 채널에 selector 지정 
        SelectionKey selectKy = serverSocket.register(selector, ops, null);
        System.out.println("등록 이후 :  " + selectKy);
        for (;;) {

            System.out.println("Waiting for select...");
            int noOfKeys = selector.select();

            System.out.println("Number of selected keys: " + noOfKeys);

            Set selectedKeys = selector.selectedKeys();
            Iterator iter = selectedKeys.iterator();

            while (iter.hasNext()) {

                SelectionKey ky = (SelectionKey) iter.next();

                if (ky.isAcceptable()) {

                    // Accept the new client connection
                    SocketChannel client = serverSocket.accept();
                    client.configureBlocking(false);

                    // Add the new connection to the selector
                    client.register(selector, SelectionKey.OP_READ);

                    System.out.println("Accepted new connection from client: " + client);
                }
                else if (ky.isReadable()) {

                    // Read the data from client

                    SocketChannel client = (SocketChannel) ky.channel();
                    ByteBuffer buffer = ByteBuffer.allocate(256);
                    client.read(buffer);
                    String output = new String(buffer.array()).trim();

                    System.out.println("Message read from client: " + output);

                    if (output.equals("Bye.")) {

                        client.close();
                        System.out.println("Client messages are complete; close.");
                    }

                } // end if (ky...)

                iter.remove();

            } // end while loop

        } // end for loop
    }

}

```
{% endcode %}

{% code title="클라이언트" lineNumbers="true" %}
```java
package org.example;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class JavaNioClient {
    public static void main(String[] args) {
        try {
            JavaNioClient.socketClient();
        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }


    public static void socketClient() throws IOException, InterruptedException {
        InetSocketAddress hostAddress = new InetSocketAddress("localhost", 5454);
        SocketChannel client = SocketChannel.open(hostAddress);

        System.out.println("Client sending messages to server..." + client.isConnected());
        Thread.sleep(3000);
        
        // 메세지 발송 
        String [] messages = new String [] {"Time goes fast.", "What now?", "Bye."};

        for (int i = 0; i < messages.length; i++) {

            byte [] message = new String(messages [i]).getBytes();
            ByteBuffer buffer = ByteBuffer.wrap(message);
            client.write(buffer);

            System.out.println(messages [i]);
            buffer.clear();
            Thread.sleep(3000);
        }

        client.close();
    }
}

```
{% endcode %}
