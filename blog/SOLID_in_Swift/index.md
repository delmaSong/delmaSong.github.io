---
layout: post
title:  "Swift 개발에 SOLID 적용하기"
subtitle: "다 이유가 있는 SOLID"
type: "Swift"
blog: true
text: true
author: "Delma Song"
post-header: true
header-img: "img/header.png"
order: 1
---

<br>

> 해당 글은, 원문 글쓴이의 동의를 얻어 '[SOLID Pirincipels Applied to Swift Development](https://medium.com/better-programming/solid-principles-application-to-swift-development-1de8d7c57fdf)'를 번역한 글입니다. 매끄러운 의미 전달을 위해 의역한 부분이 있습니다.

<br>

## Introduction
개발자로서, 아마 SOLID 원칙을 들어보거나 사용해 본 적이 있을 것이다. Robert C. Martin(Uncle Bob)은 그의 저서 `Agile Software Development: Principles, Patterns, and Practices`에서 디자인 패턴과 관련된 객체 지향 프로그래밍의 5가지 기본 원칙인 SOLID를 언급하고 있다.

우리가 이 다섯가지 원칙을 적용한다면:
- 쉽게 변경이 가능한 유연한 코드를 가지게 된다. 이것은 재사용과 유지 관리가 가능하다.
- 그렇게 개발된 소프트웨어는 안정되고, 탄탄하며 확장성이 뛰어날 것이다. (우리는 쉽게 새로운 기능을 추가할 수 있다.)
- 디자인 패턴의 사용과 함께, 응집력이 높고(시스템의 요소가 밀접하게 연관되어있다) 결합도가 낮은(요소들 간의 의존도가 낮다) 소프트웨어를 만들 수 있다.  

<br>

### SOLID란
- **S** (Single Responsibility Principle) : 단일 책임 원칙
- **O** (Open/Close Principle) : 개방/폐쇄 원칙
- **L** (Liscov Substitution Principle) : 리스코브 치환 원칙
- **I** (Interface Segregation Principle) : 인터페이스 분리 원칙
- **D** (Dependency Inversion Principle) : 의존성 역전 원칙

<br>
<hr>

## Single Responsibility Principle (단일 책임 원칙)
<br>

이 원칙에 따르면, 클래스는 하나의 이유로만 변화해야 한다. 다시 말해, 클래스는 한가지의 책임만 가지고 있어야 한다.

예시를 보자.
```swift
class LoginUser {
    func login() {
        let data = authenticareUserViaAPI()
        let user = decodeUser(data: data)
        saveToDB(array: array)
    }
    
    private func authenticareUserViaAPI() -> Data {
        // Call server to authenticate and return user's info
    }
    
    private func decodeUser(data: Data) -> User {
        // Decode data (Codable protocol) into User object
    }
    
    private func saveUserInfoOnDatabase(user: User) {
        // Save User info onto database
    }
}
```

<br>

이 클래스는 인증, 디코딩, 저장의 3가지 책임을 가지고 있다. 단일 책임 원칙을 충족하기 위해서는 이 책임을 각각의 작은 클래스로 분리해야 한다. 

<br>

```swift
class LoginUser {
    let oAuthHandler: OAuthHandler
    let decodeHandler: DecodeHandler
    let databaseHandler: DataBaseHandler
    
    init(oAuthHandler: OAuthHandler, decodeHandler: DecodeHandler, databaseHandler: DataBaseHandler) {
        self.oAuthHandler = oAuthHandler
        self.decodeHandler = decodeHandler
        self.databaseHandler = databaseHandler
    }
    
    func login() {
        let data = oAuthHandler.authenticareUserViaAPI()
        let user = decodeHandler.decodeUser(data: data)
        databaseHandler.saveUserInfoOnDatabase(user: user)
    }
}

class OAuthHandler {
    func authenticareUserViaAPI() -> Data {
        // Call server to authenticate and return user's info
    }
}

class DecodeHandler {
    func decodeUser(data: Data) -> User {
        // Decode data (Codable protocol) into User object
    }
}

class DataBaseHandler {
    func saveUserInfoOnDatabase(user: User) {
        // Save User info onto database
    }
}
```

