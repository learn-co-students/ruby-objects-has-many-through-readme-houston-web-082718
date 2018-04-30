# Ruby Object Relations: Has-Many Through

## Introduction

We've seen how objects can be related to one another directly, when one object contains a reference to another. This is the "has many"/"belongs to" association, and is a direct relationship.

However, in the real-world, different entities can be connected to one another _indirectly_ as well as directly. For example:

* A company that offers a network of doctors to their employees _through_ the company's insurance program.
* A user on a popular media sharing site can have many "likes", that occur _through_ the pictures they post.
* A Lyft driver that you are connected to _through_ the rides you've taken with them.

These are just a few examples of the kind of indirect relationships that we may need to model in our programs.

In this lesson, we'll build out just such a relationship using waiters, customers, and meals. A customer has many meals, and a customer has many waiters through those meals. Similarly, a waiter has many meals, and has many customers through meals. We'll dig into that in just a minute, but let's start with the stuff we already know.

## Our Domain Model

Domain models often mimic real life. They don't always have to, but in this case, our model is going to be something that we all know pretty well. Let's start by building out the `Customer` and `Waiter` classes.

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

As you can see, our `Customer` has a name and an age. Their name and age are set upon initialization, and because we created an attribute accessor for both, the customer is able to change their name or age. If we wanted to limit this ability to read only, we would create an attibute reader instead. The Customer class also has a class variable that tracks every instance of `customer` upon creation.

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

Our `Waiter` has a name and an attribute relating to their their years of experience. We could instantiate a waiter with anything we wanted to, but it might be fun to do some math later and see if our customers tip experienced waiters better.

## The "has many through" Relationship

As a customer, each time you go out to eat, you have a different meal. Even if you order the same exact thing in the exact same restaurant, it's a different instance of that meal. So it goes without saying that a customer can have many meals.

It's a safe bet to assume that unless you only eat at one very small restaurant, you'll have many different waiters as well. Not all at once of course, because you only have one waiter per meal. So it could be said that your relationship with the waiter is through your meal. The same could be said of the waiter's relationship with each customer.

That's the essence of the `has many through` relationship.

## How Does That Work in Code?

Great question! The way we're going to implement this relationship is by setting up our `Meal` class as a join between our `Waiter` and our `Customer`. And because we're obeying the `single source of truth`, we're going to tell the `Meal` class to know all the details of each meal. That means not only the total cost and the tip (which defaults to 0), but who the customer and waiter were for each meal.

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

That looks great! And even better, it's going to give both `customer` and `waiter` the ability to get all the information they need to without having to store it themselves. Let's build some methods.

## Building on the relationship

If you take a look at our `customer` right now, they don't really have a lot to do. Let's change that up and give them the ability to create a meal. To do this, they'll need to take in an instance of a `waiter` and supply the `total` and `tip`, which we'll have default to 0 here as well.

```ruby
  def new_meal(waiter, total, tip=0)
    Meal.new(waiter, self, total, tip)
  end
```

As you can see, we don't need to take `customer` in as an argument, because we're passing in `self` to represent the current instance of customer. This method will allow us to create new meals as a customer, and automatically associates each new meal with the customer that creates it. We can do the same thing for `waiter`.

```ruby
  def new_meal(customer, total, tip=0)
    Meal.new(self, customer, total, tip)
  end
```

Great! Now we can create waiters, customers and meals to our heart's content.

```ruby
  sam = Customer.new("Sam", 27)
  pat = Waiter.new("Pat", 22)
  alex = Waiter.new("Alex", 35)
  sam.new_meal(pat, 50, 10)
  sam.new_meal(alex, 20, 3)
  pat.new_meal(sam, 30, 5)
```

## Completing the relationship

This is awesome, but it isn't done yet! We need a way for our `customer` and `waiter` to get the information that they've created. A customer will want to know about the meals they've had after all, and you can bet a waiter will want to check out which customers tipped well. To get that information, we're going to consult the meal class.

In plain English, the customer is going to look at all of the meals, and then select only the ones that belong to them. This is pretty similar to how we're going to write it in code.

```ruby
  def meals
    Meal.all.select do |meal|
      meal.customer == self
    end
  end
```

Boom. We're iterating through every instance of Meal and returning only the ones where the meal's customer matches the current customer instance. If our customer, Sam, wants to know about all of their meals, all we need to do is call the meals method on that instance.

`sam.meals` will get an an array of all of Sam's meals, but what if we now want a list of all of the waiters that Sam has interacted with? We can simply reference the meals array! And since we have a method to get that array already, we can reuse that method.

```ruby
  def waiters
    meals.map do |meal|
      meal.waiter
    end
  end
```

And there you have it, an array of the waiters our customer has interacted with. From here, we can get the names of those waiters, average years of experience, or whatever else we want to find out. And we did it all without having to store the `waiter` information on the customer - we did it by going _through_ `meals`.

## Conclusion: The Advantages of the "has many through" relationship

Why associate customers to waiter objects _through_ meals? By associating meals to waiters, we are not only reflecting the real world situation that our program is meant to model, but we are also creating clean and re-usable code. Each class only knows about what they specifically need to know about, and there are no sloppy arrays storing data that they don't need to.

## Further Practice

Below you'll find all the code for the Customer class, including a few new methods, and you already have all the code for the Meal class. Try building out the Waiter class to match these patterns and then build methods to return the following.

* The average years of experience of all waiters
* A list of the names of customers that a specific waiter has served
* The customer that has tipped a specific waiter the highest

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
