---
layout: default
title: UI Interactions
parent: XCUITest
nav_order: 4
tags: 
    - ui test
    - xcode

---

# Exists
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Tap on elements

### Tap Element
```swift
app.staticTexts["Apple"].tap()
```

### Wait For Element Before Tapping
```swift
// File: XCTestExtensions > XCUIElementExtensions.swift

import XCTest

extension XCUIElement {
    // Custom method to wait before tapping on element
    func tapElement() {
        waitForElementToBecomeHittable(timeout: .medium)
        tap()
    }
}
```

### Force Tap On Element
 Work around for XCUITest Bug to make certain elements tappable.
```swift
// File: XCTestExtensions > XCUIElementExtensions.swift

import XCTest

extension XCUIElement {

    func forceTap() {
        if isHittable {
            tap()
        } else {
            coordinate(withNormalizedOffset: CGVector(dx: 0.0, dy: 0.0)).tap()
        }
    }
}

```

## Swipe
```swift
let element = app.collectionViews["Apple"]

// Swipe up
element.swipeUp()

// Swipe Down
element.swipeDown()

// Swipe Right
element.swipeRight()

// Swipe Left
element.swipeLeft()
```
## Text Input

### Type
```swift
let textField = app.textFields["SearchBox"]

textField.tap()
textField.typeText("hello world")
```

### Clear And Enter Text
```swift
// File: XCTestExtensions > XCUIElementExtensions.swift

import XCTest

extension XCUIElement {

    func clearAndEnterText(_ text: String) {
        if value != nil {
            guard let stringValue = value as? String else {
                XCTFail("Tried to clear and enter text into a non string value")
                return
            }
            
            tap()
            let deleteString = stringValue.map { _ in XCUIKeyboardKey.delete.rawValue }.joined()
            typeText(deleteString)
        }

        typeText(text)
    }
}
```

### Enter Text

```swift
extension XCUIElement {
    // Workaround for keyboard issues when not visible
    func enterText(text: String) {
        tapElement()
        
        if !self.waitForExistence(timeout: .small) {
            tapElement()
        }

        let dismissKeyboardButton = XCUIKeyboardKey.enter

        dismissKeyboardButton.waitForExistence(timeout: .medium)
        
        clearAndEnterText(text)
        
        if dismissKeyboardButton.isHittable {
            dismissKeyboardButton.tapElement()
        }
        
        if elementType != .secureTextField {
            waitForElementsValueToMatch(predicate: text)
        }
    }
}
```


## Alerts

### Dismissing Alerts
```swift
app.alerts["Alert_Title"].buttons["ok"].tap()
```

### Dismissing Action Sheets
```swift
app.sheets["Sheet_Title"].buttons["ok"].tap()
```

### System alerts within the application
```swift
addUIInterruptionMonitor(withDescription: "Location Services") { (alert) -> Bool in
  alert.buttons["Allow"].tap()
  return true
}

app.buttons["Request Location"].tap()
app.activate()
```

## Sliders
```swift
app.sliders.element.adjust(toNormalizedSliderPosition: 0.7)
```

validate slider value approximately 
```swift
XCTAssertEqual(app.sliders["playerScrubberView"].normalizedSliderPosition, 0.7, accuracy: 0.1)
```

## Pickers
```swift
app.pickerWheels.element.adjust(toPickerWheelValue: "Picker Wheel Item Title")
```

### Date picker
```swift
// File: PageObjects > DatePickerScreen.swift

import XCTest

class DatePickerScreen: Page {

    private var dateField: XCUIElement { return app.textFields["Date"].firstMatch }

    // MARK: Date format - day-month-year i.e 13-March-2015
    func changePickerDate(_ date: String) {

        let newDateFormat = date.components(separatedBy: "-")
        let date  = newDateFormat[0]
        let month = newDateFormat[1]
        let year  = newDateFormat[2]

        dateField.tap()

        //Day
        app.datePickers.pickerWheels.element(boundBy: 0).adjust(toPickerWheelValue: date)

        // Month
        app.datePickers.pickerWheels.element(boundBy: 1).adjust(toPickerWheelValue: month)

        //Year
        app.datePickers.pickerWheels.element(boundBy: 2).adjust(toPickerWheelValue: year)

        // Dismiss dates picker
        dateField.forceTap()
    }
}
```


## Web Links
```swift
app.links["Tweet this"].tap()
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
You can add an function in **XCUIElement** extensions that takes the **Timeout** enum and outputs an **Bool**. You get the **rawValue** for **Timeout** and pass it to the native **waitForExistence** which returns whether the element exists.  

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
Many times functions returns a value, but sometimes you don’t care what the return value is – you might want to ignore it sometimes using **@discardableResult**.

In the **PageObject** class we need to create a **private** variable that has the element identifier, this makes it very convenient way of grouping **XCUIElements** by screens. The variable name should describe the element that it will referenced, also note **XCUIElements** should always be unique and try to add **.firstMatch** on to the end of an element in order for **XCUITest** to find the element quickly.

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

In the UI test we can create an instance of the page object class **AppPreviewScreen** then reference the function we require in order to validate the screen.

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