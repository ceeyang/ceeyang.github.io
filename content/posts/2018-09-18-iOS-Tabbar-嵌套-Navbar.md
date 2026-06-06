---
title: iOS UITabBarController 嵌套 UINavigationController
date: 2018-09-18
tags: [iOS]
description: iOS UITabBarController 嵌套 UINavigationController
---

# iOS UITabBarController 嵌套 UINavigationController

    本文处理两个问题:
    1. 自定义导航按钮后, 滑动返回手势失效问题
    2. TabBar 嵌套 NavigationBar 的 TabBar 显示隐藏错乱等问题

## 主要思路
1. 使用 UITabBarController 作为 Window 的 rootViewController
2. 自定义 UINavigationController 控制器, 全局控制 tabbar 的显示与隐藏
3. 添加自定义控制器为 UINavigationController 的 root 控制器
4. UITabBarController 添加每个 UINavigationController 控制器


## 直接上代码
自定义的 UITabBarController :

```swift
import UIKit

class HomeTabBarController: UITabBarController {

    override func viewDidLoad() {
        super.viewDidLoad()
        createChildController()
    }

    /// 通过自定义方法添加所有子控制器
    func createChildController() {
        addChildVC(childVc: HomeViewController(), title: "首页", image: "IMG_Home", selectedImage: "IMG_Home_Selected")
        addChildVC(childVc: MoreViewController(), title: "应用", image: "IMG_More", selectedImage: "IMG_More_Selected")
        addChildVC(childVc: MineViewController(), title: "我的", image: "IMG_Mine", selectedImage: "IMG_Mine_Selected")
    }

    /// 自定义添加子控制器
    func addChildVC(childVc: UIViewController, title:String, image: String, selectedImage: String) {
        let nav = HomeNavigationController(rootViewController: childVc)
        let normalImage = UIImage(named: image)
        let selectedImage = UIImage(named: selectedImage)
        nav.navigationItem.title = title
        nav.tabBarItem = tabbarItem(with: title, normalImage: normalImage!, selectedImage: selectedImage!)
        addChildViewController(nav)
    }

    /// 快捷创建 UITabBarItem
    func tabbarItem(with title: String, normalImage: UIImage, selectedImage: UIImage) -> UITabBarItem{
        let image = normalImage.withRenderingMode(.alwaysOriginal)
        let _selectedImage = selectedImage.withRenderingMode(.alwaysOriginal)
        let tabBarItem = UITabBarItem(title: title, image: image, selectedImage: _selectedImage)
        tabBarItem.setTitleTextAttributes([NSAttributedStringKey.foregroundColor: UIColor.mainColor], for: .selected)
        tabBarItem.setTitleTextAttributes([NSAttributedStringKey.foregroundColor: UIColor.tabbarNormalColor], for: .normal)
        return tabBarItem
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}

//让图片和文字在 iPad 下仍然保持上下排列
extension UITabBar {

    override open var traitCollection: UITraitCollection {
        if UIDevice.current.userInterfaceIdiom == .pad {
            return UITraitCollection(horizontalSizeClass: .compact)
        }
        return super.traitCollection
    }
}

```

自定义的 UINavigationController :

```swift

import UIKit

class HomeNavigationController: UINavigationController, UIGestureRecognizerDelegate {

    override func viewDidLoad() {
        super.viewDidLoad()

        /// 自定义返回按钮后返回手势失效
        /// 手动实现返回手势代理
        interactivePopGestureRecognizer?.delegate = self
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    override func pushViewController(_ viewController: UIViewController, animated: Bool) {


        /// 如果当前控制器的子控制器个数大于等于1, 说明推出的控制器为子控制器
        if childViewControllers.count > 0 {

            /// 自定义返回按钮
            let backBtn = UIButton()
            backBtn.setImage(UIImage(named: "IMG_Back"), for: .normal)
            backBtn.addTarget(self, action: #selector(backButtonDidClick), for: .touchUpInside)
            viewController.navigationItem.leftBarButtonItem = UIBarButtonItem(customView: backBtn)

            /// 子控制器隐藏  bottomBar
            viewController.hidesBottomBarWhenPushed = true
        }
        super.pushViewController(viewController, animated: animated)
    }

    /// 返回按钮点击事件
    @objc func backButtonDidClick() {
        popViewController(animated: true)
    }

    /// 滑动返回手势
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldReceive touch: UITouch) -> Bool {

        /// 判断是否需要滑动返回手势
        if gestureRecognizer == navigationController?.interactivePopGestureRecognizer {
            return navigationController!.childViewControllers.count > 1
        }

        /// 解决当前控制器为根控制器的时候, 返回手势卡屏的问题
        if childViewControllers.count == 1 {
            return false
        }

        return true
    }
}

```
