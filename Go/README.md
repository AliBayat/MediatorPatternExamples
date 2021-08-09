## Mediator in Go

> **Mediator** is a behavioral design pattern that reduces coupling between components of a program by making them communicate indirectly, through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual components because they’re no longer dependent on the dozens of other classes.

---

### Conceptual Example

An excellent example of the Mediator pattern is a railway station traffic system. Two trains never communicate between themselves for the availability of the platform. The `stationManager` acts as a mediator and makes the platform available to only one of the arriving trains while keeping the rest in a queue. A departing train notifies the stations, which lets the next train in the queue to arrive.
 

#### train.go: Component

```go
package main

type train interface {
    arrive()
    depart()
    permitArrival()
}
```


#### passengerTrain.go: Concrete component

```go
package main

import "fmt"

type passengerTrain struct {
    mediator mediator
}

func (g *passengerTrain) arrive() {
    if !g.mediator.canArrive(g) {
        fmt.Println("PassengerTrain: Arrival blocked, waiting")
        return
    }
    fmt.Println("PassengerTrain: Arrived")
}

func (g *passengerTrain) depart() {
    fmt.Println("PassengerTrain: Leaving")
    g.mediator.notifyAboutDeparture()
}

func (g *passengerTrain) permitArrival() {
    fmt.Println("PassengerTrain: Arrival permitted, arriving")
    g.arrive()
}
```


#### freightTrain.go: Concrete component

```go
package main

import "fmt"

type freightTrain struct {
    mediator mediator
}

func (g *freightTrain) arrive() {
    if !g.mediator.canArrive(g) {
        fmt.Println("FreightTrain: Arrival blocked, waiting")
        return
    }
    fmt.Println("FreightTrain: Arrived")
}

func (g *freightTrain) depart() {
    fmt.Println("FreightTrain: Leaving")
    g.mediator.notifyAboutDeparture()
}

func (g *freightTrain) permitArrival() {
    fmt.Println("FreightTrain: Arrival permitted")
    g.arrive()
}
```


#### customer.go: Concrete observer

```go
package main

import "fmt"

type customer struct {
    id string
}

func (c *customer) update(itemName string) {
    fmt.Printf("Sending email to customer %s for item %s\n", c.id, itemName)
}

func (c *customer) getID() string {
    return c.id
}
```


#### main.go: Client code

```go
package main

func main() {

    shirtItem := newItem("Nike Shirt")

    observerFirst := &customer{id: "abc@gmail.com"}
    observerSecond := &customer{id: "xyz@gmail.com"}

    shirtItem.register(observerFirst)
    shirtItem.register(observerSecond)

    shirtItem.updateAvailability()
}
```


#### Output.txt: Execution result

```
Item Nike Shirt is now in stock
Sending email to customer abc@gmail.com for item Nike Shirt
Sending email to customer xyz@gmail.com for item Nike Shirt
```

