---
title: "맥용 메뉴 바 앱 개발하기 - 00. 팝오버 생성"
date: 2018-07-23 18:43:20 +0900
categories: Swift
tags: [macOS, popover, swift]
layout: post
comments: true
pinned: true
---

## Intro
전부터 macOS에서의 상단 메뉴 바에서 동작하는 앱을 만들어보고 싶다는 생각을 했습니다. 메뉴 바 앱은 접근성도 쉽고, 주로 생산성을 향상시켜주는 앱이 많아서 그런지 사용자와 계속 함께한다는 느낌도 있어서요. 계속 생각만 하고 미뤄오다가, 요즘 여유 시간이 생겨 연습삼아 배우면서 개발을 진행했습니다.

이런 생각으로 [Local Port Scanner](https://github.com/rajephon/LocalPortScanner)라는 이름의 앱을 만들었습니다. 기능은 간단합니다. `lsof` 명령어를 통해 현재 LISTENING 상태로 열려있는 TCP 포트 리스트를 보여줍니다.  
앱을 만들면서 여러 기능도 사용되었습니다. 권한 상승을 얻어내서 커맨드 라인 명령어를 앱에서 수행하는 법이라거나, 사용자 로그인 시 앱이 자동 실행되게 하는 방법 등이 있습니다.  
이와 관련된 것들을 이번 주제의 블로그 포스팅을 통해 순차적으로 기록해 나갈 예정입니다.

## 프로젝트 생성 및 아이콘 추가
![CreateProject]({{ site.url }}/assets/images/MenuBarApp-00/00.png)  
Xcode에서 Cocoa App으로 프로젝트를 생성해주세요. `Use Storyboards`를 제외한 `Include UI Tests` 등은 모두 체크를 해제합니다.

그리고 `AppDelegate.swift`에서 `statusItem`를 정의합니다.
```swift
let statusItem = NSStatusBar.system.statusItem(withLength:NSStatusItem.squareLength)
```
`statusItem`은 이름 그대로 `Status Item`입니다. macOS의 상단 메뉴 바에서 앱 아이콘을 표시하고 누를 경우 창을 표시하는 버튼 역할을 합니다.  

![Assets.xcassets]({{ site.url }}/assets/images/MenuBarApp-00/01.png)  
다음으로 [버튼 이미지](https://github.com/rajephon/LocalPortScanner/blob/master/PortScanner/Assets.xcassets/StatusBarButtonImage.imageset/StatusBarButtonImage%402x.png)를 추가합니다. 버튼 이미지는 32x32의 크기로 만들었습니다.  
이미지 이름을 `StatusBarButtonImage@2x.png`로 수정하여 `Assets.xcassets`로 드래그하여 추가합니다. 그리고 우측 `Image Set`의 `Render As`속성의 값을 `Template Image`로 설정합니다.

```swift
let statusItem = NSStatusBar.system.statusItem(withLength:NSStatusItem.squareLength)

func applicationDidFinishLaunching(_ aNotification: Notification) {
    if let button = statusItem.button {
        button.image = NSImage(named:NSImage.Name("StatusBarButtonImage"))
    }
}
```
다음으로 `AppDelegate.swift`에서 `statusItem`을 만들고, `statusItem.button`에 해당 이미지를 지정해줍니다.

![Manu Bar]({{ site.url }}/assets/images/MenuBarApp-00/02.png)  

그리고 Xcode에서 `Build & Run`을 할 경우, 다음과 같이 상단 메뉴 바에 우리가 추가한 아이콘이 들어가있는 것을 확인하실 수 있습니다.  
물론 아직, 동작 기능을 추가하지 않았기 때문에 눌러도 반응이 없습니다. 

## Dock Icon 비활성화 및 Main Window 제거

![Manu Bar]({{ site.url }}/assets/images/MenuBarApp-00/03.png)  

Xcode 프로젝트의 `Info.plist`에 `Application is agent (UIElement)`라는 Key를 추가합니다. 입력이 번거로우실 경우 `LSUIElement`라고 입력하면 자동으로 변환이 될 것입니다. 그리고 Value를 Boolean `YES`로 설정합니다.

![Story Board]({{ site.url }}/assets/images/MenuBarApp-00/04.png)  
다음으로 `Main.storyboard`를 엽니다. `Window Controller Scene`을 선택 - 삭제합니다.  
<u>`View Controller Scene`은 그대로 둡니다.</u> 이것은 추후에 활용할 것입니다.

다시 Xcode에서 `Build & Run`을 할 경우, Dock icon과 Main Window 없이 단지 Menu bar에 Status Item만 나타날 것입니다.

여기까지 잘 따라오셨다면 성공적인 상황이라 볼 수 있습니다 :)

## Popover 추가
이제부터, Status Item을 클릭할 경우의 팝오버 뷰가 나타나도록 새로 뷰 컨트롤러를 추가해봅니다. Xcode에서 `File - new - File...`을 클릭해서 새로운 파일을 생성합니다. 생성 항목은 `Source`의 `Cocos Class`를 선택합니다.
![New View Conroller]({{ site.url }}/assets/images/MenuBarApp-00/05.png)  

이름은 `SampleViewController`로 했습니다.  
`Subclass of`는 `NSViewController`를 지정하여 상속합니다.  
그리고 하단의 `Also create XIB file for user interface`는 <u>선택 해제</u>합니다.  
언어는 `Swift`를 사용합니다.  

`Next`를 클릭하고, 적당한 위치(`AppDelegate.swift`와 같은 위치를 권장합니다.)를 지정하여 파일을 생성합니다.

![Storyboard 2]({{ site.url }}/assets/images/MenuBarApp-00/06.png)  

`Main.storyboard`선택, 이전에 남겨뒀던 `View Controller Scene`의 `View Controller`를 선택합니다.  
`Identity Inspector`에서 Class 이름을 방금 새로만든 Class name으로 설정합니다. 저는 `SampleViewController`라고 이름을 지었으므로 `SampleViewController`라고 설정합니다. `Storyboard ID`의 값도 똑같이 지정합니다.

다음의 코드를 `SampleViewController.swift`의 끝에 추가합니다.
```swift
extension SampleViewController {
    // MARK: Storyboard instantiation
    static func freshController() -> SampleViewController {
        //1.
        let storyboard = NSStoryboard(name: NSStoryboard.Name(rawValue: "Main"), bundle: nil)
        //2.
        let identifier = NSStoryboard.SceneIdentifier(rawValue: "SampleViewController")
        //3.
        guard let viewcontroller = storyboard.instantiateController(withIdentifier: identifier) as? SampleViewController else {
            fatalError("Why cant i find SampleViewController? - Check Main.storyboard")
        }
        return viewcontroller
    }
}
```
이 코드는 다음과 같은 동작을 합니다.  
1. `Main.storyboard`를 가져옵니다.
2. `Scene Identifier`를 생성합니다. (Storyboard에서 이름에 해당하는 View Controller를 찾음. 이전에 지정한 `Storyboard ID`와 일치해야 합니다.)
3. `SampleViewController`를 인스턴스화 해서 return it.

혹시 이름을 다른 것으로 지정하셨을 경우, 해당하는 이름에 맞춰 수정해주시면 됩니다.
만약 이름이 일치하지 않을 경우 오류가 나타날 것입니다. 큰 문제가 아니기에 금방 원인을 찾으실 수 있을 것이라 생각합니다.

다시 `AppDelegate.swift`으로 돌아가 `AppDelegate` 클래스 안에 다음 속성 선언을 추가합니다.
```swift
let popover = NSPopover()
```
그 다음, `applicationDidFinishLaunching(_:)`에 코드를 추가합니다.  
```swift
func applicationDidFinishLaunching(_ aNotification: Notification) {
    // Insert code here to initialize your application
    if let button = statusItem.button {
        button.image = NSImage(named:NSImage.Name("StatusBarButtonImage"))
        button.action = #selector(togglePopover(_:))
    }
    popover.contentViewController = SampleViewController.freshController()
}
```
`button.action = #selector(togglePopover(_:))`으로 버튼 액션에 셀렉터를 지정하여 버튼이 클릭될 경우 `togglePopover(_:)` 메소드가 동작하도록 했습니다.  
또, popover 내부 콘텐츠의 ViewController를 `SampleViewController`로 세팅합니다.  
다음 코드를 `AppDelegate.swift`에 계속 추가합니다.

```swift
@objc func togglePopover(_ sender: Any?) {
  if popover.isShown {
    closePopover(sender: sender)
  } else {
    showPopover(sender: sender)
  }
}

func showPopover(sender: Any?) {
  if let button = statusItem.button {
    popover.show(relativeTo: button.bounds, of: button, preferredEdge: NSRectEdge.minY)
  }
}

func closePopover(sender: Any?) {
  popover.performClose(sender)
}
```

`showPopover()`는 사용자에게 popover를 보여줍니다. `status item`의 아이콘 밑에 말풍선 모양으로 나타납니다. `closePopover()`는 반대로 popover를 감춥니다. 그리고 `togglePopover()`는 popover가 보여지고 있을 시 감추고, 감춰져 있을 시 보여주도록 동작합니다.

![popover]({{ site.url }}/assets/images/MenuBarApp-00/07.png)  

Xcode에서 `build & run`을 수행 후 status item을 클릭시 다음과 같이 빈 팝업창이 나타날 경우 성공입니다. YEAH! 🎉

여기까지 프로젝트를 생성하고 status item 추가, 기본 세팅, 팝오버 생성까지 진행하였고, 다음 포스트에서는 `SMJobBless`와 `NSXPCConnection`를 이용해서 권한을 상승받아 콘솔 명령을 동작하는 작업을 설명해보겠습니다.

## Reference
[https://www.raywenderlich.com/165853/menus-popovers-menu-bar-apps-macos](https://www.raywenderlich.com/165853/menus-popovers-menu-bar-apps-macos)