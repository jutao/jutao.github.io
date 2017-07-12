---
title: RxJava2 入门
tags:
- 响应式编程
- 学习笔记
categories:
- Java 基础
- Android 第三方
---

# 与RxJava 1.x 的差异
以前用过 RxJava 1.x ,出了 2 之后一直没有系统的看过，今天花点时间仔细看看。RxJava 2.x 是按照 Reactive-Streams specification 规范完全的重写的，是完全独立于RxJava 1.x 而存在，它改变了以往RxJava的用法。所以还是有必要重新看一下的。先看一下 RxJava 1.x 和 2.x 的差异吧。

## Nulls
 这是一个很大的变化，熟悉RxJava 1.x 的童鞋一定都知道，1.x 是允许我们在发射事件的时候传入 null 值的，但现在我们的 2.x 不支持了，不信你试试？ 大大的 NullPointerException 教你做人。这意味着 Observable<Void> 不再发射任何值，而是正常结束或者抛出空指针。

## Flowable
  在 RxJava 1.x 中关于介绍 backpressure 部分有一个小小的遗憾，那就是没有用一个单独的类，而是使用 Observable 。而在 2.x 中 Observable 不支持背压了，将用一个全新的 Flowable 来支持背压。
  ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-878015bf0fa366db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    或许对于背压，有些小伙伴们还不是特别理解，这里简单说一下。大概就是指在异步场景中，被观察者发送事件的速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。感兴趣的小伙伴可以模拟这种情况，在差距太大的时候，我们的内存会猛增，直到OOM。而我们的 Flowable 一定意义上可以解决这样的问题。

## Single/Completable/Maybe
  其实这三者都差不多，Single 顾名思义，只能发送一个事件，和 Observable 接受可变参数完全不同。而Completable 侧重于观察结果，而Maybe 是上面两种的结合体。也就是说，当你只想要某个事件的结果（true or false）的时候，你可以使用这种观察者模式。    

## 线程调度相关
这一块基本没什么改动，但细心的小伙伴一定会发现，RxJava 2.x 中已经没有了Schedulers.immediate() 这个线程环境，还有Schedulers.test()。

## Function 相关
熟悉 1.x 的小伙伴一定都知道，我们在1.x 中是有Func1，Func2.....FuncN的，但2.x 中将它们移除，而采用Function 替换了Func1，采用BiFunction 替换了Func 2..N。并且，它们都增加了throws Exception，也就是说，妈妈再也不用担心我们做某些操作还需要try-catch了。

## 其他操作符相关
  如Func1...N 的变化，现在同样用Consumer和BiConsumer对Action1 和Action2进行了替换。后面的Action都被替换了，只保留了ActionN。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-c84517b52ab42264.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 基本操作
## create 操作符
create操作符应该是最常见的操作符了，主要用于产生一个Obserable被观察者对象，为了方便大家的认知，接下来中统一把被观察者Observable称为发射器（上游事件），观察者Observer称为接收器（下游事件）。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-23c76b89e93bbc35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
  Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
      Log.e(TAG, "Observable emit 1" + "\n");
      e.onNext(1);
      Log.e(TAG, "Observable emit 2" + "\n");
      e.onNext(2);
      Log.e(TAG, "Observable emit 3" + "\n");
      e.onNext(3);
      Log.e(TAG, "Observable emit 4" + "\n");
      e.onNext(4);
    }
  }).subscribe(new Observer<Integer>() {
    private int i;
    private Disposable mDisposable;

    @Override public void onSubscribe(@NonNull Disposable d) {
      mDisposable = d;
    }

    @Override public void onNext(@NonNull Integer integer) {
      Log.e(TAG, "onNext : value : " + integer + "\n" );
      i++;
      if(i==2){
        mDisposable.dispose();
        Log.e(TAG, "onNext : isDisposable : " + mDisposable.isDisposed() + "\n");
      }
    }

    @Override public void onError(@NonNull Throwable e) {
      Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );
    }

    @Override public void onComplete() {
      Log.e(TAG, "onComplete" + "\n" );
    }
  });
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-bccaf6389b237167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到 2.x 中有一个Disposable概念，这个东西可以直接调用切断，可以看到，当它的isDisposed()返回为false的时候，接收器能正常接收事件，但当其为true的时候，接收器停止了接收。所以可以通过此参数动态控制接收事件了。