<br>
<hr>
<br>

## Open/Close Principle (개방/폐쇄 원칙)

이 원칙에 따르면, 확장은 클래스의 행동의 변화 없이 가능해야 한다.(확장에는 열려있고 변경에는 닫혀있어야 함) 이것은 추상화에 의해 가능해진다.
<br>

```swift
class Scrapper {

    func scrapVehicles() {
        let cars = [
            Car(brand: "Ford"),
            Car(brand: "Peugeot"),
            Car(brand: "Toyota"),
        ]

        cars.forEach { car in
            print(car.getScrappingAddress())
        }

        let trucks = [
            Truck(brand: "Volvo"),
            Truck(brand: "Nissan"),
        ]

        trucks.forEach { truck in
            print(truck.getScrappingAddress())
        }
    }
}

class Car {
    let brand: String

    init(brand: String) {
        self.brand = brand
    }

    func getScrappingAddress() -> String {
        return "Cars scrapping address"
    }
}

class Truck {
    let brand: String

    init(brand: String) {
        self.brand = brand
    }

    func getScrappingAddress() -> String {
        return "Trucks scrapping address"
    }
}

```
<br>

각각의 새로운 타입의 vehicle에 대해, `getScrapingAddress()`는 다시 구현되어야만 한다. 이것은 개방/폐쇄 원칙을 어기게 된다.

이것을 해결하기 위해, `getScrapingAddress()` 메소드를 포함한 `Scrappable` 프로토콜을 만든다.
<br>

```swift
protocol Scrappable {
    func getScrapingAddress() -> String
}

class Scrapper {

    func getScrapingAddress() {
        let vehicles: [Scrappable] = [
            Car(brand: "Ford"),
            Car(brand: "Peugeot"),
            Car(brand: "Toyota"),
            Truck(brand: "Volvo"),
            Truck(brand: "Nissan"),
        ]

        vehicles.forEach { vehicle in
            print(vehicle.getScrapingAddress())
        }
    }
}

class Car: Scrappable {
    let brand: String

    init(brand: String) {
        self.brand = brand
    }

    func getScrapingAddress() -> String {
        return "Cars scrapping address"
    }
}

class Truck: Scrappable {
    let brand: String

    init(brand: String) {
        self.brand = brand
    }

    func getScrapingAddress() -> String {
        return "Trucks scrapping address"
    }
}
```


<br>
<hr>
<br>

## Liscov Substitution Principle (리스코브 치환 원칙)
1987년 [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov)에 의해 소개된 이 원칙은, 프로그램에서 어떤 클래스도 그 기능에 영향을 주지 않고 해당 클래스의 서브클래스로 대체 가능해야 한다고 명시하고 있다.

예를들어, 사용자와 연락하는(이메일 보내기와 같은) 책임을 가진 `UserService`라는 클래스가 하나 있다고 하자. 

만약 우리가 17세 이상의 사용자에게 이메일을 보내도록 비즈니스 로직을 변경해야 한다면, 새로운 비즈니스 로직을 가지는 서브클래스를 만들 수 있다

<br>

```swift
class UserService {
    func contact(user: User) {
        // Retrieve user from database
    }
}

class ValidUserService: UserService {
    override func contact(user: User) {
        guard user.age > 17 else { return }

        super.contact(user: User)
    }
}
```
<br>

이런 경우에, 사용자의 나이가 17세 이상이라는 조건을 서브클래스가 추가했기 때문에 리스코브 치환 원칙은 충족되지 않는다. 

서브클래스를 추가하지 않고 `UserService`에 (기본값을 포함한)전제 조건을 추가하는 것으로 이러한 문제를 해결할 수 있다.
<br>

