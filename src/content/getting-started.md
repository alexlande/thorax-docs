### 1
- `npm install yo`
- `npm install thorax`
- `yo thorax`

### 2

Open the directory in your favorite editor - these examples will all use sublime text. Things you should know up front: 

- you must use the seed project to build with thorax
- most of this directory structure, and the dependencies, are dev happiness and build related. this is a mobile framework - as little as possible goes over the wire. only the public folder.

Here's what you should know about:

- components, here live bower files?
- js, most of your work done here, notice init.js in particular besides the folders, note the assignment of things to hte application object though that's probably going away... 
- node modules should be self explanatory
- public is what's going over the wire, notice in particular that the index file is super lightweight, here's what should be in it... 
- stylesheets is pretty simple
- tasks is grunt
- templates is wehre all your goods go, not in index
- application.handlebars is a special beast with headers and footers and layout objects
- no lumbar.json anymore but the other json should be self explanatory
- AN IMAGE OR SERIES OF SNIPPETS OF IMAGES WILL GO HERE TO SHOW THE DIRECTORY STRUCTURE. THIS SECTION IS AN ANNOTATED IMAGE OF A DIRECTORY STRUCTURE. we will have to make it pretty and navigable though it will be a shitload of information, while still having the call to actions clear at the bottom.

### 3

Here are three tutorials of increasing complexity. 

- the first one is a very small contact list tutorial that shows the interaction of the major concepts in building a thorax app: models, collections, routers, views and templates, and shows the interaction between all of them. CRUD. Localstorage...
- the second one shows more complex views, ui patterns, all the bootstrap api implemented with thorax.. ie., pagination with collection rendering, showing child views inside of tabs, etc., also done with local storage
- the third one is networked with the rails backend and is the 10-4 app! yay! so much to talk about here. this will be less step by step, at this point, since they should be able to read a directory structure now, and more about application architecture. 


### [go to api call to action](api.html)
### [start first tutorial psuedo call to action](first_tutorial.htlm)
