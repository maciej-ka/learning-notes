React.gg, master modern React
=============================
https://react.gg/

### Why React
React was not well received when it was created.

"It would be nice to see a list of reasons to chose this over Angular"  
"I think Web Components is landing soon"  
"Every example is 100 times the code I would write in straight html"  
"Do templates have to live inside the application logic? It's poor decision"  
"Tis is terrible, so did we really not lear naything from PHP days?"  
"I really hate the embedded XML in javascript code"  
"Seems a child of PHP and Angular"

At that time jQuery, Backbone and Angular dominated.

#### jQuery
With jQuery the state of your application lived inside DOM.  
However, relying on shared mutable state was also its biggest problem.  
What started as a simple way to update DOM tree,  
typically devolved into spaghetti like mutations  
that were both hard to predict and keep track of

#### Backbone
Backbone had just 2000 lines of code.  
unopinionated and minimal  
It allowed to decouple application state from the DOM.  
Instead of living in the DOM, backbone state lived in its models.  
All the views that cared about that state, rerendered on change.  
its render function was something for users to override

#### Angular
opinionated and not minimal  
Misko presentation barefoot  
idea was: what if html would be more powerful  
it was a true framework  
and it fitted well backend developers  
who were given task to work on frontend

Problem was: Angular loved two way data binding  
view was updated when the model changed  
but also model changed when the view changed  
this was hard to follow, but also expensive  
as Angular had scan you app looking for state changes.