如果发射时间这么调用的话：
```java
@Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
  Log.e(TAG, "Observable emit 1" + "\n");
  e.onNext(1);
  Log.e(TAG, "Observable emit 2" + "\n");
  e.onNext(2);
  e.onComplete();
  Log.e(TAG, "Observable emit 3" + "\n");
  e.onNext(3);
  Log.e(TAG, "Observable emit 4" + "\n");
  e.onNext(4);
}
```
运行结果如下：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-59b0822514a6a7dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到：在发射事件中，我们在发射了数值2之后，直接调用了 e.onComlete()，虽然无法接收事件，但发送事件还是继续的.

## Map
Map基本算是RxJava中一个最简单的操作符了，熟悉RxJava 1.x的知道，它的作用是对发射时间发送的每一个事件应用一个函数，是的每一个事件都按照指定的函数去变化，而在 2.x 中它的作用几乎一致。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-ad1c2d9a5e6b2ce5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

示例代码如下：

```java
final int[] a={1,2,3};
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    for (int i : a) {
      e.onNext(i);
    }
  }
}).map(new Function<Integer, String>() {
  @Override public String apply(@NonNull Integer integer) throws Exception {
    return integer+"被转换成字符串了";
  }
}).subscribe(new Consumer<String>() {
  @Override public void accept(@NonNull String s) throws Exception {
    Log.e(TAG, s);
  }
});
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-6982fd3ca1e72b1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
是的，map基本作用就是将一个Observable通过某种函数关系，转换为另一种Observable，上面例子中就是把我们的Integer数据变成了String类型。从Log日志显而易见。

## Zip
zip专用于合并事件，该合并不是连接（连接操作符后面会说），而是两两配对，也就意味着，最终配对出的Observable发射事件数目只和少的那个相同。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-fe1359915847ffcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.zip(getStringObservable(), getIntegerObservable(), new BiFunction<String, Integer, String>() {
  @Override public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
    return s+integer;
  }
}).subscribe(new Consumer<String>() {
  @Override public void accept(@NonNull String s) throws Exception {
    Log.e(TAG, "zip : accept : " + s + "\n");
  }
});

private Observable<String> getStringObservable() {
  return Observable.create(new ObservableOnSubscribe<String>() {
    @Override public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
      if(!e.isDisposed()){
        e.onNext("A");
        Log.e(TAG, "String emit : A \n");
        e.onNext("B");
        Log.e(TAG, "String emit : B \n");
        e.onNext("C");
        Log.e(TAG, "String emit : C \n");
      }
    }
  });
}
private Observable<Integer> getIntegerObservable() {
  return Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
      if(!e.isDisposed()){
        e.onNext(1);
        Log.e(TAG, "Integer emit : 1 \n");
        e.onNext(2);
        Log.e(TAG, "Integer emit : 2 \n");
        e.onNext(3);
        Log.e(TAG, "Integer emit : 3 \n");
        e.onNext(4);
        Log.e(TAG, "Integer emit : 4 \n");
        e.onNext(5);
        Log.e(TAG, "Integer emit : 5 \n");
      }
    }
  });
}
```
运行结果如下：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-a86a3f5821c6903d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要注意的是：

1. zip 组合事件的过程就是分别从发射器A和发射器B各取出一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行的，所以上面截图中，可以看到，1永远是和A 结合的，2永远是和B结合的。

2. 最终接收器收到的事件数量是和发送器发送事件最少的那个发送器的发送事件数目相同，所以如截图中，4和5很孤单，没有人愿意和它们交往，孤独终老的单身狗。

## Concat
对于单一的把两个发射器连接成一个发射器，虽然 zip 不能完成，但我们还是可以自力更生，官方提供的 concat 让我们的问题得到了完美解决。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-af661d4e8941edd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
Observable.concat(Observable.just(1, 3, 5, 78), Observable.fromArray("A", "B", "C"))
    .subscribe(new Consumer<Object>() {
      @Override public void accept(@NonNull Object o) throws Exception {
        Log.e(TAG, "concat : " + o + "\n");
      }
    });
