## Mediator in Swift

> **Mediator** is a behavioral design pattern that reduces coupling between components of a program by making them communicate indirectly, through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual components because they’re no longer dependent on the dozens of other classes.

---

### Usage of the pattern in Swift

**Usage examples**: The most popular usage of the Mediator pattern in Swift code is facilitating communications between GUI components of an app. The synonym of the Mediator is the Controller part of MVC pattern.


### Conceptual Example

This example illustrates the structure of the **Mediator** design pattern. It focuses on answering these questions:


- What classes does it consist of?
- What roles do these classes play?
- In what way the elements of the pattern are related?

After learning about the pattern’s structure it’ll be easier for you to grasp the following example, based on a real-world Swift use case.

#### Example1.swift: Conceptual example

```swift
import XCTest

/// The Mediator interface declares a method used by components to notify the
/// mediator about various events. The Mediator may react to these events and
/// pass the execution to other components.
protocol Mediator: AnyObject {

    func notify(sender: BaseComponent, event: String)
}

/// Concrete Mediators implement cooperative behavior by coordinating several
/// components.
class ConcreteMediator: Mediator {

    private var component1: Component1
    private var component2: Component2

    init(_ component1: Component1, _ component2: Component2) {
        self.component1 = component1
        self.component2 = component2

        component1.update(mediator: self)
        component2.update(mediator: self)
    }

    func notify(sender: BaseComponent, event: String) {
        if event == "A" {
            print("Mediator reacts on A and triggers following operations:")
            self.component2.doC()
        }
        else if (event == "D") {
            print("Mediator reacts on D and triggers following operations:")
            self.component1.doB()
            self.component2.doC()
        }
    }
}

/// The Base Component provides the basic functionality of storing a mediator's
/// instance inside component objects.
class BaseComponent {

    fileprivate weak var mediator: Mediator?

    init(mediator: Mediator? = nil) {
        self.mediator = mediator
    }

    func update(mediator: Mediator) {
        self.mediator = mediator
    }
}

/// Concrete Components implement various functionality. They don't depend on
/// other components. They also don't depend on any concrete mediator classes.
class Component1: BaseComponent {

    func doA() {
        print("Component 1 does A.")
        mediator?.notify(sender: self, event: "A")
    }

    func doB() {
        print("Component 1 does B.\n")
        mediator?.notify(sender: self, event: "B")
    }
}

class Component2: BaseComponent {

    func doC() {
        print("Component 2 does C.")
        mediator?.notify(sender: self, event: "C")
    }

    func doD() {
        print("Component 2 does D.")
        mediator?.notify(sender: self, event: "D")
    }
}

/// Let's see how it all works together.
class MediatorConceptual: XCTestCase {

    func testMediatorConceptual() {

        let component1 = Component1()
        let component2 = Component2()

        let mediator = ConcreteMediator(component1, component2)
        print("Client triggers operation A.")
        component1.doA()

        print("\nClient triggers operation D.")
        component2.doD()

        print(mediator)
    }
}
```


#### OutputConceptual.txt: Execution result

```
Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.

Component 2 does C.
```

---

### Real World Example


#### Example2.swift: Real world example

