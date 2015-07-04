For all the frontend developers who just got comfortable with last week's tools, Google's Polymer version 1.0 announced at Google I/O is about to modernize your image of the "widget." Known as Web Components, Polymer streamlines the process of implementing the Web Component standard and adds cross-browser support that makes Polymer just about production ready.

To give a better example to why Polymer and Web Components are so great, I'm going to build a photostream component using the API from our gif photo booth app called Gifn. I'll explain each part as we go along.

To install Polymer we are going to use Bower, which is a package management tool for javascript. If you do not have Bower installed you can do so in the terminal with

	npm install -g bower

Initialize bower

	bower init

Follow the prompts from bower. If you are in doubt on what to answer just push enter and use the default option. After bower intializes it should place a bower.json file in your directory. To add Polymer as a dependency run this command.

	bower install --save Polymer/polymer#^1.0.0

You should now have a bower_components/ folder in your directory that contains Polymer and its dependencies.

