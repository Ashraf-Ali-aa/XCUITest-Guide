---
layout: default
title: UI Preidcate
parent: XCUITest
nav_order: 8
tags: 
    - testing
    - xcode
---

# Xcode
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Easy Predicate
When working with UI tests, there are times you may need to locate elements that are not directly accessible or filter a list of elements dynamically. EasyPredicate is a wrapper around NSPredicate designed to simplify and streamline this process.

With EasyPredicate, you can define element queries in a declarative manner, making your test code more readable and maintainable. Additionally, it includes XCUIElement helper methods that support chaining, enabling a more convenient and intuitive way to construct complex queries.

Key Features:

Declarative Syntax: Simplifies filtering and querying elements with a clear and concise syntax.
NSPredicate Integration: Leverages the power of NSPredicate for precise element matching.
Chaining Support: Provides helper methods to build queries through a fluent API, making complex queries easier to understand.
By incorporating EasyPredicate into your UI testing framework, you can reduce boilerplate code and improve the clarity of your test cases. 

```swift
let item = app.buttons.containing(predicate: .string("example button"))
```

### Implementation

### Predicate.swift
```swift
//
//  Predicate.swift
//  DemoUITests
//
//  Created by Daniel Yang on 2019/7/2.
//  Copyright © 2019 Daniel Yang. All rights reserved.
//

import Foundation
import XCTest

/// infix string of Regular expression
public enum Comparison: RawRepresentable {
    
    // MARK: - Cases
    case equals
    case notEqual
    case beginsWith
    case contains
    case endsWith
    case like
    case matches
    case other(String)
    
    // MARK: - RawRepresentable
    public var rawValue: String {
        switch self {
        case .equals: return "="
        case .notEqual: return "!="
        case .beginsWith: return "BEGINSWITH"
        case .contains: return "CONTAINS"
        case .endsWith: return "ENDSWITH"
        case .like: return "LIKE"
        case .matches: return "MATCHES"
        case .other(let comparisonOperator): return comparisonOperator
        }
    }
    
    /// default as .other case
    ///
    /// - Parameter rawValue: regular string
    public init(rawValue: String) {
        switch rawValue {
        case "=": self = .equals
        case "!=": self = .notEqual
        case "BEGINSWITH": self = .beginsWith
        case "CONTAINS": self = .contains
        case "ENDSWITH": self = .endsWith
        case "LIKE": self = .like
        case "MATCHES": self = .matches
        default: self = .other(rawValue)
        }
    }
}

/// PredicateKey
public enum PredicateKey {
    public enum bool: String    { case exists, isEnabled, isHittable, isSelected }
    public enum string: String  { case identifier, label }
    public enum type: String    { case elementType }
}

/**
 The rawValue of **EasyPredicate**
 */
public enum PredicateRawValue: RawRepresentable {
    
    // MARK: - Cases
    case bool(key: PredicateKey.bool, comparison: Comparison, value: Bool)
    case string(key: PredicateKey.string, comparison: Comparison, value: String)
    case type(value: XCUIElement.ElementType)
    case custom(regular: String)
    
    // MARK: - RawRepresentable
    /// default as custom case
    ///
    /// - Parameter rawValue: regular string
    public init?(rawValue: String) {
        self = .custom(regular: rawValue)
    }
    
    /// convert to regularString
    public var rawValue: String {
        switch self {
        case .bool(let key, let comparison, let value):
            return "\(key.rawValue) \(comparison.rawValue) \(value ? "true" : "false")"
        case .string(let key, let comparison, let value):
            return "\(key.rawValue) \(comparison.rawValue) '\(value)'"
        case .type(let value):
            return "\(PredicateKey.type.elementType.rawValue) \(Comparison.equals.rawValue) \(value.rawValue)"
        case .custom(let regular):
            return regular
        }
    }
}

/**
 MARK: - EasyPredicate
 
 Although `NSPredicate` is powerfull but the developer interface is not good enough,
 We can try to convert the hard code style into the object-oriented style as below.
 */
public enum EasyPredicate: RawRepresentable {
    
    // MARK: - Cases
    case exists(_ exists: Bool)
    case isEnabled(_ isEnabled: Bool)
    case isHittable(_ isHittable: Bool)
    case isSelected(_ isSelected: Bool)
    case label(_ comparison: Comparison, _ value: String)
    case identifier(_ identifier: String)
    case type(_ type: XCUIElement.ElementType)
    case other(_ ragular: String)
    
    // MARK: - RawRepresentable
    public init?(rawValue: PredicateRawValue) {
        switch rawValue {
        case .bool(let key, _, let value):
            switch key {
            case .exists:       self = .exists(value)
            case .isEnabled:    self = .isEnabled(value)
            case .isSelected:   self = .isSelected(value)
            case .isHittable:   self = .isHittable(value)
            }
        case .type(let value):  self = .type(value)
        case .string(let key, let comparison, let value):
            switch key {
            case .label:        self = .label(comparison, value)
            case .identifier:   self = .identifier(value)
            }
        case .custom(let regular): self = .other(regular)
        }
    }
    
    public var rawValue: PredicateRawValue {
        switch self {
        case .exists(let value):
            return .bool(key: .exists, comparison: .equals, value: value)
        case .isEnabled(let value):
            return .bool(key: .isEnabled, comparison: .equals, value: value)
        case .isHittable(let value):
            return .bool(key: .isHittable, comparison: .equals, value: value)
        case .isSelected(let value):
            return .bool(key: .isSelected, comparison: .equals, value: value)
        case .label(let comparison, let value):
            return .string(key: .label, comparison: comparison, value: value)
        case .identifier(let value):
            return .string(key: .identifier, comparison: .equals, value: value)
        case .type(let value):
            return .type(value: value)
        case .other(let value):
            return .custom(regular: value)
        }
    }
}

extension EasyPredicate: Equatable {

    // MARK: - Equatable

    /// Equatable prtocol
    ///
    /// - Parameters:
    ///   - l: left EasyPredicate
    ///   - r: right EasyPredicate
    /// - Returns: is equal result
    public static func ==(l: EasyPredicate, r: EasyPredicate) -> Bool {
        return l.regularString == r.regularString
    }
    
    /// convert to NSPredicate
    public var toPredicate: NSPredicate {
        return NSPredicate(format: regularString)
    }
    
    // MARK: - Extensions
    
    public var regularString: String {
        return rawValue.rawValue
    }

    /// merge two predicate semantics, taking their intersection
    ///
    /// - Parameter p: another easy predicate
    /// - Returns: new predicate
    public func and(_ p: EasyPredicate) -> EasyPredicate {
        return [self, p].merged(withLogic: .and)
    }
    
    /// merge two predicate semantics, taking their union
    ///
    /// - Parameter p: another easy predicate
    /// - Returns: new predicate
    public func or(_ p: EasyPredicate) -> EasyPredicate {
        return [self, p].merged(withLogic: .or)
    }

    /// reverse the semantics of predicates
    public var not: EasyPredicate {
        return EasyPredicate.other("!(\(regularString))")
    }
}

public extension Sequence where Element == EasyPredicate {
    /// convert EasyPredicates to NSCompoundPredicate
    func toPredicate(_ logic: NSCompoundPredicate.LogicalType) -> NSCompoundPredicate {
        return NSCompoundPredicate(type: logic, subpredicates: map { $0.toPredicate })
    }
    
    /// merged all EasyPredicate as one
    ///
    /// - Parameter logic: all EasyPredicate relate rule
    /// - Returns: new EasyPredicate
    func merged(withLogic logic: NSCompoundPredicate.LogicalType = .and) -> EasyPredicate {
        let regulars = map { "(\($0.regularString))" }
        let _logic = (logic == .not) ? .and : logic
        var result = regulars.joined(separator: _logic.regularString)
        if logic == .not { result = "!(\(result))" }
        return EasyPredicate.other(result)
    }
}

extension NSCompoundPredicate.LogicalType {
    /// convert LogicalType as regular string
    fileprivate var regularString: String {
        switch self {
        case .and: return " AND "
        case .or: return " OR "
        case .not: return " NOT "
        @unknown default: fatalError()
        }
    }
}
```


