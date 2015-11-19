---
layout: post
title: "Switching to Atom from Sublime Text"
date:   2015-04-18
tags: [Atom]
---
## The Switch

I have used [Sublime Text 2](http://www.sublimetext.com/2) for all my local coding during more than three years. I started using Sublime because I wanted a powerful, fast, multi platform code editor.

Around 6 months ago I read about [Atom](https://atom.io/), an open source code editor developed by Github, and I decided to give it a try, but simply scrolling through a file had a noticeable lag, so I continued with Sublime.

A couple of months ago Sublime became slow in my computer due to a problem in the new version of a package, and I decided to give Atom a new opportunity; this time I stayed with it.

## Why Atom?

Atom feels very familiar for a Sublime user, it has the same layout, and almost the same keyboard shortcuts, so switching to it is not extremely traumatic.

The main difference I found at first was the package manager. Unlike Sublime, Atom has an integrated and very functional package manager ([apm](https://github.com/atom/apm)) that can be used from the terminal or from Atom.

All the features that make Sublime great are also available in Atom, and some extra ones:

* Fuzzy file finder ```cmd+P``` or ```cmd+T```.
* Command palette ```cmd+shift+P```: Execute any command offered by Atom by typing its name.
* Split editing: Edit multiple files side by side.
* File (cmd+F) and project search ```cmd+shift+F```.
* Git integration.
* Key binding resolver ```cmd+.```: Best way to learn keyboard shortcuts.
* And lots more: Project tree view, go to line/symbol, multiple cursors, markdown preview...

## Setting up Atom

I use the theme One Dark, both for UI and syntax highlighting. Atom already comes with a bunch of pre installed packages, and I add the following ones:

* [Auto detect indentation](https://atom.io/packages/auto-detect-indentation): Automatically sets the indentation configuration to match that of the file being edited.
* [Autocomplete-plus](https://atom.io/packages/autocomplete-plus): Provides autocompletion while typing.
* [Highlight selected](https://atom.io/packages/highlight-selected): Highlights all the instances of the selected word.
* [Merge conflicts](https://atom.io/packages/merge-conflicts): Graphically resolve merge conflicts.

For Python development:

* [Linter pylint](https://atom.io/packages/linter-pylint): Python linter.
* [Autocomplete plus Python Jedi](https://atom.io/packages/autocomplete-plus-python-jedi): Python autocomplete based on Jedi.

For Go development:

* [Go plus](https://atom.io/packages/go-plus): A real must, can be configured to run go format on save, automatically add imports with goimports, and presents the output of go vet and golint in the Atom console.

For React.js development:

* [React](https://atom.io/packages/react): React.js (JSX) language support, indentation, snippets, auto completion and reformatting.

## The Good and the Bad

I have been using Atom as my main editor for a couple of months and I'm satisfied with it. It has almost all the good things of Sublime and a very active community with constant updates and improvements to both the editor and the packages.

But Atom is much less mature than Sublime and it has some rather basic issues, like Git integration not working properly when opening multiple Git repos in the same window, long start up time or not opening files bigger than 2MB; hopefully all this will be soon solved.
