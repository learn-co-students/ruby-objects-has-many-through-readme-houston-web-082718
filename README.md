# Ruby Object Relations: Has-Many Through

## Introduction

We've seen how objects can be related to one another directly, when one object contains a reference to another. 

An individual song belongs to an artist, for example. That relationship is implemented by giving a `Song` instance an `artist` `attr_accessor`. Then we set that `artist` attribute to an instance of the `Artist` class. 

An individual artist, conversely, has many songs. We implement this relationship by giving artists a `song` property that is set equal to an array of song instances. We add song instances to that array via an instance method on the `Artist` class, `#add_song`. 

Let's take a look:

```ruby
class Song
  attr_accessor :name, :artist
end
```

```ruby
class Artist
  attr_accessor :name

  def initialize
    @songs = []
  end
  
  def add_song(song)
    @songs << song
    song.artist = self
  end
end
```

Notice that in the `#add_song` method, a song is passed in as an argument and added to an artist's `@songs` array. At the same time, that song's `artist` attribute is set equal to the instance of the artist that the `#add_song` is being called on, referenced by the `self` keyword. 

This is the "has many"/"belong to" association. A song belongs to an artist and an artist has many songs. This relationship is direct. It is enacted with methods on our `Song` and `Artist` instances that directly reference and interact with objects of the other class. 

However, in the real-world, different entities can be connected to one another *indirectly* as well as directly. For example:

* A family tree in which you are directly related to your parents, and indirectly related to your grandparents. In this case, you are related to your grandparents *through* your parents. 
* A company that offers a network of doctors to their employees *through* the company's insurance program. 
* A user on a popular media sharing site can have many "likes", that occur *through* the pictures he or she posts. 

These are just a few examples of the kind of indirect relationships that we may need to model in our programs. 

In this lesson, we'll build out just such a relationship, using our music app domain model. 

## Our Domain Model

In our example, we already have a has many/belongs to relationship between songs and artists. Let's add in another model to step up the complexity of our associations. 

In the real world, individual songs belong to a genre. For example, you could classify Jay-Z's "99 Problems" as a rap song but his song "Crazy in Love", with Beyonce, is more of a pop song. 

Let's give our `Song` instances the ability to belong to a genre:

```ruby
class Song
  attr_accessor :name, :artist, :genre 
end
```

Let's build a `Genre` class, so that we can associate individual songs to complex genre objects that can contain other information pertinent to a given genre.

```ruby
class Genre
  attr_accessor :name
  
  def initialize(name)
    @name = name
  end
end
```

Great! Now we can associate a `Song` to a `Genre`. At what point should this association be created? When a song is created, it can be categorized as a particular genre. So, let's create an `#initialize` method for our `Song` class that takes in an argument of a song name and a genre. 

```ruby
class Song
  attr_accessor :name, :artist, :genre
  
  def initialize(name, genre)
    @name = name
    @genre = genre
  end
end
```

With the above code, we can assign a `Song` instance a given genre:

```ruby
rap = Genre.new("rap")
ninety_nine_problems = Song.new("99 Problems", rap)

ninety_nine_problems.genre
  # => rap
ninety_nine_problems.genre.name
  # => "rap"
```

Let's take a step back and think about the web of relationships that exists between our three classes, or models. We have an `Artist` class that produces individual artists that have many songs. We have a `Song` class that produces individual songs that belong to an artist *and* belong to a genre.

