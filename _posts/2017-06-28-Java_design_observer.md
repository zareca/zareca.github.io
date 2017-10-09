---
layout: post
title: Java设计模式(19)---观察者模式
date: 2017-06-28 
tag: Java设计模式
---

### 概念 ###

观察者模式定义了一种一对多的依赖关系,让多个观察者对象同时监听某一主题对象.这个主题对象在状态上发生变化时,会通知所有观察者对象,使他们能够自动更新自己

### 组成 ###

**抽象主题角色:**把所有观察者对象引用保存在一个集合中,每个抽象主题角色都可以有任意数量的观察者,抽象主题提供一个接口,可以增加和删除观察者角色,一般用一个抽象类和接口实现.

**抽象观察者角色:**为所有具体的观察者定义一个接口,在得到主题通知时更新自己

**具体主题角色:**在具体的主题内部状态改变时,给所有登记过的观察者发出通知,具体主题角色通常都是一个子类实现

**具体观察者角色:**该角色实现抽象观察者角色所要求更新的接口.以便使本身状态和主题状态相协调,通常用一个子类实现.如果需要,具体观察者可以保存一个指向具体主题角色的引用.

### 推模型和拉模型 ###
**推模型:**主题对象向观察者推送主题的详细信息,不管观察者是否需要,推送的信息通常是主题对象的全部或者部分数据.

**拉模型:**一般这种模型实现中,会把主题对象自身通过update方法传递给观察者,这样在观察者需要获取数据的时候,就可以通过这个引用来获取了.

### 代码 ###

**主题抽象类**

	public abstract class SubJect {

		// 用来保存注册的观察者对象
		private List<Observer> list = new ArrayList<>();
	
		// 注册观察者对象
		public void regist(Observer observer) {
			list.add(observer);
			System.out.println("regist an observer");
		}
	
		// 删除观察者对象
		public void detach(Observer observer) {
			list.remove(observer);
		}
	
		// 通知所有观察者
		public void nodifyObserver(String state) {
			for (Observer observer : list) {
				observer.update(state);
			}
		}

	}

**主题**

	public class ConcreteSubJect extends SubJect {
	
		// 通知观察者
		public void change(String state) {
			this.nodifyObserver(state);
		}
	}

**观察者接口**

	public interface Observer {
	
		// 更新接口
		void update(String state);
	}

**观察者**

	public class ConcreteObServer implements Observer {

		// 更新操作
		@Override
		public void update(String state) {
			System.out.println("观察者状态为:" + state);
		}

	}

**主函数**

    public class ObServer_SubJect_Main {
    	public static void main(String[] args) {
    		// 创建主题对象
    		ConcreteSubJect subJect = new ConcreteSubJect();
    		
    		// 创建观察者对象
    		ConcreteObServer obServer = new ConcreteObServer();
    		
    		// 将观察者对象注册到主题对象上
    		subJect.regist(obServer);
    		
    		// 改变主题状态
    		subJect.change("1");
    	}
    }

