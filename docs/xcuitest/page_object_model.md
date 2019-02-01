---
layout: default
title: Page Object Model 
parent: XCUITest
nav_order: 7
tags: 
    - testing
    - xcode
---

# Page Object Model
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Run test on different devices




```swift
// File: TestCaseExtensions >  BaseTestCase.swift

import XCTest

class BaseTestCase: XCTestCase {
    override func setUp() {
        super.setUp()
        
        continueAfterFailure = false
    }

    override func tearDown() {
        super.tearDown()
    }
}

```

```swift
// File: Classes > BaseConfiguration > BaseConfiguration.swift

import Foundation
import XCTest

class BaseConfiguration {
    let app: XCUIApplication
    let testCase: BaseTestCase

    init(testCase: BaseTestCase) {
        app           = testCase.app
        self.testCase = testCase
    }
}

```

```swift
// File: Classes > AppConfiguration > AppConfiguration.swift

import Foundation
import XCTest

class AppConfiguration: BaseConfiguration {
    override init(testCase: SKKTestCase) {
        super.init(testCase: testCase)
    }
}

```

```swift
// File: TestCaseExtensions >  BaseTestCase.swift

import XCTest

class BaseTestCase: XCTestCase {
...
    lazy var appConfiguration: AppConfiguration = { [unowned self] in
        return AppConfiguration(testCase: self)
        }()
}

```