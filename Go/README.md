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


#### mediator.go: Mediator interface

```go
package main

type mediator interface {
    canArrive(train) bool
    notifyAboutDeparture()
}
```


#### stationManager.go: Concrete mediator

```go
package main

type stationManager struct {
    isPlatformFree bool
    trainQueue     []train
}

func newStationManger() *stationManager {
    return &stationManager{
        isPlatformFree: true,
    }
}

func (s *stationManager) canArrive(t train) bool {
    if s.isPlatformFree {
        s.isPlatformFree = false
        return true
    }
    s.trainQueue = append(s.trainQueue, t)
    return false
}

func (s *stationManager) notifyAboutDeparture() {
    if !s.isPlatformFree {
        s.isPlatformFree = true
    }
    if len(s.trainQueue) > 0 {
        firstTrainInQueue := s.trainQueue[0]
        s.trainQueue = s.trainQueue[1:]
        firstTrainInQueue.permitArrival()
    }
}
```


#### main.go: Client code

```go
package main

func main() {
    stationManager := newStationManger()

    passengerTrain := &passengerTrain{
        mediator: stationManager,
    }
    freightTrain := &freightTrain{
        mediator: stationManager,
    }

    passengerTrain.arrive()
    freightTrain.arrive()
    passengerTrain.depart()
}
```


#### Output.txt: Execution result

```
Item Nike Shirt is now in stock
Sending email to customer abc@gmail.com for item Nike Shirt
Sending email to customer xyz@gmail.com for item Nike Shirt
```