```
运行结果如下:

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-71d90c9dc828d993.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图，可以看到。发射器B把自己的三个孩子送给了发射器A，让他们组合成了一个新的发射器，非常懂事的孩子，有条不紊的排序接收。

## FlatMap
FlatMap 是一个很有趣的东西，我坚信你在实际开发中会经常用到。它可以把一个发射器Observable 通过某种方法转换为多个Observables，然后再把这些分散的Observables装进一个单一的发射器Observable。但有个需要注意的是，flatMap并不能保证事件的顺序，如果需要保证，需要用到我们下面要讲的ConcatMap。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-bf1ca8bd32eaca94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
  }
}).flatMap(new Function<Integer, ObservableSource<String>>() {
  @Override public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
    list.add("value:" + integer);
    int delayTime = (int) (1 + Math.random() * 10);
    return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
  }
}).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
  @Override public void accept(@NonNull String s) throws Exception {
    Log.e(TAG, s);
  }
});
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-09bf4a298016102b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一切都如我们预期中的有意思，为了区分concatMap（下一个会讲），我在代码中特意动了一点小手脚，我采用一个随机数，生成一个时间，然后通过delay（后面会讲）操作符，做一个小延时操作，而查看Log日志也确认验证了我们上面的说法，它是无序的。

## concatMap
上面其实就说了，concatMap 与 FlatMap 的唯一区别就是 concatMap 保证了顺序，所以，我们就直接把 flatMap 替换为 concatMap 验证吧。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
  }
}).concatMap(new Function<Integer, ObservableSource<String>>() {
  @Override public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
    list.add("value:" + integer);
    int delayTime = (int) (1 + Math.random() * 10);
    return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
  }
}).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread()).subscribe(new Consumer<String>() {
  @Override public void accept(@NonNull String s) throws Exception {
    Log.e(TAG, s);
  }
});
```
运行结果如下：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-82fd843082be4a9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果的确和我们预想的一样，是按照顺序发送的。

## distinct
这个操作符非常的简单、通俗、易懂，就是简单的去重。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-d69a8097faa1a2d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(1);
    e.onNext(2);
    e.onNext(2);
    e.onNext(3);
    e.onNext(3);
  }
})
    .subscribeOn(Schedulers.newThread())
    .observeOn(AndroidSchedulers.mainThread())
    .distinct()
    .subscribe(new Consumer<Integer>() {
      @Override public void accept(@NonNull Integer s) throws Exception {
        Log.e(TAG, s + "");
      }
    });
```
运行结果如下，重复的都去掉了，不解释：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-202133fdc36de8d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## Filter
 信我，Filter 你会很常用的，它的作用也很简单，过滤器嘛。可以接受一个参数，让其过滤掉不符合我们条件的值。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-555c3596647bc849.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
    e.onNext(4);
    e.onNext(5);
    e.onNext(6);
  }
})
    .subscribeOn(Schedulers.newThread())
    .observeOn(AndroidSchedulers.mainThread())
    .filter(new Predicate<Integer>() {
      @Override public boolean test(@NonNull Integer integer) throws Exception {
        return integer>3;
      }
    })
    .subscribe(new Consumer<Integer>() {
      @Override public void accept(@NonNull Integer s) throws Exception {
        Log.e(TAG, s + "");
      }
    });
```

