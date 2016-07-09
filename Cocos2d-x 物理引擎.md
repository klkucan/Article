###1.基本流程
- 创建scene时需要注意，使用
`auto scene = Scene::createWithPhysics();`
同时你需要实现一个onEnter()方法。

- 对于UI元素来讲在创建时并没有什么特别的变化，但是需要绑定一个物理对象（body）,这个很像是Unity3D中添加碰撞体和rigibody的过程。

```
    auto ball = Sprite::create("x.png"); 
    auto body = PhysicsBody::createBox(ball->getContentSize());
    ball->setPhysicsBody(body);
    addChild(ball);
```