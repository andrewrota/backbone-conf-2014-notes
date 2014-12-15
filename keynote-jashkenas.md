# Keynote
## Jeremy Ashkenas, [@jashkenas](http://www.twitter.com/jashkenas)

### Code review of Backbone.js

**Module Madness**

The first thing you notice in Backbone is how it sets itself up in terms of modules/globals.  He'd prefer this to go away.  Hopefully with ES6 modules that will happen.

Until then we have to dealw ith things like noConflict.  Ideally you don't have two libraries running on the same page, but real world is not ideal.

**Inheritance**

`extend` is a helper function that does the basics of JS prototypal inheritance.  You make an empty child constructor function, stick the prototype on it, make a new one of those (`surrogate`) and use that as the prototype.  This is one of the reasons classes have been discussed for JavaScript, so you don't have to do this kind of dance.  Extend also copies over the properties of the parent class onto the child.  You get a prototype chain for the instance methods and then copies of the properties.  It's added to all the Backbone classes.

**Events**

The heart of Backbone is its Events system.  You can connect your data to your UI directly, but also reactively by listening to events.

First, there is sugar for handling recursive calls.  You can pass a space delimited list of event names or a key value list when you define events.  The eventsApi will recursively call the function you tried to call in the first place to every event you set up.

When you bind a callback to a model it will lazily create the storage where that callback gets stored.

`once` implements itself as an `_.once()` function that can only be called one time.

`off` works by specificity.  You can unbind all events on an object or be specific with the set of events you want to remove.  Cleans up the bookkeeping after things are removed.

`trigger` can be similiarly recursive.  It looks up th eevent and calls triggerEvents and also triggers the `all` event (if something is listening to it).

`triggerEvents` is an optimization that matters.  In most JS engines `call` is faster than `apply`.  This is optimizing for most callbacks to events that take less than three arguments. 

`listenTo` is the IoC version of `on`.  What is the object we're listening to and let's keep a reference to that object.  listenToOnce is the same pattern as once.

`stopListening` is useful if you have a view listening to several models and the view is destroyed.  In order to stop memory leaks you'd have to go to each model and unbind the listeners.  With `stopListening` you can just call `view.stopListening`.

**Models**

Models are the core of the data layer in Backbone.  The important thing to note about the constructor is the order of things that are done.

Attributes are parsed out.  Then attributes are set.

There are a bunch of functions you can override.

toJSON is a poor name because it doesn't actually give you JSON.  You return what you'd like to be serialized to JSON. 

sync can override Backbone.sync

Get provides indirection for symmetry between get/set.

Model has a value if the value of that attribute is neither null or undefined.  .has() checks != null to check for null + undefined.

Model.set A little more complicated then you might think.  The idea of Backbone events is that they are synchronous.  On the next line after .set() you can depend on all your events having fired.  So there's some tricky bookkeeping.  There's a changing flag to check if you're more than one level deep because if you are you don't want to start from scratch with your previous attributes.  The more nested change events you have keep building a diff between values.  If values aren't the same we say they've changed and push them into that list.

Once we've built up the changed list we trigger change events.  If we're still in a nested change (you can have many nested changes together but we only want one event) so we avoid firing the event.  Then at the top level we fire the change.

Backbone implements its primitives in terms of other things internally.  Unset is just a set where you pass in undefined values.

Clear unsets all the values that we've set.

changedAttributes will do a diff to tell you what the previous attribute is if its changed.

previous always tells you the previous attribute.

With models you have your clientside state, but you also want to persist that to the server.  When you save a model you can save it with a fresh set of attributes, and that gets set on the client side, otherwise we validate them.

Save is normally optimistic.  WHen you save it will set the attrs on the client and hopefully it works.  But if you sait wait to be true it will be a pessimistic update.

One tricky point is that if there are attributes and youre waiting we changing them temporarily and then we set them back to the other one.  FOr toJSON to work we need to temporarily patch in the optimistic attributes so you can send your JSON representation to the server.

Model destroying is not super fancy, except that we call stopListening by default.  So if you've been using listenTo then these will be cleaned up for free.

The way the URL is determined for the model is that if you've defined a URL root on your model or if you've set url on your collection.  If its never been saved before it will use a rail style conventiont to post to the base but if it isn't new then it will post to root/id

isNew checks if there is an ID.  Clients can't generate IDs because it needs to be unique.  If there is an ID we assume the server has knowledge of it.

There is no validate function in Backbone, but if you desire to you can create a validate() function that returns false or an error if it fails.

We proxy a bunch of underscore goodies into Model.  We take all the methods in the list and stick them on the model protype.

**Collections**

They're where your data manipulation happens.  Working with a discrete unit is fine, but you get more value working with a set of data.  Same set up for overriding default methods as Model.

Collection.set is the biggest function in Backbone.  When you set new models in a collection you add/remove/merge models.  You can do this at a particular point, or you can do it at the end.  You can have the collection sort itself or not.  And you have to trigger all the events.  First we parse the models and figure out if there is a position for them to be placed.  We check for duplicates with a modelMap.

We go through all  the models in the list and see if it is already prestent.  If it is, and we're merging, we merge in the attribute.  Here we can check if we're sorting and hasChanged and then we can set sort=true flag so we know we need to sort the entire list.  A  neat way of using bookkeeping so we don't need to resort every time.

If we're adding we push it into a list to add (or ordered list if we're ordering).

After we've merged in the new attributes we go through the list to remove.  Then we add stuff after that (if it has to go into a particular position we splice them in, otherwise we push them on the end, also one at a time so we get the sequence of add events in order).  If we need to sort, then we do a silent sort and then trigger events.

Mutators: we need to implement array mutators for collections.  Pushing is just an addition at the end of the collection.  Pop is a removal at the end.  Shift and unshift do the same at the beginning.

Collections are doubly indexed (id and ordering).  Get gives it to you by id, at gives it to you by index.  Where and findWhere will return models that match the set of attributes.

Sorting uses the comparator property/function on your collection.  Used to only support `_.sortBy` style comparators but in complicated places you can't just return a sort value so we need to support a full style sort.  Uses function.length to figure out which type.

The heart of the collection reference tracking is paying attention to all the events going on in the collection.  By default we listen to all the events on hte models.  On model event callback it will remove if a destroy event occurs.  Change will update id mapping if its changed.  Then we proxy the same event on this.

Then we implement a huge list of underscore functions by proxying.  There are two methods for proxying, though.  The default applies them on the models, but there are a few that act on the attributes of the models, not the models themselves.  A good demo of how functional and OO methods don't have to be at odds with one another and that you can find a harmony between the two.

**Views**

Views are very simple in Backbone; they don't do much.  Render function by default just returns this.  You can use whatever you'd like to optimize your views.  A view is a top level DOM object that always exists (this important because it avoids the jQuery nightmare where you're binding the click handler on an object that doesn't exist at the time).  If the element always exists then your events are always going to work.  And they are implemented efficiently because they're delegated.

**Sync**

Uses a methodMap to map between CRUD and REST.  If you're server doesn't support fancy methods like PUT/DELETE we just use POST.

**Router**

A list of regexes that is applied against the URL and picks out which ones match and sends that to you.  routeToRegExp takes your URL string and turns it into a regex.  extractParameters is the inverse.

**History**

Most of history is not super interesting anymore because it has to do with support for old browsers.  But the one cool hack is that to use a secrete iFrame for oldIE that writes new fragment URLs that get stuck in your browser history.

**Upcoming**

More hooks for overriding jQuery so you can replace it with closer to native DOM calls.