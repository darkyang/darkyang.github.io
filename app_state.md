### app状态切换时如何保存页面当前状态并在app恢复时恢复至相应状态

当修改view controller 的 restorationIdentifier时， 需要确保该view controller的所有父控制器也被标记，否则当父控制器未被标记时，所有子控制器的保存状态标记都会被无视。（原因： UIkit从根控制器开始按照页面层级遍历状态保存标记， 当遇到层级中保存状态标记为nil的控制器时，它的子控制以及弹出的控制器均会被无视）


### 选择有效的状态保存标记


### 排除不想保存状态的view controller


### 保存view controllers's view 的状态
* **给restorationIdentifier赋以一个可用的非空值**
* **使用这个view**
* **如果是table views或者 collection views，需要填充datasource**


### 载入时恢复控制器状态
* 如果页面控制器包含保存状态的类，UIKit会从这个类中获取view controller
* 如果页面控制器不包含保存状态的类，UIKit会从app delegate获取view controller
* 如果页面控制器的保存的状态正确，UIKit会使用相应的restoration paths
* 如果页面控制器通过storyboard文件保存，UIKit使用保存的故事版的信息去定为并创建控制器

```objc

+(UIViewController*) viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder 
{

   MyViewController* vc;
   
   UIStoryboard* sb = [coder decodeObjectForKey:UIStateRestorationViewControllerStoryboardKey];
   
   if (sb) {
   
      vc = (PushViewController*)[sb instantiateViewControllerWithIdentifier:@"MyViewController"];
      
      vc.restorationIdentifier = [identifierComponents lastObject];
      
      vc.restorationClass = [MyViewController class];
   }
    return vc;
  
}
```


### 序列化／反序列化控制器状态
app退出保存状态期间， UIKit调用** encodeRestorableStateWithCoder: ** 方法将要保存的状态序列化。

app恢复状态期间，UIKit调用**decodeRestorableStateWithCoder: **方法反序列化解析保存的状态。

通常需要保存/恢复的信息：

*  数据对象的引用
*  如果是容器控制器，保存子控制的引用
*  当前选择的控制器信息
*  用户配置的view， 当前页面的配置信息

```objc

-(void)encodeRestorableStateWithCoder:(NSCoder *)coder {
   [super encodeRestorableStateWithCoder:coder];
 
   [coder encodeInt:self.number forKey:MyViewControllerNumber];
}
 
-(void)decodeRestorableStateWithCoder:(NSCoder *)coder {
   [super decodeRestorableStateWithCoder:coder];
 
   self.number = [coder decodeIntForKey:MyViewControllerNumber];
   
}
```

