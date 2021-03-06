# SDMagicHook

[中文文档](./README_CN.md) [原理解析](https://mp.weixin.qq.com/s/wxigL1Clem1dR8Nkt8LLMw)

A safe and influence-restricted method hooking for both Objective-C and Swift.

## Improvement

Classical method swizzling with method_exchangeImplementations is quite simple, but it has a lot of limitations and defects:

- You have to add a new method in a class category everytime when you want to swizzle a method.
- Different method implementation with same selector in different category will case method conflicts.
- The method swizzling will affect all the instances of the target class，however in most cases it is not necessary but even has side effects.

Now SDMagicHook will solve the problems mentioned above.

## Usage

Example for hooking CALayer's `setBackgroundColor:` method to find the one who secretly changed the background color of the view:

```objc
[self.view.layer hookMethod:@selector(setBackgroundColor:) impBlock:^(CALayer *layer, CGColorRef color){
    [layer callOriginalMethodInBlock:^{
        [layer setBackgroundColor:color];
    }];
    NSLog(@"%@", [NSString stringWithFormat:@"AHA! Catch it! Here are the clues.\n\n%@", [NSThread callStackSymbols]]);
}];
```

Example for hooking UIButton's `pointInside:withEvent:` method to expand the hot area:

```objc
- (void)expand:(Boolean)yn {

    if (yn) {
        __weak typeof(self) weakSelf = self;
        _hookID = [_button hookMethod:@selector(pointInside:withEvent:) impBlock:^(UIView *v, CGPoint p, UIEvent *e){
            __block BOOL res = false;
            [v callOriginalMethodInBlock:^{
                res = [v pointInside:p withEvent:e];
            }];
            if (res) return YES;

            return [weakSelf pointCheck:p view:v];
        }];
    } else {
        [_button removeHook:@selector(pointInside:withEvent:) strId:_hookID];
    }
}
```

Example for hooking class method:

```objc
- (void)hookClassMethod {
    [[DemoVC3 class] hookMethod:@selector(testClassMethod) key:&testClassMethodTag impBlock:^(id cls){
        [cls callOriginalMethodInBlock:^{
            [[DemoVC3 class] testClassMethod];
        }];
        printf(">> hooked testClassMethod");
    }];
}

+ (void)testClassMethod {
    printf(">> %s", sel_getName(_cmd));
}

- (void)dealloc {
    [[DemoVC3 class] removeHook:@selector(testClassMethod) key:&testClassMethodTag];
}
```
Example for observe any instance's dealloc event. 
```objc
[[NSObject new] addObjectDeallocCallbackBlock:^{
    printf("Will dealloc...");
}];
```

Example for hooking UIViewController's `viewDidDisappear:` method with Swift:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    let imp: @convention(block) (UIViewController, Bool) -> Void = { (vc, flag) in
        vc.callOriginalMethod {
            vc.viewDidDisappear(flag)
        }
        print(vc)
    }
    rootVC = navigationController?.children.first
    hookID = rootVC?.hookMethod(#selector(UIViewController.viewDidDisappear(_:)), impBlock: imp)
}
```

## License

MIT License.




