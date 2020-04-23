---
layout: post
title: "Fabu Monitor"
author: Slahser
categories: [Fabu]
tags: [Monitor]
image: assets/images/react.png
rating: 5
---

- ### useEffect

副作用钩子，每一次的渲染行为都有可能会产生一些副作用，例如，发送请求，对DOM进行修改，blabla。官方文档里的例子已经非常清楚这个钩子到底能做些什么，
此文是对 **useEffect** 造成的逻辑些陷阱，以及和其他钩子做一个对比分析。

{% highlight javascript %}
import React, { useEffect, useState } from 'react';
const Example = () => {
	const [count, setCount] = useState(0);
	const [a, setA] = useState(1);
	useEffect(() => {
		setInterval(() => console.log(count), 5000);
	});
	return (
		<div>
			<div>{count}</div>
			<button onClick={() => setCount(prev => prev + 1)}>increase</button>
			<button onClick={() => setA(prev => prev + 1)}>increase</button>
		</div>
	)
}
export default Example;
{% endhighlight %}

一个例子，想想看，如果连续点击3下 **increase** 按钮，会有什么样的情况发生呢。

由于该副作用没有 **deps**, 如果每次 **setState** 之后（哪怕该state可以都没有插入DOM，由于React的更新方式是创建一个新的update object），都会触发render，
按照官网来说，每次render之后，都会造成一次副作用钩子被触发，第一次count为0（这一次是初始化时候的，并非update造成），第二次count为1，第三次count为2，第三次count为3（因为每一次传进钩子的count值还不一样），所以执行结果是
没隔5s就会打印一次0，1，2，3。

所以这样的打印只是反映当时count的一个快照，那如何才能反映实时的count值呢

- ### useRef

既然useEffect下的定时器只能反映当时的一个快照，那总有一个办法可以把count值存住，在每次打印的时候去获取实时的count值（也就是它们应该相同，且都为那一个count值）

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
          })
        }
      >
        increase
      </button>
    </div>
  );
};
export default Example;
{% endhighlight %}

