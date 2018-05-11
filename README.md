# Ruby Object Relations: Has-Many Through

## Objectives

* Understand Has-Many-Through relationships
* Construct indirect relationships between models (Customers, Waiters and Meals)
* Explore the concept of a 'joining' model
* Continue to write code using a Single Source of Truth


## Introduction

We've seen how objects can be related to one another directly when one object contains a reference to another. This is the "has many"/"belongs to" association, and is a direct relationship. For example, an artist may have many songs or a book might have many reviews.

In addition to these one-to-one and one-to-many relationships, there are relationships that need something to join them together. For example, you don't need to have a direct relationship with the pilot of a flight you're on. You have a relationship with that flight (you're taking the flight after all), and and the pilot has a relationship with the flight (they're flying it). So you have a relationship to that pilot _through_ the flight.

If you take more than one flight, you'll have these kinds of relationships with more than one pilot, all still using your ticket as the middle man. The way we refer to this is that each customer _has many_ pilots _through_ tickets.

Check out some more examples:

* A company that offers a network of doctors to their employees _through_ the company's insurance program
* A user on a popular media sharing site can have many "likes", that occur _through_ the pictures they post
* A Lyft driver that you are connected to _through_ the rides you've taken with them

In this lesson, we'll build out just such a relationship using waiters, customers, and meals. A customer has many meals, and a customer has many waiters through those meals. Similarly, a waiter has many meals, and has many customers through meals.


## Building out our Classes

Let's start by building out the `Customer` class and `Waiter` class.  We want to make sure, when building out classes, that there's something to store each instance.  That is to say: the `Customer` class should know about every `customer` instance that gets created.

```ruby
class Customer
  attr_accessor :name, :age

  @@all = []

  def initialize(name, age)
    @name = name
    @age = age
    @@all << self
  end

  def self.all
    @@all
  end

end
```

As you can see, each `customer` instance has a name and an age. Their name and age are set upon initialization, and because we created an attribute accessor for both, the customer is able to change their name or age. If we wanted to limit this ability to read only, we would create an attribute reader instead. The `Customer` class also has a class variable that tracks every instance of `customer` upon creation.

```ruby
class Waiter

  attr_accessor :name, :yrs_experience

  @@all = []

  def initialize(name, yrs_experience)
    @name = name
    @yrs_experience = yrs_experience
    @@all << self
  end

  def self.all
    @@all
  end

end
```

Each instance of the `Waiter` class has a name and an attribute describing their years of experience. Just like the `Customer` class, the `Waiter` class has a class variable that stores every `waiter` instance upon initialization.

## The "has many through" Relationship

In real life, as a customer, each time you go out to eat, you have a different meal. Even if you order the same exact thing in the exact same restaurant, it's a different instance of that meal. So it goes without saying that a customer can have many meals.

It's a safe bet to assume that unless you only eat at one very small restaurant, you'll have many different waiters as well. Not all at once of course, because you only have one waiter per meal. So it could be said that your relationship with the waiter is through your meal. The same could be said of the waiter's relationship with each customer.

That's the essence of the `has many through` relationship.

## How Does That Work in Code?

Great question! The way we're going to structure this relationship is by setting up our `Meal` class as a 'joining' model between our `Waiter` and our `Customer` classes. And because we're obeying the `single source of truth`, we're going to tell the `Meal` class to know all the details of each `meal` instance. That includes not only the total cost and the tip (which defaults to 0), but also who the `customer` and `waiter` were for each meal.

```ruby
class Meal

  attr_accessor :waiter, :customer, :total, :tip

  @@all = []

  def initialize(waiter, customer, total, tip=0)
    @waiter = waiter
    @customer = customer
    @total = total
    @tip = tip
    @@all << self
  end

  def self.all
    @@all
  end
end
```

That looks great! And even better, it's going to give both the `customer` and `waiter` instances the ability to get all the information about the meal that they need without having to store it themselves. Let's build some methods.

## Building on the relationship

If you take a look at our `customer` right now, they aren't capable of doing much. Let's change that and give them the ability to create a `meal`. To do this, they'll need to take in an instance of a `waiter` and supply the `total` and `tip`, which we'll have default to 0 here as well.

