title: 使用Physicals代替Box2D和chipmunk
date: 2014/10/1 13:00:00
tags: 
- cocos2d-x
---

## 1.概述

游戏中模拟真实的世界是个比较麻烦的事情，通常这种事情都是交给物理引擎来做。首屈一指的是`Box2D`了，它几乎能模拟所有的物理效果。而`chipmunk`则是个更轻量的引擎，能够满足简单的物理需求，比如最常用的的碰撞检测等。这些引擎在使用的过程中有个令人讨厌的地方，它们参数太多了。通常为了初始化一个简单的场景要写很多代码。在cocos2d-x 3.0版本中，出现了一个新类族——`physicals`。它将Box2D或者chipmunk做了一层封装，使我们的上层调用有更友好的接口。它通过宏来切换使用哪种物理引擎，`目前的版本只有chipmunk的实现，Box2D的实现没有写`，所以手动将宏切换的话是不行的。另外，当前版本还是有bug的，下面会提到.

## 2.原理分析
在这个版本中，物理世界的概念被加入到Scene中，即当创建一个场景时，就可以指定这个场景是否使用物理引擎。相对应的，每一个Sprite中也有body的概念。可以直接将body关联到Sprite上。Listener当然也不需要再弄一套东西来监听，只要注册到场景中就可以了。

## 3.创建场景

	Scene* HelloWorld::createScene()  
	{
		// 'scene' is an autorelease object  
		auto scene = Scene::createWithPhysics();  
      	scene->getPhysicsWorld()->setDebugDraw(true);  
     	// 'layer' is an autorelease object  
       	auto layer = HelloWorld::create();  
      	// add layer as a child to scene  
     	scene->addChild(layer);   
      	// return the scene  
     	return scene;  
    }
更改create，创建一个支持物理的世界，打开debugDraw。两行就可以搞定了。

接下来，我们要将这个World传到Layer中。所以我们在HelloWorld类中加入一个函数。将这个world存起来。 

	//...
		void setPhyWorld(PhysicsWorld* world){m_world=world;}
	private:
		PythicsWorld* m_world
	//...
	
同时在creatScene创建layer完成后，将这个值设定上。
	
	//...
		auto layer = HellowWorld::create();
		layere->setPhyWorld(scene->getPhysicsWorld());
	//...
	
## 4.创建边界
接下来，我们着手创建一个边界。我们可以方便的使用PhysicalsBody的create方法创建自己想要的物体。 

	bool helloWorld::init()
	{
		//...
		auto edgeSp = Sprite::create();  
		auto body = PhysicsBody::createEdgeBox(visibleSize,3);  
		edgeSp->setPosition(Point(visibleSize.width/2,visibleSize.height/2));  
		edgeSp->setPhysicsBody(body);  
		this->addChild(edgeSp);  
		edgeSp->setTag(0);  	
		//...
		return true;	
	}

## 5.添加元素
关联body与sprite从未如此简单，我们只需创建一个body，创建一个sprite然后将body设置为sprite的body即可。 

	auto body = PhysicsBody::createBox(Size(80, 40));  
	sp->setPhysicsBody(body); 

### 3.0版bug
 在这其中，当前版本的cocos2d-x 3.0有一个小问题。关联的时候，并未将body相应的owner设置为对应的sprite，我们需要修改sprite.cpp中的`setPhysicsBody`这个函数。增加最后一行。

	void Sprite::setPhysicsBody(PhysicsBody* body)  
	{  
		_physicsBody = body;  
		_physicsBody->retain();  
		_physicsBody->setPosition(getPosition());  
		_physicsBody->setRotation(getRotation());  
		_physicsBody->_owner = this;  
	}  
## 6.碰撞检测
碰撞检测的回调是在world中注册函数来实现的。首先我们在HelloWorld中声明一个变量。并重写OnEnter方法。  
声明
  
	PhysicsContactListener m_listener;  
实现

	void HelloWorld::onEnter()
	{
		Layer::onEnter(); 
		
		m_listener.onContactBegin = [=](const PhysicsContact& contact)  
		{
			auto cnt = const_cast<PhysicsContact*>(&contact);  
			
			auto sp = cnt->getShapeA()->getBody()->getOwner();  
			int tag = sp->getTag();
			if(tag == 1)
			{
				//...
			} 
			
			sp = cnt->getShapeB()->getBody()->getOwner(); 
			tag = sp->getTag(); 
			if(tag == 1)
			{
				//...
			} 
			//...
			return true;
		};
		
		m_world->registerContactListener(&m_listener);
	}
	
其中，我们将listener的onContactBegin方法重写。并通过`shape->body->owner`的方式来取到sprite。更改它的显示。最后将listener注册到m_world中。

## 7.总结
通过创建一个支持Physicals的场景，来创建物理系统。将body创建出来，并调用sprite的`setPhysicsBody`来为一个sprite设定body。通过`PhysicsContactListener`来创建一个Listener并通过`registerContactListener`将其注册，来处理碰撞。
	