```swift
class UserService {
    func contact(user: User, minAge: Int = 0) {
        guard user.age > minAge else { return }
        // Retrieve user from database
    }
}
```

<br>
<hr>
<br>

## Interface Segregation Principle 인터페이스 분리 원칙

<br>
인터페이스 분리 원칙은 일반적인 인터페이스를 가지는 것보다 구체적인 각각의 다른 인터페이스를 가지는 것이 낫다고 명시한다.

또한, 사용하지 않는 메서드는 구현 할 필요가 없다는 것을 명시한다.

예를들어, 동물이 움직이는 메소드를 포함한 `AnimalProtocol`을 만들어 보자. 

```swift
protocol AnimalProtocol {
    func walk()
    func swimm()
    func fly()
}

struct Animal: AnimalProtocol {
    func walk() {}
    func swimm() {}
    func fly() {}
}

struct Whale: AnimalProtocol {
    func swimm() {
    	//Whale은 이 메소드만 구현하면 된다
        //다른 메소드는 상관 없음
    }
    
    func walk() {}
    
    func fly() {}
}
```
<br>

`Whale`이 프로토콜을 채택했음에도 메소드 두개는 구현하지 않게된다. 해결 방법은 각 3개의 인터페이스(프로토콜)을 만들어 각각 필요한 프로토콜만 채택하는 것이다.
<br>


```swift
protocol WalkProtocol {
    func walk()
}

protocol SwimmProtocol {
    func swimm()
}

protocol FlyProtocol {
    func fly()
}

struct Whale: SwimmProtocol {
    func swimm() {}
}

struct Crocodile: WalkProtocol, SwimmProtocol {
    func walk()
    func swimm() {}
}
```

<br>
<hr>
<br>

## Dependency Inversion Principle 의존성 역전 원칙
<br>
이 원칙을 따르면
- 상위 클래스가 하위 클래스에 의존하지 않게 된다. 둘다 추상타입에 의존하게 된다.
- 추상타입은 구체타입에 의존하지 않게 된다. 구체타입이 추상타입에 의존하게 된다.

이 원칙은 모듈간의 의존성을 낮추도록 하고 따라서 클래스간의 결합도를 낮추도록 한다.

예제를 보자.
```swift
class User {
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

class CoreData {
    func save(user: User) {
        // Save user on database
    }
}

class UserService {
    func save(user: User) {
        let database = CoreData()
        database.save(user: user)
    }
}
```
<br>

데이터를 저장할 때, `CoreData` 대신 Realm 데이터베이스를 사용하고 싶다면 어떻게 해야할까?
위 예시처럼 클래스의 인스턴스는 강한 결합을 만들어내므로, 만약 다른 데이터베이스를 이용하고 싶다면 코드를 다시 실행해야 할 것이다.

이것을 해결하기 위해, CoreData와 Realm이 공통적으로 채택할 프로토콜을 만들고, 데이터 저장하는 타입을 해당 프로토콜로 두면 된다.

<br>


```swift
protocol Storable { }

extension Object: Storable { } // Realm Database

extension NSManagedObject: Storable { } // Core Data Database

protocol StorageManager {
  /// Save Object into Realm database
  /// - Parameter object: Realm object (as Storable)
  func save(object: Storable)
}
```
<br>

`Storable` 프로토콜을 채택하는 `User` 와 `UserService` 클래스는 `StorageManager` 프로토콜을 채택하므로 데이터베이스를 변경하더라도 전체 구현 코드를 변경할 필요가 없다.

<br>

```swift
class User: Storable {
    var name: String
    
    init(name: String) {
        self.name = name
    }
}

class UserService: StorageManager {
    func save(object: Storable) {
        // Saves user to database
    }
}

```

<br>

### Conclusion
이러한 원칙들이 재사용 가능하고, 유지보수가 용이한 더 높은 품질의 코드를 만들 수 있게 함으로써 코드의 경직되고 취약한 부분을 감소하게 할 것이다. 