For all the frontend developers who just started to feel comfortable with last week's tools, Google's Polymer version 1.0 announced at Google I/O is about to modernize your image of the "widget." Known as Web Components, Polymer streamlines the process of implementing the Web Component standard and adds cross-browser support that makes Polymer just about production ready.

To give a better example to why Polymer and Web Components are so great, I'm going to build a photo stream component using the API from our gif photo booth app called Gifn. I'll explain each part as we go along.

First create a new folder in the command line using
	
	mkdir polymer_tutorial

And then enter the folder using
	
	cd polymer_tutorial

To install Polymer we are going to use Bower, which is a package management tool for javascript. If you do not have Bower installed you can do so with

	npm install -g bower

and then initialize bower.

	bower init

Follow the prompts from bower. If you are not sure how to answer a question, just push enter and use the default option. After bower intializes it should place a bower.json file in your directory. To add Polymer as a dependency run this command

	bower install --save Polymer/polymer#^1.0.0

You should now have a bower_components/ folder in your directory that contains Polymer and its dependencies.

From here, add two files to your root directory. Add an index.html file and a gifn-feed.html file that will contain your Polymer component. Your directory should now look something like this

	polymer_tutorial/
		bower_components/
		bower.json
		gifn-feed.html
		index.html

First let's open up the index.html file and setup our html to display the polymer component. Here is all of the html needed to use a simple component.

	<!doctype html>
	<html>
	    <head>
	        <title>Gifn Polymer</title>
	        <script src="bower_components/webcomponentsjs/webcomponents.min.js"></script>
	        <link rel="import" href="gifn-feed.html">
	    </head>
	    <body>
	        <gifn-feed event="prplhq"></gifn-feed>
	    </body>
	</html>

There are three main parts to initializing the component in your index.html file. First you need to add the webcomponents library

	<script src="bower_components/webcomponentsjs/webcomponents.min.js"></script>

This library is required for all Polymer components and any custom components you create yourself. There is also a smaller webcomponents-lite.min.js library that can be used in replacement if you are only building elements for browsers that currently support the Web Components API (Chrome 36+ for example).

Next is the link tag importing our gifn-feed web component to be used.

	<link rel="import" href="gifn-feed.html">

And then our actual web component in use

	<gifn-feed event="prplhq"></gifn-feed>

Everything looks pretty normal except for the actual web component. The gifn-feed element is obviously not your normal html and event="prplhq" is not an attribute you would expect either. This will all make sense once we start fleshing out the component. Let's get started.

First open up the gifn-feed.html file and add the required base Polymer element at the top of your file

	<link rel="import" href="bower_components/polymer/polymer.html">

One of the cool aspects about Polymer is that it allows you to import and extend elements in a very modular way. Outside of the required base polymer.html you could also import iron-ajax.html for easy ajax calls or even a material design button with paper-button.html. You can check out some ready to use components Google has created at https://elements.polymer-project.org/.

Since gifn-feed is going to be used in our html, we are going to initialize it as a DOM module like so

	<dom-module id="gifn-feed">
	</dom-module>

Now for the fun part. Let's initialize the polymer component with the base setup inside a script tag.

	<dom-module id="gifn-feed">

		<script>
	        Polymer({
	            is: "gifn-feed",

	            properties: {
	              event: String
	            }
	        });
    	</script>

	</dom-module>

The "is:" attribute tells Polymer to look for an element with the tag gifn-feed and the "properties:" attribute tells Polymer to look for an "event" attribute that will have a string value. 

Our web component still does nothing, it's simply initialized. To add an action to the component, you can use one of the lifecycle callbacks, or in our case, use the "ready" callback for when our component's DOM has been initialized. Here is the gifn-feed component with the "ready" callback.

	<dom-module id="gifn-feed">
		<script>
	        Polymer({
	            is: "gifn-feed",

	            properties: {
	              event: String
	            },

	            ready: function() {
	                var self = this;
	                var xhr = new XMLHttpRequest();
	                var baseUrl = 'https://rest.gifn.it';
	                self.gifns = [];
	                
	                // Send an ajax request to get the gif data
	                xhr.open('GET', baseUrl + '/gif/' + self.event);
	                xhr.send();
	                xhr.onreadystatechange = function() {

	                    if(xhr.readyState == 4 && xhr.status == 200){
	                        var gifns = [];
	                        var response = JSON.parse(xhr.responseText);

	                        //Build out the urls for the 5 latest gifs
	                        for(var i = 0; i < 5; i++){
	                            var gifUrl = baseUrl + '/asset/' + self.event  + '/' + response.gifs[i].slug + '.thumb.gif';
	                            gifns.push({ url: gifUrl });
	                        }

	                        self.gifns = gifns;
	                    }
	                }
	            }
	        });
	    </script>

	</dom-module>

