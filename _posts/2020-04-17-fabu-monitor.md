---
layout: post
title: "Fabu Monitor"
author: Slahser
categories: [Fabu]
tags: [Monitor]
image: assets/images/monitor.png
rating: 5
---

**Fabu Monitor** 是一个实时反映自动驾驶状态的可视化系统。比如当前车辆的速度，路线规划，周围障碍物的情况等等。

它的代码分为三个部分

```
├── Monitor
│     └── backend
│     └── frontend
│     └── conf
```

这篇文章主要是关于 _frontend_ 部分的代码，这里分为两个部分：

- 由 **three.js** 控制的3D渲染部分
- 由 **react.js** 控制的组件部分

```
├── frontend/src
│     └── components
│     └── renderer
│     └── store
│     └── styles
│     └── utils
│     └── index.hbs
│     └── index.js
```

- components 里是在 [ **material-ui** ](https://https://material-ui.com "Material-ui") 的基础上封装的一些UI组件。
	- 单例组件直接放在 **components** 文件夹下, 禁止开放 **className** 属性供外部调用，如必须提供样式调用，可使用 **width** 或者 **height** 这样的方式让外部单独使用。
	- 可复用组件放在 **components/common** 文件夹下，必须提供 **className** 属性供外部使用，方便外部重写组件最外层元素样式。
	- **makeStyles** 定义的公共样式需放在组件内部。
	- 统一 **hook** 写法，UI组件均使用FC方式定义，例如：
			{% highlight javascript %}
			const exampleComponent = () => <div>Hello World</div>;
			{% endhighlight %}

- renderer 里是存放的是关于3D场景渲染类的封装，主要是在[ **three.js** ](https://threejs.org "Three.js")上做的封装。目前包括自车，障碍物，点云，地图以及某些特地场景的组件比如桥吊，挂车。
	- 每个文件只能定义一个渲染类，例如：
{% highlight javascript %}
class Obstacles {
	/***/
	getMesh(index) {
		if (index < this.obstacleCache.length) {
			return this.obstacleCache[index];
		}
		const obstacle = this.addMesh();
		this.obstacleCache.push(obstacle);
		this.scene.add(obstacle);
		return obstacle;
	}
}
export default Obstacles
{% endhighlight %}
	