运行结果如下，不满足 test 函数的值都被过滤掉了：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-6c778402192b29ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## buffer
 buffer 操作符接受两个参数，buffef(count,skip)，作用是将 Observable 中的数据按 skip (步长) 分成最大不超过 count 的 buffer ，然后生成一个 Observable 。也许你还不太理解，我们可以通过我们的示例图和示例代码来进一步深化它。


 ![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-603f6a74c62adf9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.just(1, 2, 3, 4, 5).buffer(3, 2).subscribe(new Consumer<List<Integer>>() {
  @Override public void accept(@NonNull List<Integer> integers) throws Exception {
    Log.e(TAG, "buffer size : " + integers.size() + "\n");
    Log.e(TAG, "buffer value : ");
    for (Integer integer : integers) {
      Log.e(TAG, integer + "");
    }
  }
});
```

运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-8c29428bb8596c77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如图，我们把1,2,3,4,5依次发射出来，经过buffer 操作符，其中参数 skip 为3， count 为2，而我们的输出 依次是 123，345，5。显而易见，我们 buffer 的第一个参数是count，代表最大取值，在事件足够的时候，一般都是取count个值，然后每天跳过skip个事件。其实看 Log 日志，我相信大家都明白了。

## timer
timer 很有意思，相当于一个定时任务。在1.x 中它还可以执行间隔逻辑，但在2.x中此功能被交给了 interval，下一个会介绍。但需要注意的是，timer 和 interval 均默认在新线程。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-1cdf87a3d2fec9f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
final long now=System.currentTimeMillis();
Log.e(TAG,
    "nowTime:" + now);
Observable.timer(2, TimeUnit.SECONDS)
    .subscribeOn(Schedulers.io())
    //timer 默认在新线程，所以需要切换回主线程
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Consumer<Long>() {
      @Override public void accept(@NonNull Long aLong) throws Exception {
        Log.e(TAG,
            "after:" + (System.currentTimeMillis()-now)+"ms");
      }
    });
```

运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-e703531b0ffab1d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，大约延时了两秒执行接收。

## interval
如同我们上面可说，interval 操作符用于间隔时间执行某个操作，其接受三个参数，分别是第一次发送延迟，间隔时间，时间单位。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-e62e3d397d3ff006.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
mDisposable = Observable.interval(0, 1000, TimeUnit.MILLISECONDS)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Consumer<Long>() {
      @Override public void accept(@NonNull Long aLong) throws Exception {
        Log.e(TAG, "NowTime:" + getNowStrTime());
      }
    });

    public static String getNowStrTime(){
      long time = System.currentTimeMillis();
      @SuppressLint("SimpleDateFormat") SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
      return sdf.format(new Date(time));
    }
```
输出结果如下：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-a92e17644ff9b27c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

的确如我们所想是一秒执行一次，然而，心细的小伙伴可能会发现，由于我们这个是间隔执行，所以当我们的Activity都销毁的时候，实际上这个操作还依然在进行，所以，我们得花点小心思让我们在不需要它的时候干掉它。查看源码发现，我们subscribe(Cousumer<? super T> onNext)返回的是Disposable，我们可以在这上面做文章。

加上下面的代码可以保证不会内存泄漏：

```java
@Override protected void onDestroy() {
  super.onDestroy();
  if (mDisposable != null && !mDisposable.isDisposed()) {
    mDisposable.dispose();
  }
}
```

## doOnNext
其实觉得 doOnNext 应该不算一个操作符，但考虑到其常用性，我们还是咬咬牙将它放在了这里。它的作用是让订阅者在接收到数据之前干点有意思的事情。假如我们在获取到数据之前想先保存一下它，无疑我们可以这样实现。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
    e.onNext(4);
  }
}).doOnNext(new Consumer<Integer>() {
  @Override public void accept(@NonNull Integer integer) throws Exception {
    Log.e(TAG, "doOnNext 保存 " + integer + "成功" + "\n");
  }
}).subscribe(new Consumer<Integer>() {
  @Override public void accept(@NonNull Integer integer) throws Exception {
    Log.e(TAG, "doOnNext :" + integer + "\n");
  }
});
```
运行结果如下：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-64704ce2672ab371.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## skip
skip 很有意思，其实作用就和字面意思一样，接受一个 long 型参数 count ，代表跳过 count 个数目开始接收。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-09c61aba056997f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
    e.onNext(4);
  }
}).skip(1).subscribe(new Consumer<Integer>() {
  @Override public void accept(@NonNull Integer integer) throws Exception {
    Log.e(TAG, "doOnNext :" + integer + "\n");
  }
});
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-328c41fa54733455.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## take
take，接受一个 long 型参数 count ，代表至多接收 count 个数据。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-3a7d8393623657e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
  @Override public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
    e.onNext(1);
    e.onNext(2);
    e.onNext(3);
    e.onNext(4);
  }
}).take(3).subscribe(new Consumer<Integer>() {
  @Override public void accept(@NonNull Integer integer) throws Exception {
    Log.e(TAG, "doOnNext :" + integer + "\n");
  }
});
```
运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-0bdea9318a398922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## just
   just，没什么好说的，其实在前面各种例子都说明了，就是一个简单的发射器依次调用onNext()方法。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-56dda9065be1311e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```java
Observable.just(1,2,3).subscribe(new Consumer<Integer>() {
  @Override public void accept(@NonNull Integer integer) throws Exception {
    Log.e(TAG,"accept : onNext : " + integer + "\n" );
  }
});
```

运行结果如下：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3054656-ab51266eb1c2bfe1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