What we are doing in this ready function is making an ajax request to the Gifn API to retrieve a list of gif images. We then build out the urls and place them in an array that is set to our polymer object. Let me explain some of the more important aspects.

	var self = this;

Although this is a common practice for dealing with internal scope in javascript functions, it's especially important to have access to the parent polymer object so you can bind data to the view. Here's an example of it being used.

	xhr.open('GET', baseUrl + '/gif/' + self.event);

This line is responsible for initializing the ajax request to our API, but self.event is what makes it truly special. Remember the event="prplhq" attribute on our gifn-feed tag? It is now directly bound to our polymer object for use in the javascript, html template and even in our component's css. Here's another example.

	self.gifns = gifns;

This time instead of taking a value from our component's element attribute, we are creating an array of urls inside the component that can then be used throughout. Polymer is flexible in that it allows for both one-way and two-way data binding.

Now that we have our data set, let's create the view for our component. Above our script tag add

	<template>
    </template>

This is the base template tag required for all views. Inside of the template tag you can add html, other imported polymer web components, and even some special helper elements used for if-statements and looping through data. Since we have an array of urls, let's add a dom-repeat helper tag.

	<template>
        <template is="dom-repeat" items="{{gifns}}">
            <img src="{{item.url}}" />
        </template>
    </template>

Notice the {{gifns}} value in the items tag? This is how you directly access attributes bound to your polymer object. So if you wanted, you could also add a title for the event like so
	
	<h1>{{event}}</h1>

In the case of the special dom-repeat element, it requres an "items" attribute that references an array value. The dom-repeat element will then loop through the array with each value being accessed using the "item" variable.

After seeing the completed view, some of you may be wondering why I chose to make this string concatenation mess in the ready function.

    for(var i = 0; i < 5; i++){
        var gifUrl = baseUrl + '/asset/' + self.event  + '/' + response.gifs[i].slug + '.thumb.gif';
        gifns.push({ url: gifUrl });
    }

You all would probably agree this is the better and cleaner option.

	<template is="dom-repeat" items="{{gifns}}">
        <img src="https://rest.gifn.it/asset/{{event}}/{{item.slug}}.thumb.gif" />
    </template>

However, this is currently not possible and is probably one of the biggest downfalls of the template builder in Polymer 1.0. String concatenation and whitespace with view variables is currently not supported in Polymer. Each element must take up the entire space of a view's element or the entire space of a string in an attribute. 

For example, this will work.

	<h1>Welcome <span>{{name}}</span></h1>

but this will not
	
	<h1>Welcome {{name}}</h1>
	
To finalize the view, let's add a touch of css to our component to make everything a little more fancy. Let's add this style block above our template tags.

	<style>
        img {
            display:inline-block;
        }
    </style>

Alright, maybe not that fancy, but I really hate floats. Another great feature of Polymer is that it keeps your component's css local so you do not need to worry about conflicting css rules.

And that's it! Here is the polymer component in it's entirety.

	<link rel="import" href="bower_components/polymer/polymer.html">

	<dom-module id="gifn-feed">
	    <style>
	        img {
	            display:inline-block;
	        }
	    </style>

	    <template>
	        <template is="dom-repeat" items="{{gifns}}">
	            <img src="{{item.url}}" />
	        </template>
	    </template>

	    <script>
	        Polymer({
	            is: "gifn-feed",
	            properties: {
	              event: String
	            },
	            ready: function() {

	                var self = this;
	                var xhr = new XMLHttpRequest();
	                var baseUrl = 'https://rest.gifn.it';
	                self.gifns = [];
	                
	                // Send an ajax request to get the gif data
	                xhr.open('GET', baseUrl + '/gif/' + self.event);
	                xhr.send();
	                xhr.onreadystatechange = function() {
	                    if(xhr.readyState == 4 && xhr.status == 200){

	                        var gifns = [];
	                        var response = JSON.parse(xhr.responseText);

	                        //Build out the urls for the 5 latest gifs
	                        for(var i = 0; i < 5; i++){
	                            var gifUrl = baseUrl + '/asset/' + self.event  + '/' + response.gifs[i].slug + '.thumb.gif';
	                            gifns.push({ url: gifUrl });
	                        }

	                        // Bind the response back to the {{gifns}} repeater
	                        self.gifns = gifns;`
	                    }
	                }
	            }
	        });
	    </script>
	</dom-module>

To view your polymer component you will need to host it through some type of server. This could be a local server, vagrant box, MAMP, or one of my quick command line favorites

	python -m SimpleHTTPServer

Enjoy!

A working demo can be seen at http://polymer.gifn.it and all of the source code can be viewed on my github at https://github.com/adamboerema/polymertutorial

For further documentation check out https://www.polymer-project.org/1.0/ and http://webcomponents.org/
