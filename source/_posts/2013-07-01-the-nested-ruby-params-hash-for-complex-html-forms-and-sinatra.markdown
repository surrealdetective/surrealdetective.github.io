---
layout: post
title: "The Nested Ruby Params Hash for Complex HTML Forms and Sinatra"
date: 2013-07-01 23:02
comments: true
categories: 
---

##Showcasing the Normal, Unaltered Use of the Params Hash

The params hash is a ["rack"](http://en.wikipedia.org/wiki/Rack)  object that is accessible through Sinatra's request object. This params hash stores information posted by html forms in the form of -you guessed it - a hash, which contains a key-value pair. The key of this pair defaults to the value of the ```<input>```'s name="attribute". Meanwhile, the info that the user submits is the value of that key-value pair. 

```html
<form action='/' method="POST">
     <label>Location</label>
          <input name="location"/>
     <input type="submit">Submit</input>
```

For the above html, you can access the user's entry in app.rb within the ```post '/' do``` route:

```ruby
post '/' do
  @location = params[:name]
  "Ain't #{@location} grand?"
end
```

If I entered "New York City" into the ```<input name="location"/>```, then ```@location = "New York City"```. 
Similarly, if there was another input, ```<input name="arrival_date">```, where I entered "June," then the params hash would have two key-value pairs.

```
params == {:location => "New York City", :arrival_date => "June"}
```

##Kick it up a Notch with a Nested Params Hash 

Amazingly, you can modfiy the data structure of the params hash by carefully altering the name attribute of ```<input>```. For example, you can make the params hash contain an array of hashes. Because this process is slightly complex, I will first show off what's possible with an example, then list further below the key points for understanding how to do it.

```html
<h1>List of Games with Name, Difficulty, and Number of Players</h1>
<form action='/games' method="POST">
  <% 5.times do %>
    <fieldset>
      <li>
        <label>Name</label>
        <input ... name="game[][name]"/></li>
      <li>
        <label>Difficulty</label>
        <input ... name="game[][difficulty]" />
      </li>
      <li>
        <label>Players</label>
        <input ... name="game[][players]" />
      </li>
    </fieldset>
  <% end %>
  <input type="submit"/>
</form>
```

The above form outputs the following params hash:

```ruby 
params == {"game"=>[{"name"=>"Connect 4", "difficulty"=>"Easy", "players"=>"2"}, {"name"=>"Go Fish", "difficulty"=>"Easy", "players"=>"4"}, {"name"=>"Monopoly", "difficulty"=>"Easy", "players"=>"4"}, {"name"=>"Chess", "difficulty"=>"Hard", "players"=>"2"}, {"name"=>"Settlers of Catan", "difficulty"=>"Medium", "players"=>"4"}]}
```

As you can see, the params hash's top-layer has only one key-value pair, with "game" as its key. If you look at the html form, this initially makes sense because game is repeated as the top-layer for all three inputs. 

As programmers, we then have a choice of modifying the value of this pair from a string into something else. If we want to turn the pair's value into an array, then we write an empty array as above. If we want to make the value a hash, then we surround the attribute in square brackets.

Notice that each hash in the array contains 3 key-value pairs. Just like the first example with "arrival_date" and "location," adding a key-value pair on the same layer will add a key-value pair to that same hash - not create another hash inside a hash. 

At this point, your brain might be wheeling a bit. Hopefully these rules below will help you clarify how to alter the params hash through html forms.

##Primer for Understanding Params

1. Realize that because the params hash stores form data, the user's entry will always be the value of a key-value pair.


2. If you think of params hash as having a pointer to what will be returned, then everytime you add an extra layer, the data type of the extra layer - be it array or hash, will be the return of the formerly-lowest layer. 

    * In other words, ```<input name="game[][name][]" />``` sets the value of the ```:name=>value``` pair to an array, although this would break your app because according to rule 1, the user's entry must be the value of a key-value pair, which isn't the case here. 

    * Therefore you must end it with a key-value pair like ```<input name="game[][name][][syllables]" />```. Apologies for the quirky example, but here the user would input the number of syllables in the game's name, of which there are many: such as "ping pong," "table tennis," and "pong."

3. By convention, you leave the top-layer of the params hash as being without square brackets, despite the fact that it is a key-value pair inside the params hash.

4. When in doubt, practice using the following method in irb:

```ruby
require 'rack'
def p(params) 
 Rack::Utils.parse_nested_query(params)
end
```

and use the following syntax: ```p("game[][name][][syllables]" )```

Now as a challenge, can you decipher what the above params hash looks like?


Answer:
```params == {:game => [{:name=> [ {:syllables=> value} ] }] }```