```ruby
  def new_meal(waiter, total, tip=0)
    Meal.new(waiter, self, total, tip)
  end
```

As you can see, we don't need to take `customer` in as an argument, because we're passing in `self` as reference to the current instance of customer. This method will allow us to create new meals as a `customer`, and automatically associates each new `meal` with the `customer` that created it. We can do the same thing for the `Waiter` class.

```ruby
  def new_meal(customer, total, tip=0)
    Meal.new(self, customer, total, tip)
  end
```

Great! Now we can create `waiters`, `customers` and `meals` to our heart's content.

```ruby
  sam = Customer.new("Sam", 27)
  pat = Waiter.new("Pat", 22)
  alex = Waiter.new("Alex", 35)
  sam.new_meal(pat, 50, 10)
  sam.new_meal(alex, 20, 3)
  pat.new_meal(sam, 30, 5)
```

## Completing the relationship

This is awesome, but it isn't done yet! We need a way for our `customer` and `waiter` instances to get information about the meals they've created.

In real life, after all, a customer will want to know about the meals they've had. And you can bet a waiter will want to check out which customers tipped well. To get that information, we're going to consult the `Meal` class.

In plain English, the customer is going to look at all of the meals, and then select only the ones that belong to them. This is pretty similar to how we're going to write it in code.

```ruby
  def meals
    Meal.all.select do |meal|
      meal.customer == self
    end
  end
```

Boom. We're iterating through every instance of `Meal` and returning only the ones where the meal's `customer` matches the current `customer` instance. If our customer, Sam, wants to know about all of their meals, all we need to do is call the `#meals` method on that instance.

`sam.meals` will get an an array of all of Sam's meals, but what if we now want a list of all of the waiters that Sam has interacted with? We can simply reference the meals array! And since we have a method to get that array already, we can reuse that method.

```ruby
  def waiters
    meals.map do |meal|
      meal.waiter
    end
  end
```

And there you have it, an array of the `waiters` our `customer` has interacted with. From here, we can get the names of those waiters, average years of experience, or whatever else we want to find out. And we did it all without having to store the `waiter` information on the customer - we did it by going _through_ `meals`.

## Conclusion: The Advantages of the "has many through" relationship

Why associate customers to waiter objects _through_ meals? By associating meals to waiters, we are not only reflecting the real world situation that our program is meant to model, but we are also creating clean and re-usable code. Each class only knows about what they specifically need to know about, and we create a single source of truth by keeping our information central in our relationship model.


## Further Practice

Below you'll find all the code for the `Customer` class, including a few new methods, and you already have all the code for the `Meal` class. Try building out the `Waiter` class to match these patterns and then build methods to return the following.

* The average years of experience of all waiters
* A list of the names of customers that a specific waiter has served
* The customer that has tipped a specific waiter the highest
* The average tips for the most experienced waiter and the average tips for the least experienced waiter.

```ruby
class Customer
  attr_accessor :name, :age

  @@all = []

  def initialize(name, age)
    @name = name
    @age = age
    @@all << self
  end

  def self.all
    @@all
  end

  def meals
    Meal.all.select do |meal|
      meal.customer == self
    end
  end

  def waiters
    meals.map do |meal|
      meal.waiter
    end
  end

  def new_meal(waiter, total, tip=0)
    Meal.new(waiter, self, total, tip)
  end

  def new_meal_20_percent(waiter, total)
    tip = total * 0.2
    Meal.new(waiter, total, tip)
  end

  def self.oldest_customer
    oldest_age = 0
    oldest_customer = nil
    self.all.each do |customer|
      if customer.age > oldest_age
        oldest_age = customer.age
        oldest_customer = customer
      end
    end
    oldest_customer
  end

end
```

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/ruby-objects-has-many-through-readme' title='Ruby Object Relations: Has-Many Through'>Ruby Object Relations: Has-Many Through</a> on Learn.co and start learning to code for free.</p>

<p class='util--hide'>View <a href='https://learn.co/lessons/ruby-objects-has-many-through-readme'>Has Many Objects Through</a> on Learn.co and start learning to code for free.</p>
