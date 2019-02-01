---
layout: default
title: Testing if elements exist
parent: XCUITest
nav_order: 3
tags: 
    - exists
    - ui test
    - xcode

---

# Testing if an elements exist
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Assert an Element Exists

### Exists
```swift
XCTAssert(app.staticTexts["Apple"].exists)
```

### Wait For Existence
```swift
XCTAssert(app.staticTexts["Apple"].waitForExistence(timeout: 2))
```

### Wait For Element To Become Hittable
Due to the way `XCUITest` works there might be instances where an element exists but the element is not tappable an example of this is an element exists but its off screen, `waitForElementToBecomeHittable` waits until you can interact with the element. 
```swift
// File: XCTest > XCUIElement > XCUIElement.swift
import XCTest

extension XCUIElement {

    @discardableResult
    func waitForElementToBecomeHittable(timeout: Timeout) -> Bool {
        return waitForExistence(timeout: timeout) && isHittable
    }
}

```

### Wait For Element To Become Unhittable
Sometimes we need to validate if an element has disappeared rather than if it has appeared, within `waitForElementToBecomeUnhittable` we call `NSPredicate` to check if element does not `exists` and also is not `hittable`, Then we wait for a given period before we return the `Boolean` statement.  
```swift
// File: XCTest > XCUIElement > XCUIElement.swift
import XCTest

extension XCUIElement {

    @discardableResult
    func waitForElementToBecomeUnhittable(timeout: Timeout) -> Bool {
        let predicate   = NSPredicate(format: "exists == false && isHittable == false")
        let expectation = XCTNSPredicateExpectation(predicate: predicate, object: self)

        let result = XCTWaiter().wait(for: [ expectation ], timeout: timeout.rawValue)

        return result == .completed
    }
}
```

### Wait For Elements Value To Match
```swift
// File: XCTestExtensions > XCUIElementExtensions.swift

import XCTest

extension XCUIElement {

    @discardableResult
    func waitForElementsValueToMatch(predicate: String, timeOut: Timeout = .medium) -> Bool {
        let predicate   = NSPredicate(format: "value BEGINSWITH '\(predicate)'", argumentArray: nil)
        let expectation = XCTNSPredicateExpectation(predicate: predicate, object: self)

        let result = XCTWaiter().wait(for: [ expectation ], timeout: timeout.rawValue)

        return result == .completed
    }
}
```


## Case Study
XCUItest allows you to set custom wait times for XCUIElements but having predefined timeouts comes in
handy, this can be accomplished by creating an enum that contain different timeouts then this can be used instead of hardcoded numbers. 

```swift
// File: Data > TimeOut.swift
/// Data file
enum Timeout: TimeInterval {
    case extraSmall = 1
    case small      = 5
    case medium     = 10
    case large      = 20
}
```
You can add an function in `XCUIElement` extensions that takes the `Timeout` enum and outputs an `Bool`. You get the `rawValue` for `Timeout` and pass it to the native `waitForExistence` which returns whether the element exists.  

```swift
// File: XCTest > XCUIElement > XCUIElement.swift
import XCTest

extension XCUIElement {

    @discardableResult
    func waitForExistence(timeout: Timeout) -> Bool {
        return waitForExistence(timeout: timeout.rawValue)
    }
}
```
Many times functions returns a value, but sometimes you don’t care what the return value is – you might want to ignore it sometimes using `@discardableResult`.

In the `PageObject` class we need to create a `private` variable that has the element identifier, this makes it very convenient way of grouping `XCUIElements` by screens. The variable name should describe the element that it will referenced, also note `XCUIElements` should always be unique and try to add `.firstMatch` on to the end of an element in order for `XCUITest` to find the element quickly.

You can create a method that uses the variable to check if the element exists and the method then can be accessed by other classes, using this approach we remove code duplication but also it makes it easier to manage the UI tests.   

```swift
// File: PageObjects >  AppPreviewScreen.swift

import XCTest

class AppPreviewScreen: Page {
    private var okButton: XCUIElement { return app.buttons["ok"].firstMatch }

    func isOkButtonExists() -> Bool {
        okButton.waitForExistence(timeout: .small)
    }
}

```

In the UI test we can create an instance of the page object class `AppPreviewScreen` then reference the function we require in order to validate the screen.

```swift
// File: UITest >  ExampleTest.swift

import XCTest

class ExampleTest: XCTestCase {
    override func setUp() {
        super.setUp()
        
        continueAfterFailure = false
        var appPreview: AppPreviewScreen = AppPreviewScreen(testCase: self)
    }

    override func tearDown() {
        super.tearDown()
    }

    func testWaitForElement() {
        XCTAssert(appPreview.isOkButtonExists())
    }
}

```