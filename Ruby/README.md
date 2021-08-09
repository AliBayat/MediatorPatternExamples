## Mediator in Ruby

> **Mediator** is a behavioral design pattern that reduces coupling between components of a program by making them communicate indirectly, through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual components because they’re no longer dependent on the dozens of other classes.

---

### Usage of the pattern in Ruby

**Usage examples**: The most popular usage of the Mediator pattern in Ruby code is facilitating communications between GUI components of an app. The synonym of the Mediator is the Controller part of MVC pattern.


### Conceptual Example

This example illustrates the structure of the **Mediator** design pattern. It focuses on answering these questions:


- What classes does it consist of?
- What roles do these classes play?
- In what way the elements of the pattern are related?


#### main.rb: Conceptual example

```ruby
# The Mediator interface declares a method used by components to notify the
# mediator about various events. The Mediator may react to these events and pass
# the execution to other components.
class Mediator
  # @abstract
  #
  # @param [Object] sender
  # @param [String] event
  def notify(_sender, _event)
    raise NotImplementedError, "#{self.class} has not implemented method '#{__method__}'"
  end
end

class ConcreteMediator < Mediator
  # @param [Component1] component1
  # @param [Component2] component2
  def initialize(component1, component2)
    @component1 = component1
    @component1.mediator = self
    @component2 = component2
    @component2.mediator = self
  end

  # @param [Object] sender
  # @param [String] event
  def notify(_sender, event)
    if event == 'A'
      puts 'Mediator reacts on A and triggers following operations:'
      @component2.do_c
    elsif event == 'D'
      puts 'Mediator reacts on D and triggers following operations:'
      @component1.do_b
      @component2.do_c
    end
  end
end

# The Base Component provides the basic functionality of storing a mediator's
# instance inside component objects.
class BaseComponent
  # @return [Mediator]
  attr_accessor :mediator

  # @param [Mediator] mediator
  def initialize(mediator = nil)
    @mediator = mediator
  end
end

# Concrete Components implement various functionality. They don't depend on
# other components. They also don't depend on any concrete mediator classes.
class Component1 < BaseComponent
  def do_a
    puts 'Component 1 does A.'
    @mediator.notify(self, 'A')
  end

  def do_b
    puts 'Component 1 does B.'
    @mediator.notify(self, 'B')
  end
end

class Component2 < BaseComponent
  def do_c
    puts 'Component 2 does C.'
    @mediator.notify(self, 'C')
  end

  def do_d
    puts 'Component 2 does D.'
    @mediator.notify(self, 'D')
  end
end

# The client code.
c1 = Component1.new
c2 = Component2.new
ConcreteMediator.new(c1, c2)

puts 'Client triggers operation A.'
c1.do_a

puts "\n"

puts 'Client triggers operation D.'
c2.do_d
```


#### Output.txt: Execution result

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
