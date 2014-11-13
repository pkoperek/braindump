---
layout: post
title: Use Text or create new Text - this is the question
comments: true
---

One of the things, which gets repeated in all MapReduce tutorials, is that if you use `TextOutputFormat` and spit out `Text` objects, you need to avoid creating them. Instead of that, you can just create a single instance, store it in a field and use `set` method. Unfortunately `set` for `byte` array doesn't behave in the most intuitive way... The input array is copied to an internal buffer - unless the new data chunk is bigger, its using the same piece of memory over and over again. Guess for performance reasons, the unused part of array is left dirty. If you want to get the actual data from the internal buffer - you need to use `copyBytes`. Unfortunately - it seems that not all parts of Hadoop code know that. For example, the `TextOutputFormat` class uses this method:

```java
private void More ...writeObject(Object o) throws IOException {
    if (o instanceof Text) {
        Text to = (Text) o;
        out.write(to.getBytes(), 0, to.getLength());
    } else {
        out.write(o.toString().getBytes(utf8));
    }
}
```

There are two solutions for this problem: 

  * create new `Text` class each time a new tuple is going to be emitted ([some information][1] over the internet indicate this might not be the worst idea)
  * clear `Text` instance by calling `set("")`

But which is the better one? The only metric in this case is speed of execution... 
Below there is a small test case I wrote to check, which solution is better.

```java
@Test
public void shouldTellWhatIsBetterCreatingTextOrReusing() throws Exception {

    byte[] longText = toBytes("123456789");

    int iterations = 1000000;

    System.out.println("reusing... ");
    long start1 = System.nanoTime();
    Text text = new Text();
    for (int i = 0; i < iterations; i++) {
        text.set(longText);
        text.set(""); // resets the 
    }
    long stop1 = System.nanoTime();

    long diff1 = stop1 - start1;
    System.out.println("reusing took: " + diff1 + " " + (diff1/1000000000));

    System.out.println("creating... ");
    long start2 = System.nanoTime();
    for (int i = 0; i < iterations; i++) {
        new Text(longText);
    }
    long stop2 = System.nanoTime();

    long diff2 = stop2 - start2;
    System.out.println("creating took: " + diff2 + " " + (diff2/1000000000));
}

```

The result is ... interesting:

```
reusing... 
reusing took: 104000088 0
creating... 
creating took: 4898000991 4
```

So actually it is faster to clear the `Text` instance. I was expecting that kind of result, but to be honest the difference is much higher than I thought. Either way - this only means that all those tutorials contain good advice! 

[1]: http://programmers.stackexchange.com/questions/149563/should-we-avoid-object-creation-in-java