```swift
import XCTest

class MediatorRealWorld: XCTestCase {

    func test() {

        let newsArray = [News(id: 1, title: "News1", likesCount: 1),
                         News(id: 2, title: "News2", likesCount: 2)]

        let numberOfGivenLikes = newsArray.reduce(0, { $0 + $1.likesCount })

        let mediator = ScreenMediator()

        let feedVC = NewsFeedViewController(mediator, newsArray)
        let newsDetailVC = NewsDetailViewController(mediator, newsArray.first!)
        let profileVC = ProfileViewController(mediator, numberOfGivenLikes)

        mediator.update([feedVC, newsDetailVC, profileVC])

        feedVC.userLikedAllNews()
        feedVC.userDislikedAllNews()
    }
}

class NewsFeedViewController: ScreenUpdatable {

    private var newsArray: [News]
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ newsArray: [News]) {
        self.newsArray = newsArray
        self.mediator = mediator
    }

    func likeAdded(to news: News) {

        print("News Feed: Received a liked news model with id \(news.id)")

        for var item in newsArray {
            if item == news {
                item.likesCount += 1
            }
        }
    }

    func likeRemoved(from news: News) {

        print("News Feed: Received a disliked news model with id \(news.id)")

        for var item in newsArray {
            if item == news {
                item.likesCount -= 1
            }
        }
    }

    func userLikedAllNews() {
        print("\n\nNews Feed: User LIKED all news models")
        print("News Feed: I am telling to mediator about it...\n")
        newsArray.forEach({ mediator?.likeAdded(to: $0) })
    }

    func userDislikedAllNews() {
        print("\n\nNews Feed: User DISLIKED all news models")
        print("News Feed: I am telling to mediator about it...\n")
        newsArray.forEach({ mediator?.likeRemoved(from: $0) })
    }
}

class NewsDetailViewController: ScreenUpdatable {

    private var news: News
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ news: News) {
        self.news = news
        self.mediator = mediator
    }

    func likeAdded(to news: News) {
        print("News Detail: Received a liked news model with id \(news.id)")
        if self.news == news {
            self.news.likesCount += 1
        }
    }

    func likeRemoved(from news: News) {
        print("News Detail: Received a disliked news model with id \(news.id)")
        if self.news == news {
            self.news.likesCount -= 1
        }
    }
}

class ProfileViewController: ScreenUpdatable {

    private var numberOfGivenLikes: Int
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ numberOfGivenLikes: Int) {
        self.numberOfGivenLikes = numberOfGivenLikes
        self.mediator = mediator
    }

    func likeAdded(to news: News) {
        print("Profile: Received a liked news model with id \(news.id)")
        numberOfGivenLikes += 1
    }

    func likeRemoved(from news: News) {
        print("Profile: Received a disliked news model with id \(news.id)")
        numberOfGivenLikes -= 1
    }
}

protocol ScreenUpdatable: class {

    func likeAdded(to news: News)

    func likeRemoved(from news: News)
}

class ScreenMediator: ScreenUpdatable {

    private var screens: [ScreenUpdatable]?

    func update(_ screens: [ScreenUpdatable]) {
        self.screens = screens
    }

    func likeAdded(to news: News) {
        print("Screen Mediator: Received a liked news model with id \(news.id)")
        screens?.forEach({ $0.likeAdded(to: news) })
    }

    func likeRemoved(from news: News) {
        print("ScreenMediator: Received a disliked news model with id \(news.id)")
        screens?.forEach({ $0.likeRemoved(from: news) })
    }
}

struct News: Equatable {

    let id: Int

    let title: String

    var likesCount: Int

    /// Other properties

    static func == (left: News, right: News) -> Bool {
        return left.id == right.id
    }
}
```


#### OutputRealWorld.txt: Execution result

```
News Feed: User LIKED all news models
News Feed: I am telling to mediator about it...

Screen Mediator: Received a liked news model with id 1
News Feed: Received a liked news model with id 1
News Detail: Received a liked news model with id 1
Profile: Received a liked news model with id 1
Screen Mediator: Received a liked news model with id 2
News Feed: Received a liked news model with id 2
News Detail: Received a liked news model with id 2
Profile: Received a liked news model with id 2


News Feed: User DISLIKED all news models
News Feed: I am telling to mediator about it...

ScreenMediator: Received a disliked news model with id 1
News Feed: Received a disliked news model with id 1
News Detail: Received a disliked news model with id 1
Profile: Received a disliked news model with id 1
ScreenMediator: Received a disliked news model with id 2
News Feed: Received a disliked news model with id 2
News Detail: Received a disliked news model with id 2
Profile: Received a disliked news model with id 2
```