/UITest/Extensions/XCUIElementQuery+helpers.swift
```swift
public extension XCUIElementQuery {
    
    /// get the results which matching the EasyPredicates
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate's rules
    ///   - logic: rules relate
    /// - Returns: ElementQuery
    func matching(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElementQuery {
        return matching(predicates.toPredicate(logic))
    }
    func matching(predicate: EasyPredicate) -> XCUIElementQuery {
        return matching(predicate.toPredicate)
    }
    
    /// get the taget element which matching the EasyPredicates
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate's rules
    ///   - logic: rule's relate
    /// - Returns: result target
    func element(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElement {
        return element(matching: predicates.toPredicate(logic))
    }
    func element(predicate: EasyPredicate) -> XCUIElement {
        return element(matching: predicate.toPredicate)
    }

    /// get the results in the query's descendants which matching the EasyPredicates
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate's rules
    ///   - logic: rule's relate
    /// - Returns: result target
    func descendants(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElementQuery {
        return descendants(matching: .any).matching(predicates: predicates, logic: logic)
    }
    func descendants(predicate: EasyPredicate) -> XCUIElementQuery {
        return descendants(matching: .any).matching(predicate: predicate)
    }

    /// filter the query by rules to create new query
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate's rules
    ///   - logic: rule's relate
    /// - Returns: result target
    func containing(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElementQuery {
        return containing(predicates.toPredicate(logic))
    }
    func containing(predicate: EasyPredicate) -> XCUIElementQuery {
        return containing(predicate.toPredicate)
    }
}
```


