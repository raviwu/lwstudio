+++
title = "筆記 | Practical Object-Oriented Design in Ruby (POODR)"
date = 2017-01-03T13:20:41+08:00
publishdate = 2017-01-03T13:20:41+08:00
tags = ["coding", "ruby", "book", "notes"]
draft = false

description = ""
summary = ""
keywords = []

[amp]
    elements = []

[author]
    name = "Ravi Wu"
    homepage = "https://raviwu.github.io"

[image]
    src = ""

[sitemap]
    changefreq = "monthly"
    priority = 0.5
    filename = "sitemap.xml"
+++

[Practical Object-Oriented Design in Ruby (POODR)](http://www.poodr.com/) 不會很厚，循序漸進地介紹物件導向設計的各種重要概念，而且範例用的是 Ruby 來解說，挺親切的。

除了各種設計原則之外，也簡要解釋了 Inheritance / Module / Composition 的使用時機與差異。

最後一章介紹測試原則，除了說明一般的測試原則之外（例如主要應測試 public interface / incoming message / outgoing command ），也很清楚地說明要怎樣去分別把不同的測試責任分在 module / test double / test class 上，以及讓 test double 與實際的程式碼同步的技巧，可以反覆閱讀的參考書。

# Chapter 1 - Object-Oriented Design

> P.4
>
> Practical design does not anticipate what will happen to your application, it merely accepts that something will and that, in the present, you cannot know what. It doesn’t guess the future; it preserves your options for accommodating the future. It doesn’t choose; it leaves you room to move.
The purpose of design is to allow you to do design later and its primary goal is to reduce the cost of change.

> P.63
>
> The design goal, as always, is to retain maximum future flexibility while writing only enough code to meet today’s requirements.

## Design Principles

- Single Responsibility
- Open-Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion
- DRY (Don't Repeat Yourself)
- LoD (Law of Demeter)

## Design Patterns

Specific patterns are aim to solve specific problem, only apply pattern on the problem if the problem is what that pattern aims to solve.

# Chapter 2 - Designing Classes with a Single Responsibility

What means easy to change in Programming?

- Change has no unexpected side effects
- Small changes in requirements require correspondingly small changes in code
- Existing code is easy to reuse
- The easiest way to make a change is to add code that in itself is easy to change

Code should be

- Transparent = The consequences of changes should be obvious in the code that is changing and in distant code that relies upon it
- Resonable = The cost of any change should be proportional to the benefits the change achieves
- Usable = Existing code should be usable in new and unexpected contexts
- Exemplary = The code itself should encourage those who change it to perperuate these qualities

How to help find out whether a Class is having single responsibility:

- Ask questions like = ‘Dear Mr. Gear, what is your ratio?’, ‘Dear Mr. Gear, what is your tire size?’, if the question seems like asking the wrong guys, then the responsibility is probably belongs to others.
- Use one sentenct to describe the class. The sentence should not have ‘and’ or ‘or’ that indicating out the class is doing more than one thing.

Tips to write code embraces change:

- Depends on behavior, not data.
  * Always wrap instance variables in accessor methods instead of directly referring to variables.
  * Hide data structure, if a object relying on a data argument like `locations = [[5, 12], [2, 3], [4, 2]]` wherein each `location[0]` means `x` and `location[1]` means `y`, then better wrap the `location` to a Location Class that stores x and y as its property. Ruby has a convinient `Struct` class to easily provide simple object that stores the data and creates methods to response with stored data.
- Extract Responsibilities from methods. Method itself should be single responsibility if possible, benifits to extract reponsibilities from methods:
  * Expose previously hidden qualities = refactoring a class so that all of its methods have a single responsibility has a clarifying effect on the class.
  * Avoid the need for comments
  * Encourage reuse
  * Easy to move to another class
- Isolate Extra Responsibilities in Classes

# Chapter 3 - Managing Dependencies

> P.35
>
> Because well designed objects have a single responsibility, their very nature requires that they collaborate to accomplish complex tasks. This collaboration is powerful and perilous. To collaborate, an object must know something know about others. Knowing creates a dependency. If not managed carefully, these dependencies will strangle your application.

## Recognizing Dependencies

An object has a dependency when it knows:

- The name of another class
- The name of a message that it intends to send to someone other than self
- The arguments that a message requires
- The order of those arguments
- Other Dependencies:
  * a object knowing the name of a message you plan to send to someone other than `self`, this is a violation of LoD, can be resolve by flexible interface, checkout chapter 4 creating flexible interfaces
  * test depends on code, checkout chapter 9 designing cost-effective tests

## Resolving Dependencies

### Inject Dependencies

Try not to hard code dependencies, use 'message' the method or the class really cares:

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    ...
  end

  # hard code the Wheel in gear_inches method
  # call this method with Gear.new(52, 11, 26, 1.5).gear_inches#
  def gear_inches
    ratio * Wheel.new(rim, tire).diameter
  end
end
```

```ruby
class Gear
  attr_reader :chaining, :cog, :wheel

  def initialize(chainring, cog, wheel)
    ...
  end

  # Moving the creation of Wheel outside the Gear class, decoupling the Wheel and Gear class.
  # call this method with Gear.new(52, 11, Wheel.new(26, 1.5)).gear_inches
  def gear_inches
    ratio * wheel.diameter
  end
end
```

### Isolate Dependencies

Prepare a space to isolate dependencies within given zone

#### Isolate Instance creation

1. Create Dependencies when initialize

```ruby
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, rim, tire)
    ...
    @wheel = Wheel.new(rim, tire)
  end

  def gear_inches
    ratio * wheel.diameter
  end
end
```

2. Isolate the Dependency in method

```ruby
class Gear
  attr_reader :chainring, :cog, :rim, :tire

  def initialize(chainring, cog, rim, tire)
    ...
  end

  def gear_inches
    ratio * wheel.diameter
  end

  private

  def wheel
    @wheel ||= Wheel.new(rim, tire)
  end
end
```

#### Isolate Vulnerable External Messages

external messages = messasges that are 'send to someone other than `self`'

This refactor depends on whether this extraction is worth to be DRYed out.

```ruby
def gear_inches
  ratio * diameter
end

private

def diameter
  wheel.diameter
end
```

### Remove Augument-Order Dependencies

Use args Hash to initialize the object, keys does not has order issue.

Explicity define default or raise argument error in case the argument does not provide required values.

```ruby
# this line will fall back to default value 18 whenever the args[:cog] are evaluated to false,
# in other word, if args[:cog] are assigned to 'false' then @cog will be assigned to 18
@cog = args[:cog] || 18

# To use the provided value whenever the key existed, can use fetch
# the default value is assigned only when the fetched key does not exist
@cog = args.fetch(:cog, 18)

# have a default hash to be merged is another solution
def default_args
  { cog = 18 }
end

def initialize(args)
  args = default_args.merge(args)
  ...
  @cog = args[:cog]
end
```

If the object belongs to external libs, then constructing a `LibObjectWrapper` for that object that use Hash parameter to initialize `LibObject` instead of a ordered argument array might worth the efforts. This is called factory patterns. An object whose purpose is to create other objects is a factory; the word factory implies nothing more, and use of it is the most expedient way to communicate this idea.

## Manageing Dependency Direction

Object should depend on the Class that tends to have less changes if possible.

- Some classes are more likely than others to have changes in requirements.
- Concrete classes are more likely to change than abstract classes.
- Changing a class that has many dependents will result in widespread consequences.

```ruby
# Gear depends on Wheel
class Gear
  attr_reader :chainring, :cog, :wheel

  def initialize(chainring, cog, rim, tire)
    ...
    @wheel = Wheel.new(rim, tire)
  end

  def gear_inches
    ratio * wheel.diameter
  end
end
```

```ruby
# Wheel depends on Gear
class Gear
  attr_reader :chainring, :cog

  def initialize(chainring, cog)
    ...
  end

  def gear_inches(diameter)
    ratio * diameter
  end
end

class Wheel
  attr_reader :rim, :tire, :gear
  def initialize(rim, tire, chainring, cog)
    ...
    @gear = Gear.new(chainring, cog)
  end

  def diameter
    rim + (tire * 2)
  end

  def gear_inches
    gear.gear_inches(diameter)
  end
end
```

> P.57
>
> Dependency management is core to creating future-proof applications. Injecting dependencies creates loosely coupled objects that can be reused in novel ways. Isolating dependencies allows objects to quickly adapt to unexpected changes. Depending on abstractions decreases the likelihood of facing these changes.
>
> The key to managing dependencies is to control their direction. The road to maintenance nirvana is paved with classes that depend on things that change less often than they do.

# Chapter 4 - Creating Flexible Interfaces

There are two kinds of interface:

- Classes implement methods, some of those methods are intended to be used by others and these methods make up its public interface.
- A set of messages where the messages themselves define the interface. Many different classes may, as part of their whole, implement the methods that the interface requires. It’s almost as if the interface defines a virtual class; that is, any class that implements the required methods can act like the interface kind of thing.

This chapter is handling first kind of interface, left the last kind being handled in Chapter 5 - Reducing Costs with Duck Typing.

## Defining interface

**Public Interfaces** The methods that make up the public interface of your class comprise the face it presents to the world. They:

- Reveal its primary responsibility
- Are expected to be invoked by others
- Will not change on a whim
- Are safe for others to depend on
- Are thoroughly documented in the tests

**Private Interfaces** All other methods in the class are part of its private interface. They:

- Handle implementation details
- Are not expected to be sent by other objects
- Can change for any reason whatsoever
- Are unsafe for others to depend on
- May not even be referenced in the tests

### Using Sequence Diagrams

Illustrating out the message between classes in sequence diagrams can help to visualize the interaction between classes' interfaces.

### Asking for “What” Instead of Telling “How”

Message being sent to object is better to ask result instead of instruct how to perform the action, for instance, `Customer` class does not need to instruct `place_order`, `deduct_stock`, `process_payment`, `generate_invoice` to `Store`. `Customer` will be better off by `checkout` with his `Cart` instance and get a `Invoice` back.

### Seeking Context Independence

The things that `DomainClass` knows about other objects make up its context. Context is a coat that `DomainClass` wears everywhere; any use of `DomainClass`, be it for testing or otherwise, requires that its context be established. Deduce the context of `DomainClass` can help to reuse the class and easier to be test with.

### Trusting Other Objects

`DomainClass` just need to provide `what` they need and `what` they are, then trust the `result` that message receiver respond with.

> P.74
>
> This blind trust is a keystone of object-oriented design. It allows objects to collab- orate without binding themselves to context and is necessary in any application that expects to grow and change.

### Using Messages to Discover Objects

If you find out current class is handling message that might not be single responsibility enough, then there's a chance that you need a new class to handle this message. For instance, a mailer shall not handle the condition whether not to send out this email, so if there's need to filter out the condition that skipping the mailing action, then you might need to have a new class that picks up this responsibility instead of coding the skip mailing logic within mailer class.

## Writing Code That Puts Its Best (Inter)Face Forward

### Create Explicit Interfaces

Every time you create a class, declare its interfaces. Methods in the public interface should

- Be explicitly identified as such
- Be more about what than how
- Have names that, insofar as you can anticipate, will not change
- Take a hash as an options parameter

Use of `public`, `protected`, `private` keywords serves two distinct purposes. First, they indicate which methods are stable and which are unstable. Second, they control how visible a method is to other parts of your application.

### Honor the Public Interfaces of Others

> P.78
>
> Do your best to interact with other classes using only their public interfaces. ... If your design forces the use of a private method in another class, first rethink your design. It’s possible that a committed effort will unearth an alternative; you should try very hard to find one.

If depending on other class's private method is required in your design no matter how hard you try to avoid, isolate the dependencies within safe zone.

## The Law of Demeter

> P.80
>
> The Law of Demeter (LoD) is a set of coding rules that results in loosely coupled objects. Loose coupling is nearly always a virtue but is just one component of design and must be balanced against competing needs. Some Demeter violations are harmless, but others expose a failure to correctly identify and define public interfaces.
>
> Demeter is often paraphrased as “only talk to your immediate neighbors” or “use only one dot.”

The risk of violating the LoD is that if there's any change occurs between the chained method's return value, the result of final output might break.

Code like `message.conversation.project.start_project!` will break whenever the implementation of each Class or method call is changed, or when the records does not found in DB. Better avoid a long method chain like this.

To be clear, a method chain like `hash.keys.sort.join` are chaining basic object in Ruby Core that tends to be very stable, this kind of 'violations' would probably not harming the application at all. LoD is more like a recommendation than a restriction.

Something like `message.conversation.project.start_project!` shows that the code is instructing not only 'what' they want, but also 'I know how exactly to get the thing I want'. When this is happening, maybe the caller is appropriate to be responsible to get the result that even though it knows how to get it.

> P.83
>
> Focusing on messages reveals objects that might otherwise be overlooked. When messages are trusting and ask for what the sender wants instead of telling the receiver how to behave, objects naturally evolve public interfaces that are flexible and reusable in novel and unexpected ways.

# Chapter 5 - Reducing Costs with Duck Typing

> P.85
>
> Duck typed objects are chameleons that are defined more by their behavior than by their class. This is how the technique gets its name; if an object quacks like a duck and walks like a duck, then its class is immaterial, it’s a duck.

Use duck typing as a proxy of a group of class that behaves like a duck in the point of the caller's view. If a `Trip` instance requires a `Preparer` that can handle `prepare_trip` message, the `Trip` does not cares about whether the preparer is a `Bus`, `TourGuide`, `Mechanic`.

Something like below occurs in your code, then you might need a duck:

- Case statements that switch on class
- `kind_of?` and `is_a?`
- `responds_to?`

The Trip has to know exactly the Class of preparer and the public method calls to prepare the trip

```ruby
class Trip
  attr_reader :bicycles, :customers, :bus

  def prepare(preparers)
    preparers.each do |preparer|
      case preparer
      when Mechanic
        preparer.prepare_bicycles(bicycles)
      when TourGuide
        preparer.buy_food(customers)
      when Driver
        preparer.gas_up(bus)
        preparer.fill_water_tank(bus)
      end
    end
  end
end
```

```ruby
if preparer.kind_of?(Mechanic)
  preparer.prepare_bycycles(bicycles)
elsif preparer.kind_of?(TourGuide)
  preparer.buy_food(customers)
elsif preparer.kind_of?(Driver)
  preparer.gas_up(bus)
  preparer.fill_water_tank(bus)
end
```

```ruby
if preparer.responds_to?(:prepare_bicycles)
  preparer.prepare_bycycles(bicycles)
elsif preparer.responds_to?(:buy_food)
  preparer.buy_food(customers)
elsif preparer.responds_to?(:gas_up)
  preparer.gas_up(bus)
  preparer.fill_water_tank(bus)
end
```

Changing above code to duck typing will rearrange the public interface between `Trip`, `Mechanic`, `TourGuide`, and `Driver`. Instead of letting `Trip` knows about how all kinds of preparer class and how to let them prepare things, `Trip` only send a `prepare_trip` messages to the preparer, and all preparers has to get correct repsonses, whether asking for more inputs or handling the provided ones.

```ruby
class Trip
  def prepare(preparers)
    preparers.each { |preparer| preparer.prepare_trip(vehicle = bus, customers = customers, bicycles = bicycles) }
  end
end

class Driver
  def prepare_trip(trip_requirement={})
    gas_up(trip_requirement[:vehicle])
    fill_water_tank(trip_requirement[:vehicle])
  end
end

class TourGuide
  def prepare_trip(trip_requirement={})
    buy_food(trip_requirement[:customers])
  end
end

class Mechanic
  def prepare_trip(trip_requirement={})
    prepare_bycycles(trip_requirement[:bicycles])
  end
end
```

Better can share the Duck Type code through Modules.

> P104.
>
> Messages are at the center of object-oriented applications and they pass among objects along public interfaces. Duck typing detaches these public interfaces from specific classes, creating virtual types that are defined by what they do instead of by who they are.
>
> Duck typing reveals underlying abstractions that might otherwise be invisible. Depending on these abstractions reduces risk and increases flexibility, making your application cheaper to maintain and easier to change.

# Chapter 6 - Acquiring Behavior Through Inheritance

> P105.
>
> Inheritance is, at its core, a mechanism for automatic message delegation. It defines a forwarding path for not-understood messages.

> P112.
>
> objects receive messages. No matter how complicated the code, the receiving object ultimately handles any message in one of two ways. It either responds directly or it passes the message on to some other object for a response.
Inheritance provides a way to define two objects as having a relationship such that when the first receives a message that it does not understand, it automatically forwards, or delegates, the message to the second.
>
> Ruby has single inheritance. A superclass may have many subclasses, but each subclass is permitted only one superclass.

Every new class defined in Ruby will be automatically inherit from `Object`, Ruby automatically forward message to subclass chain in search of a matching method implementation.

> P113.
>
> subclasses are everything their superclasses are, plus more. An instance of String is a String, but it’s also an Object.

> P117.
>
> Subclasses are specializations of their superclasses. For inheritance to work, two things must always be true. First, the objects that you are modeling must truly have a generalization–specialization relationship. Second, you must use the cor- rect coding techniques.

Create an abstract superclass that contains the common behavior of the subclasses, but the initialization will be from the fully featured subclasses, not the abstract superclass.

> P118.
>
> Abstract classes exist to be subclassed. This is their sole purpose. They provide a common repository for behavior that is shared across a set of subclasses—subclasses that in turn supply specializations.

Extracting the common behavior to an abstract class is easier to demote the specific behavior down to subclass from an inflated class. The 'push-everything-down-and-then-pull-some-things-up' strategy is an important part of extracting abstract superclass refactoring.

> P122.
>
> If you begin this refactoring with that first version of Bicycle, attempting to isolate the concrete code and push it down to RoadBike, any failure on your part will leave dangerous remnants of concreteness in the superclass. However, if you start by moving every bit of the Bicycle code to RoadBike, you can then carefully identify and promote the abstract parts without fear of leaving concrete artifacts.

## Template method pattern

Defining a basic structure in the superclass and sending messages to acquire subclass-specific contributions.

To prevent the subclass not implementing all required template methods. Any class that uses the template method pattern must supply an implementation for every message it sends, a simple exception raise could done this job.

> P129.
>
> Creating code that fails with reasonable error messages takes minor effort in the present but provides value forever. Each error message is a small thing, but small things accumulate to produce big effects and it is this attention to detail that marks you as a serious programmer.

```ruby
class Bicycle
  attr_reader :size, :chain, :tire_size

  def initialize(args={})
    @size = args[:size]
    @chain = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size
  end

  def default_chain
    '10-speed'
  end

  def default_tire_size
    raise NotImplementedError, "This #{self.class} cannot respond to:"
  end
end

class RoadBike < Bicycle
  # ...
  def default_tire_size
    '23'
  end
end

class MountainBike < Bicycle
  # ...
  def default_tire_size
    '2.1'
  end
end
```

## Managing Coupling between Superclasses and subclasses

When a subclass is implementing a method that send `super` to its superclass, it implies that the subclass is knowing things about its superclass.

> P132.
>
> Knowing things about other classes, as always, creates dependencies and dependencies couple objects together. The dependencies in the code above are also the booby traps; both are created by the sends of super in the subclasses.

> P134.
>
> forcing a subclass to know how to interact with its abstract superclass causes many problems. When a subclass sends super it’s effectively declaring that it knows the algo- rithm; it depends on this knowledge. If the algorithm changes, then the subclasses may break even if their own specializations are not otherwise affected.

### Using hook messages to decoupling subclasses

```ruby
class Bicycle
  def initialize(args={})
    @size = args[:size]
    @chain = args[:chain] || default_chain
    @tire_size = args[:tire_size] || default_tire_size

    post_initialize(args)
  end

  def post_initialize(args)
    nil
  end
end

class RoadBike < Bicycle
  def post_initialize(args)
    @tape_color = args[:tape_color]
  end
end
```

Provide a hook method to subclass can remove the dependencies that letting subclass knowing what is implemented in superclass.

> P135.
>
> RoadBike is still responsible for what initialization it needs but is no longer responsible for when its initialization occurs. This change allows RoadBike to know less about Bicycle, reducing the coupling between them and making each more flexible in the face of an uncertain future. RoadBike doesn’t know when its post_initialize method will be called and it doesn’t care what object actually sends the message. Bicycle (or any other object) could send this message at any time, there is no requirement that it be sent during object initialization.

Using the hook method on the method can looks like this:

```ruby
class Bicycle
  # ...
  def spares
    { tire_size = tire_size,
      chain = chain }.merge(local_spares)
  end

  def local_spares # hook for subclasses to override
    {}
  end
end

class RoadBike < Bicycle
  # ...
  def local_spares
    { tape_color = tape_color }
  end
end
```

# Chapter 7 - Sharing Role Behavior with Modules

> P142.
>
> Some problems require sharing behavior among otherwise unrelated objects. This common behavior is orthogonal to class; it’s a `role` an object plays. Many of the roles needed by an application will be obvious at design time, but it’s also common to discover unanticipated roles as you write the code.

> P143.
>
> Many object-oriented languages provide a way to define a named group of methods that are independent of class and can be mixed in to any object. In Ruby, these mix-ins are called modules. Methods can be defined in a module and then the module can be added to any object.

Module and Inheritance is a `behaves-like-a` versus `is-a` difference, each choice has distinct consequences.

## Method Looking Up Path

When a method is called on an object, the lookup path of that methods has principle as below, take a `mountain_bike = MountainBike.new` instance as example:

- Check the method defined by Singleton Class in `mountain_bike` (Methods defined only in this `mountain_bike` instance)
- Check module extended into the Singleton Class (Methods defined in modules with which this `mountain_bike` instance has been extended)
- Check method defined within the object's own class (Method defined in `MountainBike` class)
- Check module included in the object's class (Methods defined in modules included in class `MountainBike`)
- Check method defines within the object's superclass (Method defined in `Bike` class)
- Check module included in the object's superclass (Methods defined in modules included in class `Bike`)

## Recognizing the antipatterns

- if an object uses a variable with a name like `type` or `category` to determine what message to send to self, this implies the code has to be changed whenever there's new `type` or `category` being added.
- when a sending object checks the class of a receiving object to determine what message to send, there's possible that we have a duck type here.

## Respect the Contract

> P.160
>
> Subclasses agree to a contract; they promise to be substitutable for their superclasses. Substitutability is possible only when objects behave as expected and subclasses are expected to conform to their superclass’s interface. They must respond to every message in that interface, taking the same kinds of inputs and returning the same kinds of outputs. They are not permitted to do anything that forces others to check their type in order to know how to treat them or what to expect of them.

### Liskov Substitution Principle (LSP)

> Let q(x) be a property provable about objects x of type T. Then q(y)
should be true for objects y of type S where S is a subtype of T.

Checkout Chinese explaination for LSP [here](http://teddy-chen-tw.blogspot.tw/2012/01/4.html)

## Decouple Classes

> P.161
>
> Avoid writing code that requires its inheritors to send `super`, use hook messages to allow subclasses to participate while absolving them of responsibility for knowing that abstract algorithm.

> P162.
>
> Shallow, narrow hierarchies are easy to understand.
> Shallow, wide hierarchies are slightly more complicated.
> Deep, narrow hierarchies are a bit more challenging and unfortunately have a natural tendency to get wider, strictly as a side effect of their depth.
> Deep, wide hierarchies are difficult to understand, costly to maintain, and should be avoided.

# Chapter 8 - Combining Objects with Composition

Use an association like relation to let an instance of `Users` class act like an `array` of `User` instance, is like compose the `Users` with lots of `User` that behaves like `User`, these `User` does not requires to be all the same class implementation, it just needs to be able to response the message that `Users` might want to passing to its every `user` component.

To construct a class to act like an `Array` in Ruby, the convinient way is to include `Enumerable` module into the Class `Users` and define `each` method for the `Users` class so that other methods defined in `Enumerable` are able to follow with.

> P.183
>
> Delegation creates dependen- cies; the receiving object must recognize the message and know where to send it. Composition often involves delegation but the term means something more. A composed object is made up of parts with which it expects to interact via well-defined interfaces. Composition describes a has-a relationship. Meals have appetizers, uni- versities have departments, bicycles have parts. Meals, universities, and bicy- cles are composed objects. Appetizers, departments, and parts are roles. The composed object depends on the interface of the role.
>
> This leaves a gap in the definition that is filled by the term aggregation. Aggregation is exactly like composition except that the contained object has an independent life. Universities have departments, which in turn have pro- fessors. If your application manages many universities and knows about thousands of professors, it’s quite reasonable to expect that although a depart- ment completely disappears when its university goes defunct, its professors continue to exist. The university–department relationship is one of composition (in its strictest sense) and the department–professor relationship is aggregation.

## Deciding Between Inheritance and Composition

### Pros and Cons of Inheritance

Well organized inheritance structure is easy to extend and maintain, adding new subclass to an existing hierarchy requires no changes to existing code.

The downside of the inheritance then becomes the subclass might not want all of the superclass's attributes and behavior. And messy inheritance structure makes the code less reusable as it supposed to be.

### Pros and Cons of Composition

Object that participate in composition are small, structurally independent, and have well-defined interfaces. This allows their seamless transition into pluggable, interchangeable components. Well-composed objects are therefore easily usable in new and unexpected contexts.

On the other hand, a composed object relies on its many parts.

> P.187
>
> The benefits of structural independence are gained at the cost of automatic mes- sage delegation. The composed object must explicitly know which messages to delegate and to whom. Identical delegation code may be needed by many different objects;
composition provides no way to share this code.

- Use Inheritance for is-a Relationships
- Use Duck Types for behaves-like-a Relationships
- Use Composition for has-a Relationships

# Chapter 9 - Designing Cost-Effective tests

The intentions of Tests:

- Finding Bugs
- Supplying Documentation
- Deferring Design Decisions
- Supporting Abstractions
- Exposing Design Flaws

Knowing what to test = The safest way to accomplish this is to test everything just once and in the proper place.

> P.195
>
> The design principles you are enforcing in your application apply to your tests as well. Each test is merely another application object that needs to use an existing class. The more the test gets coupled to that class, the more entangled the two become and the more vulnerable the test is to unnecessarily being forced to change.

Test the response of the incoming message of the receiver object, and only test the outgoing message that has side effect is correctly triggered.

> P.197
>
> These messages are commands and it is the responsibility of the sending object to prove that they are properly sent. Proving that a message gets sent is a test of behavior, not state, and involves assertions about the number of times, and with what arguments, the message is sent.

## Test Approach = BDD versus TDD

> P.199
>
> BDD takes an outside-in approach, creating objects at the boundary of an application and working its way inward, mock- ing as necessary to supply as-yet-unwritten objects. TDD takes an inside-out ap- proach, usually starting with tests of domain objects and then reusing these newly created domain objects in the tests of adjacent layers of code.

## Deleting Unused Interfaces

> P.202
>
> Do not test an incoming message that has no dependents; delete it. You application is improved by ruthlessly eliminating code that is not actively being used. Such code is negative cash flow, it adds testing and maintenance burdens but provides no value. Deleting unused code saves money right now, if you do not do so you must test it.

## Proving the Public Interface

Test on the class's public interface, make sure the class's interface is stable and predictable.

If one class uses another object, then the test will rely on creating classes that is required to interact with the tested interface, choose the approach that requires less resources to setup those dependencies, in other word, make the test run as fast as you can so that you will be more willing to run the test as frequent as it should be.

## Isolating the Object Under Test

Use module to extract the role that shared among classes, increase the reuse of the test configuration.

```ruby
module PreparerInterfaceTest
  def test_implements_the_preparer_interface
    assert_respond_to(@object, :prepare_trip)
  end
end

class MechanicTest < MiniTest::Unit::TestCase
  include PreparerInterfaceTest
  def setup
    @mechanic = @object = Mechanic.new
  end
  # other tests which rely on @mechanic
end

class TripCoordinatorTest < MiniTest::Unit::TestCase
  include PreparerInterfaceTest
  def setup
    @trip_coordinator = @object = TripCoordinator.new
  end
end

class DriverTest < MiniTest::Unit::TestCase
  include PreparerInterfaceTest
  def setup
    @driver = @object = Driver.new
  end
end
```

> P.223
>
> Defining the PreparerInterfaceTest as a module allows you to write the test once and then reuse it in every object that plays the role. The module serves as a test and as documentation. It raises the visibility of the role and makes it easy to prove that any newly created Preparer successfully fulfills its obligations.

## Creating test doubles that can sense the reasonable failed

Using mock might create a heaven like environment for each tests if all mocks are perfectly return the exact epect values that the test expect, changes on the mocked classes might not get reflected on the test results since the changes on mocked class does not impact on the mock behavior itself.

```ruby
module DiameterizableInterfaceTest
  def test_implements_the_diameterizable_interface
    assert_respond_to(@object, :width)
  end
end

class DiameterDouble
  def diameter
    10
  end
end

# Prove the test double honors the interface this # test expects.
class DiameterDoubleTest < MiniTest::Unit::TestCase
  include DiameterizableInterfaceTest
  def setup
    @object = DiameterDouble.new
  end
end

class GearTest < MiniTest::Unit::TestCase
  def test_calculates_gear_inches
    gear = Gear.new(
      chainring = 52,
      cog = 11,
      wheel = DiameterDouble.new
    )
    assert_in_delta(47.27, gear.gear_inches, 0.01)
  end
end
```

Above `DiameterizableInterfaceTest` ensures the double is expected to respond to `width` message, so the unsync between `DiameterDouble` and the `DiameterizableInterfaceTest` can now expose the defect of current code / test.

and when we fix the Double to have latest `diameter` interface, then the `GearTest` that using the previous `width` interface will failed, hence expose the possible defect existed in the `Gear` class.

## Testing Inherited Code

### Specifying the Inherited Interface

```ruby
module BicycleInterfaceTest
  def test_responds_to_default_tire_size
    assert_respond_to(@object, :default_tire_size)
  end

  def test_responds_to_default_chain
    assert_respond_to(@object, :default_chain)
  end

  def test_responds_to_chain
    assert_respond_to(@object, :chain)
  end

  def test_responds_to_size
    assert_respond_to(@object, :size)
  end

  def test_responds_to_tire_size
    assert_respond_to(@object, :tire_size)
  end

  def test_responds_to_spares
    assert_respond_to(@object, :spares)
  end
end

class BicycleTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  def setup
    @bike = @object = Bicycle.new(tire_size = 0)
  end
end

class RoadBikeTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  def setup
    @bike = @object = RoadBike.new
  end
end
```

### Specifying Subclass Responsibilities

```ruby
module BicycleSubclassTest
  def test_responds_to_post_initialize
    assert_respond_to(@object, :post_initialize)
  end

  def test_responds_to_local_spares
    assert_respond_to(@object, :local_spares)
  end

  def test_responds_to_default_tire_size
    assert_respond_to(@object, :default_tire_size)
  end
end

class RoadBikeTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  include BicycleSubclassTest
  def setup
    @bike = @object = RoadBike.new
  end
end
```

### Confirming Superclass Enforcement

The Bicycle class should raise an error if a subclass does not implement default_tire_size. Even though this requirement applies to subclasses, the actual enforcement behavior is in Bicycle.

```ruby
class BicycleTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  def setup
    @bike = @object = Bicycle.new(tire_size = 0)
  end

  def test_forces_subclasses_to_implement_default_tire_size
    assert_raises(NotImplementedError) { @bike.default_tire_size }
  end
end
```

### Testing Unique Behavior

After testing the shared interface and the configurations with modules, it's time to test the concrete subclass behavior. The concrete behavior is therefore existed in the Concrete Subclass test.

```ruby
class RoadBikeTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  include BicycleSubclassTest
  def setup
    @bike = @object = RoadBike.new(tape_color = ‘red’)
  end

  def test_puts_tape_color_in_local_spares
    assert_equal 'red', @bike.local_spares[:tape_color]
  end
end
```

### Testing Abstract Superclass Behavior

Create a Stub itself subclass from the to-be-tested Superclass to test the Superclass Behavior could decouple especific subclass from the Superclass tests.

```ruby
# Make sure this StubbedBike behaves correctly the class can be checked by
# the BicycleSubclassTest module like the DiameterizableInterfaceTest we did
class StubbedBikeTest < MiniTest::Unit::TestCase
  include BicycleSubclassTest
  def setup
    @object = StubbedBike.new
  end
end

class StubbedBike < Bicycle
  def default_tire_size
    0
  end

  def local_spares
    { saddle = 'painful' }
  end
end

class BicycleTest < MiniTest::Unit::TestCase
  include BicycleInterfaceTest
  def setup
    @bike = @object = Bicycle.new(tire_size = 0)
    @stubbed_bike = StubbedBike.new
  end

  def test_forces_subclasses_to_implement_default_tire_size
    assert_raises(NotImplementedError) { @bike.default_tire_size }
  end

  def test_includes_local_spares_in_spares
    assert_equal @stubbed_bike.spares, { tire_size = 0,
                                         chain = '10-speed',
                                         saddle = 'painful' }
  end
end
```
