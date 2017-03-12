---
title: 3D-Touch的使用
date: 2017-02-28 23:17:28
tags: [实用技术]
categories: iOS
---

2015年9月10日，苹果在新品发布会上宣布了3D-Touch功能。这是一种立体触控技术，屏幕可感应不同的感压力度触控，很明显这种技术给APP的交互又增添了样式。Hann作为一名iOS开发者自然对这种新技术感到好奇，经过一番的资料搜集与学习，成功的将这种功能添加到正在开发的APP中。将3DTouch有关的知识点记录下来以供自己查阅，也给有需求的开发人员提供一些参考。


## 3DTouch的使用
开发者应该关注哪几部分？
*	外部启动标签（Home Screen Quick Actions）
*	peek、pop以及action的使用

<!-- more -->

## 外部启动标签（Home Screen Quick Actions）
外部启动标签给快速使用APP的某些特定功能提供了便利，按压具有3DTouch功能的APP图标可以激活标签。外部标签的添加上分为两种：静态添加标签、动态添加标签。静态标签是直接在项目的info.plist配置，动态标签需要在AppDelegate启动入口进行添加。下面是两种方式的添加，以及添加所涉及到的类、类型介绍。


### 静态标签
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_staticTag_img.png" width = "800",height = "240">

上图所示为静态标签在info.plist中的配置内容，即在plist中添加Array类型的`UIApplicationShortcutItems`以及字典类型的标签实体，下面对这些key和value进行介绍：

```objc
// info.plist

UIApplicationShortcutItems[{

UIApplicationShortcutItemIconFile：图标名，（如果使用系统图标key为UIApplicationShortcutItemIconType）

UIApplicationShortcutItemTitle：标签名称，（必须设置）

UIApplicationShortcutItemSubtitle：子标签名称，

UIApplicationShortcutItemType：标签的ID，（必须设置）

{UIApplicationShortcutItemUserInfo[{key:value}...](附加信息)

}...]
```


### 动态标签								

在添加动态标签之前我们先来了解几个相关的类：


`UIApplicationShortcutIcon`标签图标类
```objc	
@interface UIApplicationShortcutIcon : NSObject <NSCopying>

// 使用系统icon时用这个API创建UIApplicationShortcutIcon对象.
+ (instancetype)iconWithType:(UIApplicationShortcutIconType)type;

// 使用提供的图片自定义UIApplicationShortcutIcon对象，templateImageName为图片名.
+ (instancetype)iconWithTemplateImageName:(NSString *)templateImageName;
@end
```
`UIApplicationShortcutItem`标签类
```objc
@interface UIApplicationShortcutItem : NSObject <NSCopying, NSMutableCopying>
/**
  * type: 标签ID
  * localizedTitle: 标签名称
  * localizedSubtitle: 子标题名称
  * icon: 标签图标对象
  * userInfo: 附加信息
  */
- (instancetype)initWithType:(NSString *)type localizedTitle:(NSString *)localizedTitle localizedSubtitle:(nullable NSString *)localizedSubtitle icon:(nullable UIApplicationShortcutIcon *)icon userInfo:(nullable NSDictionary *)userInfo;

// 图标对象
@property (nullable, nonatomic, copy, readonly) UIApplicationShortcutIcon *icon;

// 对应静态标签的UIApplicationShortcutItemUserInfo.
@property (nullable, nonatomic, copy, readonly) NSDictionary<NSString *, id <NSSecureCoding>> *userInfo;
@end
```
在了解这两个类后我们来看看动态添加标签的代码：

```objc
// AppDelegate.m
// - (BOOL)application:didFinishLaunchingWithOptions:
// 动态添加快捷启动
    UIApplicationShortcutIcon *iconThree = [UIApplicationShortcutIcon iconWithTemplateImageName:@"showItemIconThree"];
    UIApplicationShortcutItem *itemThree = [[UIApplicationShortcutItem alloc] initWithType:@"shortcutTypeThree" localizedTitle:@"动态标签" localizedSubtitle:nil icon:iconThree userInfo:nil];
    [[UIApplication sharedApplication] setShortcutItems:@[itemThree]];
```
现在你可以使用静态、动态以及混合方式创建快捷启动标签，注意启动标签最多可以有4个。下面就是我们创建的三个标签：
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_iconTags_img.png" width="350",height="450">

### 标签的点击事件交互
	