/UITest/Extensions/XCUIElement+helpers.swift
```swift
// MARK: - Custom Extension
public extension XCUIElement {
    
    // MARK: - Traversing
    
    /// get the results in the descendants which matching the EasyPredicates
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate's rules
    ///   - logic: rule's relate
    /// - Returns: result target
    @discardableResult
    func descendants(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElementQuery {
        return descendants(matching: .any).matching(predicates: predicates, logic: logic)
    }
    @discardableResult
    func descendants(predicate: EasyPredicate) -> XCUIElementQuery {
        return descendants(matching: .any).matching(predicate.toPredicate)
    }
    
    /// Returns a query for direct children of the element matching with EasyPredicates
    ///
    /// - Parameters:
    ///   - predicates: EasyPredicate rules
    ///   - logic: rules relate
    /// - Returns: result query
    @discardableResult
    func children(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType = .and) -> XCUIElementQuery {
        return children(matching: .any).matching(predicates: predicates, logic: logic)
    }
    @discardableResult
    func children(predicate: EasyPredicate) -> XCUIElementQuery {
        return children(matching: .any).matching(predicate: predicate)
    }
}

public extension Sequence where Element: XCUIElement {
    
    /// get the elements which match with identifiers and predicates limited in timeout
    ///
    /// - Parameters:
    ///   - predicates: predicates as the match rules
    ///   - logic: relation of predicates
    ///   - timeout: if timeout == 0, return the elements immediately otherwise retry until timeout
    /// - Returns: get the elements
    func elements(predicates: [EasyPredicate], logic: NSCompoundPredicate.LogicalType, timeout: Int) -> [Element] {
        if predicates.count <= 0 { fatalError("predicates cannpt be empty!") }
        
        let array = map { $0 } as NSArray
        let filteredElements = array.filtered(using: predicates.toPredicate(logic))
        if filteredElements.count > 0 || timeout <= 0 {
            return filteredElements as! [Element]
        } else {
            sleep(1)
            return self.elements(predicates: predicates, logic: logic, timeout: timeout - 1)
        }
    }
    
    /// get the first element was matched predicate
    func anyElement(predicate: EasyPredicate) -> Element? {
        return elements(predicates: [predicate], logic: .and, timeout: 0).first
    }
}
```


**Source:** [UI Predicate](https://github.com/ZhipingYang/Einstein/blob/master/Class/UITest/Model/EasyPredicate.swift)

