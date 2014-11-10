![ArangoDB-Logo](https://www.arangodb.com/logo/Arangodb_Logo_400.png)

<div class="fa fa-exclamation-triangle" style="color:rgba(240,210,0,1)"> The cookbook is still in progress. <div class="fa fa-exclamation-triangle"></div></div> 

In the ArangoDB cookbook you will find some recipes for specific problems regarding ArangoDB. 

##How to contribute

Do you have recipes for the ArangoDB Cookbook?  <br>
Write them down and we will add them to the Cookbook.

It's pretty easy. You write a markdown file in the recipes folder. At the end of your recipe you can add your name and some tags so other users can find your recipe faster.

An Example:
```
Author: [yourName](https://github.com/yourName)
Tags: #arangodb #cookbook #aql
```

After that you edit `SUMMARY.md` and simply add a title and the file name to the end of it. 

Now make a pull request and we will add your recipe as soon as possible.

##How to use a local Cookbook

If you want to create a local Cookbook perform the following steps:

* Install the [Gitbook][1]:

```
$ npm install gitbook -g
```

* Go into the recipes folder
* Use `gitbook build`
* open 'recipes/_book/index.html

[1]: https://github.com/GitbookIO/gitbook