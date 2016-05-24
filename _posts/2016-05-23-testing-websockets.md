---
layout: post
title: Testing Websockets with Spring
description: "Testing Websockets with Stomp Client"
tags: [test, websocket, spring, stomp]
image:
  feature: abstract-1.jpg
  credit: dargadgetz
---

In the last week, I had a problem after including some new libraries in the project that I was working on.  There was no exception occurring, but the Web Sockets stopped to work.

Unfortunately, we just realized the problem after some days.  This gave me some hours of work (and some gray hairs), trying to figure out manually what commit breaks the web socket.   Actually, it was my mistake,  I didn't write any integration test for the Web Socket.

After discovering the problem. (an integration problem with the lib spring-cloud-sleuth),
I decided to write a test for it.  I don't want to have the same problem again!

I had never written tests for Web Sockets before, and I didn’t find any clear and good example for it.  Then, I decided to share my solution here, maybe it will help someone that needs it.

This is the websocket configuration using Spring:

{% highlight java %}
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry
                .addEndpoint("/websocket")
                .setAllowedOrigins("*")
                .withSockJS();
    }
}s
{% endhighlight %}

And here I put the test using Stomp Client.  Basically, the test connects to the Web Socket,
then subscribes to the topic and sends a message.  I used the class BlockingQueue, in order to wait for the asynchronous response.

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = App.class)
@WebIntegrationTest
public class WebSocketTest {

    static final String WEBSOCKET_URI = "ws://localhost:8080/websocket";
    static final String WEBSOCKET_TOPIC = "/topic";

    BlockingQueue<String> blockingQueue;
    WebSocketStompClient stompClient;

    @Before
    public void setup() {
        blockingQueue = new LinkedBlockingDeque<>();
        stompClient = new WebSocketStompClient(new SockJsClient(
                asList(new WebSocketTransport(new StandardWebSocketClient()))));
    }

    @Test
    public void shouldReceiveAMessageFromTheServer() throws Exception {
        StompSession session = stompClient
                .connect(WEBSOCKET_URI, new StompSessionHandlerAdapter() {})
                .get(1, SECONDS);
        session.subscribe(WEBSOCKET_TOPIC, new DefaultStompFrameHandler());

        String message = "MESSAGE TEST";
        session.send(WEBSOCKET_TOPIC, message.getBytes());

        Assert.assertEquals(message, blockingQueue.poll(1, SECONDS));
    }

    class DefaultStompFrameHandler implements StompFrameHandler {
        @Override
        public Type getPayloadType(StompHeaders stompHeaders) {
            return byte[].class;
        }

        @Override
        public void handleFrame(StompHeaders stompHeaders, Object o) {
            blockingQueue.offer(new String((byte[]) o));
        }
    }
}
{% endhighlight %}


## Lesson learned

Always write integration tests.  It can prevent bugs when you upgrade libraries, or when you include new ones. It also can save some hours and headaches in the future.

If you want, you can clone the complete project from my github:

[https://github.com/rafaelhz/testing-spring-websocket.git](https://github.com/rafaelhz/testing-spring-websocket.git) 

If you have some doubts, or have some suggestion to improve this code, please let me know.

That’s it for today friends, until next time!