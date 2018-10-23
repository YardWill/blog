> demo 地址 https://github.com/YardWill/beginner-reactive-programming-with-rxjs

> 我们通过四个例子来看rxjs.

## 1. example-1

```
const promise = new Promise((resolve) => {
  setTimeout(() => {
    resolve('Hello from a Promise!');
  }, 2000);
});

promise.then(value => console.log(value));
```
这是一个简单的promise实现2000ms之后的输出。我们接下来使用rxjs来试试如何实现这个功能。


## 2. example-2

```
import { Observable } from 'rxjs/Observable';

const observable = new Observable((observer) => {
  setTimeout(() => {
    observer.next('Hello from a Observable!');
  }, 2000);
});

observable.subscribe(value => console.log(value));
```

我们先通过Observable的接口来注册一个可观测对象，（如果对观察者模式不太熟悉的话可以先看这篇文章[观察者简单实现](https://www.jianshu.com/p/9061b7da246a)）

之后我们使用subscribe来订阅一个事件（subscribe函数内部执行在observer内注册的函数），当observer触发next的时候就执行subscribe内注册的函数。

我们来看看源码。
以下是Observable的构造函数，可以看出，除了绑定subscribe函数之外，什么都没有做。
```
constructor(subscribe?: (this: Observable<T>, subscriber: Subscriber<T>) => TeardownLogic) {
  if (subscribe) {
    this._subscribe = subscribe;
  }
}
```

下面是Observable类上的subscribe方法。简单来讲就是去执行了在constructor内注册的subscribe函数，并将observerOrNext push到subscriptions内。然后return的对象是一个subscriptions。
```
  subscribe(observerOrNext?: PartialObserver<T> | ((value: T) => void),
            error?: (error: any) => void,
            complete?: () => void): Subscription {

    const { operator } = this;
    const sink = toSubscriber(observerOrNext, error, complete);

    if (operator) {
      operator.call(sink, this.source);
    } else {
      sink.add(
        this.source || (config.useDeprecatedSynchronousErrorHandling && !sink.syncErrorThrowable) ?
        this._subscribe(sink) :
        this._trySubscribe(sink)
      );
    }

    if (config.useDeprecatedSynchronousErrorHandling) {
      if (sink.syncErrorThrowable) {
        sink.syncErrorThrowable = false;
        if (sink.syncErrorThrown) {
          throw sink.syncErrorValue;
        }
      }
    }

    return sink;
  }
```
你一定觉得很疑惑Observable、subscribe、subscriptions这些名字都代表什么意思。我们下面来讲一个通俗的故事。

> 从前有一家鲜奶公司，提供不同种类的鲜奶，每天都会送货一次。有一天，小明花了20/天的钱去订了这家公司的进口鲜奶，小花花了10元/天的钱订了国产鲜奶。那么公司收到的订单列表就是这两个，公司会在固定时间去把鲜奶送到用户的手上。

我们看看这个故事里面的observable、subscribe、subscriptions分别是什么？
1. observable： 毫无疑问，鲜奶公司是一个可订阅对象，我们可以向鲜奶公司订阅我们需要的鲜奶。
2. subscribe： subscribe这是一个动作，小明和小花去定了这家鲜奶公司的牛奶。
3. subscriptions： 订完牛奶后，小明和小花的订单就已经在鲜奶公司的订单列表上了，这个订单列表就是subscriptions。
4. 另外，我们将setTimeout改成setInterval，这时我们就可以想象鲜奶公司每天都会触发发货的工作，也就是执行next方法。next方法可以当做是鲜奶公司对照着订单列表对小明和小花进行发货。

这样看起来，理解这几个对象应该不难了吧。我们把这个故事改编成代码。
```
import { Observable } from 'rxjs/Observable';

// 鲜奶公司
const interval$ = new Observable((observer) => {
  let count = 0;
  const interval = setInterval(() => {
    console.log('鲜奶公司准时发货');
    observer.next(count += 1);
  }, 1000);

  return () => {
    clearInterval(interval);
  };
});

// 小明订奶
const littleMing = count => console.log('小明收到', count, '瓶奶');
// 小花订奶
const littleHua = count => console.log('小花收到', count, '瓶奶');
const subscription1 = interval$.subscribe(littleMing);
const subscription2 = interval$.subscribe(littleHua);
```
subscriptions包含[subscription1, subscription2]。
那么接下来我们再来思考一个问题，如果小明不想继续订牛奶了，他应该怎么通知鲜奶公司不再发货？我们来看 example-3

##  3. example-3
我们通过上面的故事来修改一下我们的代码。
```
import { Observable } from 'rxjs/Observable';

// 鲜奶公司
const interval$ = new Observable((observer) => {
  let count = 0;
  const interval = setInterval(() => {
    console.log('鲜奶公司准时发货');
    observer.next(count += 1);
  }, 1000);

  return () => {
    clearInterval(interval);
  };
});

// 小明订奶
const littleMing = count => console.log('小明收到', count, '瓶奶');
// 小花订奶
const littleHua = count => console.log('小花收到', count, '瓶奶');
const subscription1 = interval$.subscribe(littleMing);
const subscription2 = interval$.subscribe(littleHua);
setTimeout(() => subscription1.unsubscribe(), 3000);
```

我们可以看到在最后我们把subscription1给unsubscribe了，我们看看unsubscribe函数内做了什么？

```
  /**
   * Disposes the resources held by the subscription. May, for instance, cancel
   * an ongoing Observable execution or cancel any other type of work that
   * started when the Subscription was created.
   * @return {void}
   */
  unsubscribe(): void {
    let hasErrors = false;
    let errors: any[];

    if (this.closed) {
      return;
    }

    let { _parent, _parents, _unsubscribe, _subscriptions } = (<any> this);

    this.closed = true;
    this._parent = null;
    this._parents = null;
    // null out _subscriptions first so any child subscriptions that attempt
    // to remove themselves from this subscription will noop
    this._subscriptions = null;

    let index = -1;
    let len = _parents ? _parents.length : 0;

    // if this._parent is null, then so is this._parents, and we
    // don't have to remove ourselves from any parent subscriptions.
    // 移除subscription
    while (_parent) {
      _parent.remove(this);
      // if this._parents is null or index >= len,
      // then _parent is set to null, and the loop exits
      _parent = ++index < len && _parents[index] || null;
    }

    if (isFunction(_unsubscribe)) {
      let trial = tryCatch(_unsubscribe).call(this);
      if (trial === errorObject) {
        hasErrors = true;
        errors = errors || (
          errorObject.e instanceof UnsubscriptionError ?
            flattenUnsubscriptionErrors(errorObject.e.errors) : [errorObject.e]
        );
      }
    }
    
    if (isArray(_subscriptions)) {

      index = -1;
      len = _subscriptions.length;

      while (++index < len) {
        const sub = _subscriptions[index];
        if (isObject(sub)) {
          let trial = tryCatch(sub.unsubscribe).call(sub);
          if (trial === errorObject) {
            hasErrors = true;
            errors = errors || [];
            let err = errorObject.e;
            if (err instanceof UnsubscriptionError) {
              errors = errors.concat(flattenUnsubscriptionErrors(err.errors));
            } else {
              errors.push(err);
            }
          }
        }
      }
    }

    if (hasErrors) {
      throw new UnsubscriptionError(errors);
    }
  }
```
这里的代码理解起来也不难，最后去执行了clearInterval(interval)将定时器去掉，并把当前的订单（subscription）移除出订单列表(subscriptions)。

## 4. example-4
接下来小明和小花都有各自的需求改变，比如小明想要每天两瓶奶，而小花需要隔天收到一瓶奶，那么我们应该怎么做呢？看下面代码。
```
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/filter';

// 鲜奶公司
const interval$ = new Observable<number>((observer) => {
  let count = 0;
  const interval = setInterval(() => {
    console.log('鲜奶公司准时发货');
    observer.next(count += 1);
  }, 1000);

  return () => {
    clearInterval(interval);
  };
});

// 小明订奶
const littleMing = count => console.log('小明收到', count, '瓶奶');
// 小花订奶
const littleHua = count => console.log('小花收到', count, '瓶奶');

// 小明打算每天多订一份的鲜奶
const subscription1 = interval$
  .map(value => value * 2)
  .subscribe(littleMing);

// ----1----2----3----4--->
//      map => x * 2
// ----2----4----6----8--->

// 小花打算让鲜奶公司隔天送一瓶
const subscription2 = interval$
  .filter(value => value % 2 === 0)
  .map(value => value / 2)
  .subscribe(littleHua);

// ----1----2----3----4--->
//      filter & map
// ---------1---------2--->
```
在这里我们引入了map和filter这两个rx的操作符，来实现我们需要变更的需求，看起来是不是很简单。当然还有更多的操作符（我们就不一一介绍了），操作符也是rxjs内的很大一部分组成，可以把它比作是lodash内的工具类。

## 5. example-5
我们最后再来看看rxjs在前端事件监听上的用法。
```
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/fromEvent';
import 'rxjs/add/observable/merge';
import 'rxjs/add/operator/scan';
import 'rxjs/add/operator/map';

const incrementClicks$ = Observable.fromEvent(document.getElementById('increment'), 'click');
const decrementClicks$ = Observable.fromEvent(document.getElementById('decrement'), 'click');

Observable
  .merge(incrementClicks$, decrementClicks$)
  .map((event: any) => parseInt(event.target.value, 10))
  .scan((total, value) => total + value, 0)
  .subscribe((total) => {
    document.getElementById('counter').innerText = total.toString();
  });
```

merge将两个Observable对象合成一个Observable对象，然后map对数据进行操作。
scan可以比作一个reducer函数，每一次click之后会对数据进行处理，并保留之前的数据。
最后我们去subscribe这个事件，然后做出相应修改。

> 个人博客 https://www.yardwill.com/