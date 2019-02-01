---
layout: default
title: Scrolling
parent: XCUITest
nav_order: 5
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

## Scrolling

### Scroll to an item
```swift
    func isVisible(_ app: XCUIApplication) -> Bool {
        let window = app.windows.element(boundBy: 0).firstMatch

        return waitForElementToBecomeHittable(timeout: .small) && !frame(app).isEmpty && window.frame(app).contains(frame(app))
    }
```

**Source:** [Scroll helper](http://averagepro.com/2016/02/12/more-xctest-ut-testing-a-helper-function-to-scroll-to-a-cell-in-a-uicollectionview/)

The helper function scrolls 1/2 the height of the collection View over and over until either the cell you are looking for is hittable, or the scroll doesn’t actually change anything (you hit the top or bottom of the collection view).

Note that the first check it does for the cell, looks like this:

```swift 
collectionViewElement.cells.matchingIdentifier(cellIdentifier).count > 0
```

This lets you query the collectionView cells to see if the identifier is present without having the test fail by directly checking for the cell with collectionViewElement.cells[cellIdentifier], you would get a failure in the test, and it wouldn’t continue.

The code checks to see if the touch changed anything by keeping track of the ‘middle’ cell in the list of cells for the collection view (which by-the-way, might include cells that are no longer displayed), and determines if it’s the same cell id and same frame to see if anything has changed.

After the cell is found to be hittable, if you want it fully visible, there is a second loop that scrolls up or down by a smaller amount (1/2 the height of the cell) until the cell’s frame is fully contained by the collection view’s frame.



```swift
import XCTest

extension Page {
    @discardableResult
    func scrollTo(
        _ target: XCUIElement,
        scrollView collectionView: XCUIElement,
        scrollDirection: ScrollDirection = .bottomToTop,
        maxiumumAttempts: Int = 5,
        scrollDistance: CGFloat = 0.1
        ) -> Bool {
        
        return smartScroll(
            target: target,
            scrollView: collectionView,
            scrollDirection: scrollDirection,
            maxiumumAttempts: maxiumumAttempts,
            scrollDistance: scrollDistance
        )
    }
    
    private func smartScroll(
        target: XCUIElement,
        scrollView: XCUIElement,
        scrollDirection: ScrollDirection,
        maxiumumAttempts: Int,
        matchByLabel: String? = nil,
        scrollDistance: CGFloat
        ) -> Bool {
        
        /*
         * This currently works on the assumption that items are returned left to right and top to bottom.
         */
        func midItem() -> XCUIElement? {
            let children = scrollView.children(matching: .any)
            return children.count > 0 ? children.element(boundBy: children.count / 2) : nil
        }
        
        var scrollAttempt          = 0
        var lastMidChildIdentifier = Optional("")
        var lastMidChildRect       = Optional(CGRect.zero)
        
        var targetDistance: CGFloat = scrollDistance
        
        var currentMidChild = midItem()
        
        let elementIsNotInView = !(lastMidChildIdentifier == currentMidChild?.identifier &&
            (currentMidChild?.frame(app) ?? CGRect.infinite).equalTo(lastMidChildRect ?? CGRect.zero))

        // The default behaviour is to scroll the collection view until the element exits using the defined
        // scroll direction, but there are instances where it would make sense to reverse the scroll direction
        // if the element exists and the element is on the opposite side of the scroll direction.
        // An example of this is if we need the item to be centre with in the collection view but the item
        // is to the left of the centre item like in the a carousel.
        //
        // Example:
        //            Scroll Direction: Right ----->
        //
        //                 Current Item
        //                       v
        // | Item 5 | Item 1 | Item 2 | Item 3 | Item 4 |
        //              ^
        //        Desired Item
        //
        //     <----- Reverse The Scroll Direction
        func normalize(scrollDirection: ScrollDirection) -> ScrollDirection {
            guard target.waitForElementToBecomeHittable(timeout: .small) else {
                targetDistance = scrollDistance
                
                return scrollDirection
            }
            
            targetDistance = 0.5
            
            let elementFrame   = target.frame(app)
            let scollViewFrame = scrollView.frame(app)
            
            // Calculate the element position compared to the scrollView
            let elementIsToTheLeft   = elementFrame.midX < scollViewFrame.midX
            let elementIsToTheBottom = elementFrame.midY > scollViewFrame.midY
            
            switch scrollDirection {
            case .leftToRight where !elementIsToTheLeft:
                return scrollDirection.inverted()
            case .rightToLeft where elementIsToTheLeft:
                return scrollDirection.inverted()
            case .topToBottom where elementIsToTheBottom:
                return scrollDirection.inverted()
            case .bottomToTop where !elementIsToTheBottom:
                return scrollDirection.inverted()
            default:
                return scrollDirection
            }
        }
        // Wait for ScrollView to exist
        _ = scrollView.waitForExistence(timeout: .small)

        while elementIsNotInView && scrollAttempt < maxiumumAttempts {

            let collectionViewValue = scrollView.value as? String ?? ""

            if target.isVisible(app) || matchByLabel != nil && collectionViewValue == matchByLabel {
               return true
            }
            
            lastMidChildIdentifier = currentMidChild?.identifier
            lastMidChildRect       = currentMidChild?.frame(app)
            
            let (startOffset, endOffset) = normalize(scrollDirection: scrollDirection).vectors(targetDistance: targetDistance)
            
            scrollView
                .coordinate(withNormalizedOffset: startOffset)
                .press(forDuration: 0.01, thenDragTo: scrollView.coordinate(withNormalizedOffset: endOffset))
            
            scrollAttempt  += 1
            currentMidChild = midItem()
        }
        return false
    }

}
```

```swift
    enum ScrollDirection {
        case topToBottom
        case bottomToTop
        case leftToRight
        case rightToLeft
        
        func inverted() -> ScrollDirection {
            switch self {
            case .topToBottom:
                return .bottomToTop
            case .bottomToTop:
                return .topToBottom
            case .leftToRight:
                return .rightToLeft
            case .rightToLeft:
                return .leftToRight
            }
        }
        
        func vectors(targetDistance: CGFloat) -> (start: CGVector, finish: CGVector) {
            switch self {
            case .topToBottom:
                return (start: CGVector(dx: 0.99, dy: targetDistance), finish: CGVector(dx: 0.99, dy: 0.9))
            case .bottomToTop:
                return (start: CGVector(dx: 0.99, dy: 0.9), finish: CGVector(dx: 0.99, dy: targetDistance))
            case .leftToRight:
                return (start: CGVector(dx: targetDistance, dy: 0.99), finish: CGVector(dx: 0.9, dy: 0.99))
            case .rightToLeft:
                return (start: CGVector(dx: 0.9, dy: 0.99), finish: CGVector(dx: targetDistance, dy: 0.99))
            }
        }
    }
```
