---
layout: post
title: Disruptor
categories: java
---

## What is Disruptor ?

> Disruptor is a general-purpose mechanism for solving a difficult problem in concurrent programming.

So, I think disruptor is a tool for builiding high-performance Java application.
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

