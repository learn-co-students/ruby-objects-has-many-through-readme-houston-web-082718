# Ruby Objects Has Many Through

## Outline

Objects can relate directly to each other when one object contains a reference to another object.

A song can belong to an artist by containing an Artist instance in the property `artist`

song.artist = Artist.new

Artists have many songs by having a property songs that is a collection of all the songs instance that belong to the artist.

artist.songs = [Song.new]

What if songs also belonged to another object like a Genre.

song.genre = Genre.new

entity diagram of the relationships with songs belonging to artist and genres.

How would we figure out all the genres that an artist has created?

We know we can find all the songs that belong to an artist and we know that each song has a genre. Aren't the genres of an artist defined by collecting all the genres of all the artists songs? How might we implement that relationship where artists have many genres not directly but through their songs?

First, let's build an instance method genres on artists.

We said we need to find all the artists songs and then collect the individual genres of each of their songs.

`self.songs.collect{|s| s.genre}`

That's called a has many through relationship. the genres instance method simply queries the relating objects for deeper relationships.

Artists could have a literal property, genres, and store the genres of each of their songs in their own instance variable @genres, but then we'd have to maintain that data in our code. Going through songs means that as long as songs properly relate to artists and genres, we don't need to maintain any more data state.

Let's implement the opposite of this relationship for genres.

`self.songs.collect{|s| s.artist}`

Let's make it unique though.

Through relationships provide methods that use existing relationships to discover deeper relationships instead of storing the state of the relationships.
