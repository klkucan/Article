###1.
实际上Cocos2d-x的内存几乎是与OC的管理方式一致的，在OC的开发中习惯于自己管理对象内存的申请和释放，而在cCocos2d-x中因为对于大多数（至少我目前遇到的全部）类都有一个create方法，这个方法会将对象在创建完成后加入autorelease进行自动管理。
###2.
需要知道的是Cocos2d-x的Director类中的mainLoop方法：

```
void DisplayLinkDirector::mainLoop()
{

    if (_purgeDirectorInNextLoop)
    {
        _purgeDirectorInNextLoop = false;
        purgeDirector();
    }
    else if (_restartDirectorInNextLoop)
    {
        _restartDirectorInNextLoop = false;
        restartDirector();
    }
    else if (! _invalid)
    {
        drawScene();     
        // release the objects
        PoolManager::getInstance()->getCurrentPool()->clear();
    }
}
```
这个方法会在每一帧执行一次，注意最后一行，每次都会调用当前的AutoReleasePool的clear方法，而这个方法的核心是：opp~，每一帧都会释放一次对象。看到这里会比较奇怪，比如我在一个scene中创建一个layer，layer中添加一个sprite，这个sprite是用create方法创建的，创建完成的一瞬间引用计数为1，add到layer时为2，此时一帧结束后为1，那么难道下一帧结束后又会变成0进而释放吗？显然不会，从OC的角度看在layer->addChild后，这个spite就和layer关联到了一起，而layer和其相关的scene关联，直到这个layer或者scene被销毁时关联才结束，也就是sprite的引用计数才会变成0。但是不理解的是每帧的autorelease都会减少一次引用计数，那么为什么没有减去这个sprite的引用计数呢？难道UI树有什么神奇的地方。

```
std::vector<Ref*> releasings;
releasings.swap(_managedObjectArray);
for (const auto &obj : releasings)
{
    obj->release();
}
``` 