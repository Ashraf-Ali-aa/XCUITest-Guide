---
layout: default
title: Testing on different devices 
parent: XCUITest
nav_order: 6
tags: 
    - testing
    - xcode
---

# Testing on different devices
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Run test on different devices

```swift
import Foundation
import XCTest

public extension XCUIApplication {
    
    /// Check if application is running on simulator
    public var isRunningOnSimulator: Bool {
        #if targetEnvironment(simulator)
            return true
        #else
            return false
        #endif
    }

    public var isRunningOnRealDevice: Bool {
        return !isRunningOnSimulator
    }

    public var isRunningOnTablet: Bool {
        return UIDevice.current.userInterfaceIdiom == .pad
    }

    public var isRunningOnPhone: Bool {
        return UIDevice.current.userInterfaceIdiom == .phone
    }
}
```

```swift
// File: TestCaseExtensions > DeviceTypeProtocol.swift

protocol TestOnAnyDevice {}
protocol TestOnPhone {}
protocol TestOnTablet {}
protocol TestOnRealDevice {}
protocol TestOnSimulator {}

```

```swift
// File: TestCaseExtensions >  BaseTestCase.swift

import XCTest

class BaseTestCase: XCTestCase {

    // Check if tests are compatible with the current device
    override func perform(_ run: XCTestRun) {
        let test = run.test

        let testOnPhone      = test is TestOnPhone
        let testOnRealDevice = test is TestOnRealDevice
        let testOnSimulator  = test is TestOnSimulator
        let testOnTablet     = test is TestOnTablet

        if (testOnRealDevice && app.isRunningOnSimulator) || (testOnSimulator && app.isRunningOnRealDevice) {
            return reasonTestWasSkipped(test.name, message: "Not supported for the current hardware type")
        }

        if (testOnPhone && app.isRunningOnTablet) || (testOnTablet && app.isRunningOnPhone) {
            return reasonTestWasSkipped(test.name, message: "Not supported for the current device type")
        }

        super.perform(run)
    }

    private func reasonTestWasSkipped(_ testName: String, message: String) {
        let messageForSkipedTest = """
        -------- Test Skipped --------
        \(testName)       [\(message)]
        ------------------------------
        """

        return print(messageForSkipedTest)
    }
}

```

```swift
// File: UITest >  ExampleTest.swift

import XCTest

class ExampleTest: BaseTestCase, TestOnPhone {

    func testWaitForElement() {
        ...
    }
}

```

```swift
// File: UITest >  ExampleTestTablet.swift

import XCTest

class ExampleTestTwo: BaseTestCase, TestOnTablet, TestOnSimulator {

    func testWaitForElement() {
        ...
    }
}

```