```objc
// AppDelegate.m
- (void)application:(UIApplication *)application performActionForShortcutItem:(UIApplicationShortcutItem *)shortcutItem completionHandler:(void (^)(BOOL))completionHandler {
    
    UINavigationController *nav = (UINavigationController *)self.window.rootViewController;
    BSDetailViewController *detailVC = [[BSDetailViewController alloc] init];
    
    if ([shortcutItem.type isEqualToString:@"shortcutTypeOne"]) {
        detailVC.navTitle = @"静态标签一";
        
    } else if ([shortcutItem.type isEqualToString:@"shortcutTypeTwo"]) {
        detailVC.navTitle = @"静态标签二";
        
    } else if ([shortcutItem.type isEqualToString:@"shortcutTypeThree"]) {
        detailVC.navTitle = @"动态标签";
        
    }
    
    [nav pushViewController:detailVC animated:YES];
}
```
到此外部启动标签的创建以及处理事件都已经介绍完了，如果看文章比较枯燥请配合[demo源码](https://github.com/CoderHann/3DTouchDemo "3DTouchDemo")进行查看.
	
## peek、pop & action
	
在设置了具有peek、pop的页面跟用户交互的时候主要有三个阶段：①轻按：提示用户这个有预览功能，即被选中的空间会凸显出来周围变得模糊。②加大力度：进入peek预览模式，如果设置了该预览模式下的一些操作可以向上滑动显示actions。③更大力度：激活pop，该阶段一般是跳到指定的控制器。

轻按状态：
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_Press_img.png" width="250",height="500"alt="press">
peek预览：
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_peek_img.png" width="250",height="500"alt="peek">
actions：
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_actions_img.png" width="250",height="500"alt="action">
pop激活：
<img src="https://raw.githubusercontent.com/CoderHann/imageSource/master/3DTouch/3dtouch_pop_img.png" width="250",height="500"alt="pop">



要达到上述效果，在开发中都需要做什么呢？下面一步一步揭晓！
### peek & pop

首先我们将能触发peek、pop功能的控制器遵守UIViewControllerPreviewingDelegate协议

```objc
// BSTableViewController.m
@interface BSTableViewController()<UIViewControllerPreviewingDelegate,BSDetailViewControllerDelegate>
@end
```
然后注册该控制器作为peek和pop预览的代理以及提供预览视图容器
```objc
// BSTableViewController.m
// 重要
[self registerForPreviewingWithDelegate:self sourceView:self.view];
```
实现代理方法：
```objc
// BSTableViewController.m
// UIViewControllerPreviewingDelegate
- (UIViewController *)previewingContext:(id<UIViewControllerPreviewing>)previewingContext viewControllerForLocation:(CGPoint)location {
    self.selectedCell = [self searchCellWithPoint:location];
    previewingContext.sourceRect = self.selectedCell.frame;
    
    NSLog(@"peek");
    BSDetailViewController *detailVC = [[BSDetailViewController alloc] init];
    detailVC.delegate = self;
    detailVC.navTitle = self.selectedCell.textLabel.text;
    return detailVC;
}

- (void)previewingContext:(id<UIViewControllerPreviewing>)previewingContext commitViewController:(UIViewController *)viewControllerToCommit {
    NSLog(@"pop");
    [self tableView:self.tableView didSelectRowAtIndexPath:[self.tableView indexPathForCell:self.selectedCell]];
}
```

### actions
上面的一步操作使我们的主界面`BSTableViewController`有了轻按、peek以及pop的功能，还有个是action，这个action不属于主界面而是详情界面`BSDetailViewController`的事件,在[demo](https://github.com/CoderHann/3DTouchDemo "3DTouchDemo")里面，详情将事件代理了主界面去处理事情。
`BSDetailViewController`生成事件方法：
```objc
// BSDetailViewController.m
- (NSArray<id<UIPreviewActionItem>> *)previewActionItems {
    //
    UIPreviewAction *action1 = [UIPreviewAction actionWithTitle:@"删除" style:UIPreviewActionStyleDefault handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        if ([self.delegate respondsToSelector:@selector(detailViewController:DidSelectedDeleteItem:)]) {
            [self.delegate detailViewController:self DidSelectedDeleteItem:self.navTitle];
        }
    }];
    //
    UIPreviewAction *action2 = [UIPreviewAction actionWithTitle:@"返回" style:UIPreviewActionStyleDefault handler:^(UIPreviewAction * _Nonnull action, UIViewController * _Nonnull previewViewController) {
        if ([self.delegate respondsToSelector:@selector(detailViewControllerDidSelectedBackItem:)]) {
            [self.delegate detailViewControllerDidSelectedBackItem:self];
        }
    }];
    
    NSArray *actions = @[action1,action2];
    
    return actions;
}
```

好了，上面将peek、pop以及action也介绍完了，看到这你应该对3DTouch的使用有了一定的认识，还想更深入的学习有关3Dtouch请查看官方文档，想看本博客demo源码的点击我[CoderHann](https://github.com/CoderHann/3DTouchDemo "3DTouchDemo")进行查看。


