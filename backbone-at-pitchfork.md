# Backbone at Pitchfork
## Matt Dennewitz, [@mattdennewitz](http://www.twitter.com/mattdennewitz)

Stack:

* jQuery + Backbone in the front
* Django in the back
* Varnish between the two

Backbone gave us the structure and support for building interaction.

**Audio player**

Had to support several different services.  Some have to stream over RTMP.

Translates really easily into Backbone.  Tracks = Backbone.Model.  Playlists = Backbone.Collection.  Play control = Backbone.View.  UI = (several) Backbone.View.

Used a View for a Controller because it's class-like.

Onboarding is pretty simple because the code isn't that complex.  Low complexity --> high productivity.

Backbone has spreaded to cover most clickable modules on the site.

Moving away from a traditional CMS and into something with more atomic blocks of content.

Cover stories call out valuable stories.  Each cover story has sequenced panels.  The attraction of using Backbone was that it was very light and simple.  Every panel is a View. Views only care about what they must.

Backbone was love at first line of code.