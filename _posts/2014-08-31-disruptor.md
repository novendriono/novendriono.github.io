---
layout: post
title: Disruptor
categories: java
---

## What is Disruptor ?

> Disruptor is a general-purpose mechanism for solving a difficult problem in concurrent programming.

So, I think disruptor is a tool for building high-performance Java application.
It's an immensely useful library. As a preface, you may read [Disruptor project on GitHub](http://lmax-exchange.github.io/disruptor/) or
[Martin Fowler's review](http://martinfowler.com/articles/lmax.html).

## Ring Buffer

<img src="https://lh5.googleusercontent.com/-sDbRQ1Z_YBE/VAKbWvyFjoI/AAAAAAAAApY/-yAS4Wqg4bE/w1044-h369-no/IMG_20140831_104706~2.jpg"/>
Disruptor use a ring buffer data structure to achieve performance. Ring Buffer will contains event from our application. A publisher will
write an event into the ring buffer.

Let's create an event

{% highlight java %}
// ApprovedEvent.java
public class ApprovedEvent {
  public String remarks;

  public ApprovedEvent(String remarks) {
    this.remarks = remarks;
  }
}
{% endhighlight %}

This event have a String property called remarks and will populated into ring buffer using an EventFactory.
Okay, now we need an event factory.

{% highlight java %}
// ApprovedEvent.java
import com.lmax.disruptor.EventFactory;

public class ApprovedEvent {
  ...

  public static final EventFactory<ApprovedEvent> FACTORY =
    new EventFactory<ApprovedEvent>() {
      @Override
      public ApprovedEvent newInstance() {
        ApprovedEvent event = new ApprovedEvent("");
        return event;
      }
    }
}
{% endhighlight %}

Now create an event handler to handle ApprovedEvent.
{% highlight java %}

import com.lmax.disruptor.EventHandler;

public class ApprovedEventHandler implements
  EventHandler<ApprovedEvent>{

	@Override
	public void onEvent(ApprovedEvent event, long sequence, boolean endOfBatch)
			throws Exception {
		System.out.println("Event remarks: " + event.remarks);
	}

}
{% endhighlight %}

Now, we will create a Main class to wire disruptor to EventHandler.

{% highlight java %}
import com.lmax.disruptor.RingBuffer;
import com.lmax.disruptor.dsl.Disruptor;

public class Main {

  @SuppressWarnings("unchecked")
  public static void main(String[] args) {

    int bufferSize = 1024;

    Executor executor = Executors.newCachedThreadPool();

    Disruptor<ApprovedEvent> disruptor = new Disruptor<ApprovedEvent>(
        ApprovedEvent.FACTORY, bufferSize, executor);

    disruptor.handleEventsWith(new ApprovedEventHandler());

    RingBuffer<ApprovedEvent> ringBuffer = disruptor.start();

    for (int i = 0, size = ringBuffer.getBufferSize(); i < size ; i++) {
      long sequence = ringBuffer.next();
      try {			
        System.out.println(sequence);
        ApprovedEvent event = ringBuffer.get(sequence);
        event.remarks = "Test " + i;
      } finally {
        ringBuffer.publish(sequence);
      }
    }
  }
}

{% endhighlight %}