![](http://readme-pics.s3.amazonaws.com/Screen%20Shot%202015-11-03%20at%2012.23.17%20PM.png) 

## The "has many through" Relationship

Here we can see that an artist has many songs and that each song belongs to one artist and one genre. This diagram makes it clear to us that an artist *does* have a connection to genres. That connection exists *through* the many songs that an artist owns. **This is the "has many through" relationship**. A given object has many of another type of object. That second object belongs to (or has many) of a third type of object. Therefore, the first object has many of the third object as well. 

So, if artists do in fact have many genres, *through* songs, how can we ask a given artist to show us the genres that it is associated with? An artist has a genre through the songs that it has created, so in order to ask an artist about it's genre, we have to go through that artist's songs. 

We'll need to collect all of the songs of a given artist and *then* collect the genre associated to each of those songs. 

Let's build an instance method on the `Artist` class to accomplish that. 

### Building on the "has many through" Relationship

We'll call our instance method `#genres`. This method will collect all of the genres of all of the songs of a given artist. 

```ruby
class Artist
  attr_accessor :name
  
  def initialize(name)
    @songs = []
  end
  
  def add_song(song)
    @songs << song
    song.artist = self
  end
  
  def songs
    @songs
  end
  
  def genres
    self.songs.collect do |song|
      song.genre
    end
  end
end
```

The `#genres` method iterates over the `@songs` array, stored in the `#songs` instance method, and calls the `#genre` method on each song in order to collect the genre that is associated to that song. The return value of the `#genres` method should be an array of genre objects.  

Let's see it in action:

```ruby
jay_z = Artist.new("Jay-Z")

rap = Genre.new("rap")
pop = Genre.new("pop")

ninety_nine_problems = Song.new("99 Problems", rap)
crazy_in_love = Song.new("Crazy in Love", pop)

jay_z.add_song(ninety_nine_problems)
jay_z.add_song(crazy_in_love)

jay_z.genres
  # => [rap, pop]
```

By simply querying an artist's songs for their genre, we use existing relationships, in this case the relationship between an artist and a song and a song and a genre, to discover deeper relationships. 

### Completing the Relationship

Right now, an artist can tell us about its songs and about its genres. But, a genre can't tell us about its songs and its artists. Let's fix that now. 

The first thing we want to do is build the direct relationship between a song and a genre. A song already belongs to a genre. Let's build the inverse of that relationship, the "has many" side. A genre should have many songs:

```ruby
class Genre
  attr_accessor :name
  
  def initialize(name)
    @name = name
    @songs = []
  end
end
```

Now, when we initialize a new genre, we do so with an empty collection of songs. Let's build a method, `#add_song`, that adds a new song to the given genre's collection:


```ruby
class Genre
  attr_accessor :name
  
  def initialize(name)
    @name = name
    @songs = []
  end
  
  def add_song(song)
    @songs << song
  end
end
```

Now, we can refactor our `Song` class such that when a new song is instantiated it gets associated to a genre *and* the given genre adds that song to it's collection. This is similar to our `add_song` method from the `Artist` class:

```ruby
class Song
  attr_accessor :name, :artist, :genre
  
  def initialize(name, genre)
    @name = name
    @genre = genre
    genre.add_song(song)
  end
end
```

Now let's see it in action:

```ruby
rap = Genre.new("rap")

ninety_nine_problems = Song.new("99 Problems", rap)
lucifer = Song.new("Lucifer", rap)

rap.songs
  # => [ninety_nine_problems, lucifer]
```

Now a song knows about the genre it belongs to a genre knows about the many songs that it has. 

Let's put the finishing touches on our "has many through" relationship and tell our genres how to show us the artists they are associated to *through* the songs it has. 

```ruby
class Genre
  attr_accessor :name
  
  def initialize(name)
    @name = name
    @songs = []
  end
  
  def add_song(song)
    @songs << song
  end
  
  def artists
    @songs.collect do |song|
      song.artist
    end
  end
end
```

The `Genre#artist` method iterates over the genre's `@songs` collection, calls the `#artist` method on each song object and collects the resulting artists. 


## Conclusion: The Advantages of the "has many through" relationship

Why associate artists to genre objects *through* songs? By associating songs to genres, we are not only reflecting the real world situation that our program is meant to model, we are creating clean and re-usable code. Without the song/genre association, you'll find that you have to add a given song to an artist's list of songs and then separately add a genre to that artist. 

With our "through association", as long as we have properly associated a song to a given genre and that same song to a given artist, our connection between an artist and his or her genres will automatically follow. 

<a href='https://learn.co/lessons/ruby-objects-has-many-through-readme' data-visibility='hidden'>View this lesson on Learn.co</a>
