---
layout: post
title: "React Hook"
author: Slahser
categories: [Course]
tags: [React]
image: assets/images/react.png
rating: 5
---

### 前言
关于 **react hook** 的一些理解，想到哪里就说到哪里吧，也没有看过其他的关于这方面的文章，单单是自己领悟，如果有错的地方，欢迎指正。
- ### useEffect
副作用钩子，每一次的渲染行为都有可能会产生一些副作用，例如，发送请求，对DOM进行修改，blabla。官方文档里的例子已经很清楚这个钩子到底能做些什么，
此文是对 **useEffect** 造成的逻辑陷阱，以及和其他钩子做一个对比分析。
{% highlight javascript %}
import React, { useEffect, useState } from 'react';
const Example = () => {
	const [count, setCount] = useState(0);
	useEffect(() => {
		setInterval(() => console.log(count), 5000);
	});
	return (
		<div>
			<div>{count}</div>
			<button onClick={() => setCount(prev => prev + 1)}>increase</button>
		</div>
	);
}
export default Example;
{% endhighlight %}

一个例子，想想看，如果连续点击3下 **increase** 按钮，会有什么样的情况发生呢。

由于该副作用没有 **deps**, 如果每次 **setState** 之后（哪怕该state可以都没有插入DOM，由于React每次会创建一个新的update object），都会触发render，
按照官网资料，每次render之后，都会造成一次副作用钩子被触发，第一次count为0（这一次是初始化时候的，并非update造成），第二次count为1，第三次count为2，第三次count为3（因为每一次传进钩子的count值还不一样），所以执行结果是
每隔5s就会打印一次0，1，2，3。


所以这样的打印只是反映当时count的一个快照，那如何才能反映实时的count值呢

- ### useRef
既然useEffect下的定时器只能反映当时的一个快照，那总有一个办法可以把count值存住，在每次打印的时候去获取实时的count值（也就是它们应该完全相同）。
{% highlight javascript %}
import React, { useEffect, useState, useRef } from "react";
const Example = () => {
  const [count, setCount] = useState(0);
  const ref = useRef(count);
  useEffect(() => {
    setInterval(() => console.log(ref.current), 5000);
  }, [count]);
  return (
    <div>
      <div>{count}</div>
      <button
        onClick={() =>
          setCount(prev => {
            const ret = prev + 1;
            ref.current = ret;
            return ret;
          });
        }
      >
        increase
      </button>
    </div>
  );
};
export default Example;
{% endhighlight %}

当**useRef**如果没有initialValue时，useRef会创建一个新的对象，保存**current**值，并且会直接将该对象**seal**
当它有了initialValue时，会直接把这个值（原始类型）或者对象（引用类型）返回。因此这个值始终会是最后修改的那一次，也就是实时的，不再是一个快照了。

- ### useMemo

**useMemo** Hook 在网上的例子非常多，但是还是会发现很多初学者会把useMemo和useEffect搞混，以为可以用**useEffect**去实现**useMemo**的效果，这是一个误解。

useEffect的执行周期是在渲染之后，而**计算开销**以及条件等行为都是在渲染之前产生的，因此要摆脱重新**computed**，不可能在**useEffect**里完成。**useMemo**就闪亮登场了。顾名思义，**useMemo**其实就是在memorize里面去取，如果发现自己的**deps**是没有变化的，那么默认它的输出也是不会有变化的，而恰好这个时候memorise里面有这个函数执行之后遗留的值，那么我就把这个值直接返回，省掉那些计算的开销，这就像之前**React Class Component**里提供的**shouldComponentUpdate(传说中的SCU)**过程。例如:
{% highlight javascript %}
import React, { useState } from "react";

const Example = ({ bigArray }) => {
  const [, setReRender] = useState(true);

  const getBigArrayMark = array => {
    let ret;
    const start = new Date().getTime();
    for (let i = 0; i < array.length; i++) {
      if (array[i]) {
        ret = i;
        break;
      }
    }
    const end = new Date().getTime();
    console.log("总耗时: ", end - start);
    return ret;
  };

  return (
    <div>
      <div>{getBigArrayMark(bigArray)}</div>
      <button onClick={() => setReRender(prev => !prev)}> re-render </button>
    </div>
  );
};
export default Example;
{% endhighlight %}

首先这个bigArray我们假设它是不变的，每次调用**setReRender**都会重新导致**getBigArrayMark**会被重新调用一次，这个和**useEffect**并没有什么关系，但是这造成很多不必要的开销。优化之后：
{% highlight javascript %}
import React, { useState, useMemo } from "react";

const Example = ({ bigArray }) => {
  const [, setReRender] = useState(true);

  const getBigArrayMark = array => {
    let ret;
    const start = new Date().getTime();
    for (let i = 0; i < array.length; i++) {
      if (array[i]) {
        ret = i;
        break;
      }
    }
    const end = new Date().getTime();
    console.log("总耗时: ", end - start);
    return ret;
  };

  const memo_value = useMemo(
    () => getBigArrayMark(bigArray), [bigArray]);
  return (
    <div>
      <div>{memo_value}</div>
      <button onClick={() => setReRender(prev => !prev)}> re-render </button>
    </div>
  );
};
export default Example;
{% endhighlight %}
现在不管你如何**setState**, 只要**deps**没变，都不会触发**getBigArrayMark**，从而提高页面的性能，既节省了CPU，又节省了内存，美滋滋。

- ### useCallback
函数版的**useMemo**，当父组件传递函数给子组件时，如果父组件更新，因为这个函数发生了改变，会导致子组件也更新，那**useCallback**就可以去避免这样的情况，如果父组件更新，
在**deps**都没有改变的情况下，取出来memorize里的函数由于就是上一次创建的函数，所以不会导致子组件的更新，避免不必要的渲染。例如：
{% highlight javascript %}
//  parent.js
const Parent = () => {
  const [count, setCount] = useState(0);
  const [, setValue] = useState(0x000000);
  const callback = useCallback(() => count, [count]);
  return (
    <div>
      <Header callback={callback}/>
      {count}
      <button onClick={() => setCount(prev => prev + 1)}>Increase Count</button>
      <button onClick={() => setValue(prev => prev + 1)}>Increase Value</button>
    </div>
  );
};


// child.js
const Child = ({ callback }) => {
  const [count, setCount] = useState(() => callback());
  useEffect(() => {
    console.log("update hEad");
    setCount(callback());
  }, [callback]);
  return <div>{count}</div>;
};
{% endhighlight %}
从上面的例子可以看出，点击Increase Count，函数更新，触发child的副作用，重新setCount；点击Increase Value，函数不会变化，也不能触发Child的副作用，子组件不变。





