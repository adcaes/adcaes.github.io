---
layout: post
title: "Getting started with Twisted"
date:   2015-09-14
tags: [Programming, Python]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
  show_in_list: false
---

## Introduction

Twisted is an asynchronous programming framework for Python. Twisted is huge and can be a bit confusing at first, but I will try to explain the basics to be able to get started using it.

First of all though, it is important to be familiar with the computing model behind Twisted: Asynchronous programming. If you are already familiar with this topic feel free to skip the next section.

## Asynchronous Programming

Let's start by reviewing the other two traditional computing models to be able to compare with them: single thread model and multithread model.

The traditional single thread (synchronous) model is very simple: only one task is performed at a time, and a new task can't start until the previous one has finished.

The multithread model is a bit more complex. In it, each task is executed by a separate operating system thread, and the operating system can replace the running task by another one at any time. On a system with multiple cores, different threads can run truly concurrently, or may be interleaved on a single core. It is important to notice though, that in Python, because of the [Global Interpreter Lock (GIL)](https://wiki.python.org/moin/GlobalInterpreterLock), multithread applications never run truly concurrently.

In the asynchronous model, tasks are interleaved with one another in a single thread. The main characteristic of an application following this model is that when the task being run is going to block, the application will continue executing another task, minimizing the time that the whole application is blocked. In this model, the running task is replaced by another when either it finishes or it is going to block.

An application benefits from using this model if it consists of a set of independent tasks with a fair amount of blocking. The most common cause for tasks to block is waiting for I/O to complete, like reading or writing from the network or the file system. That's why the canonical applications for this model are a web server or a web client.

## The Reactor and Deferreds

Twisted has two main components: the Reactor and Deferreds

The Reactor reacts to events and schedules tasks, that’s why it is known as Reactor or Event Loop. The job of the Reactor is to manage the pool of tasks that are waiting to be executed. When the running task gives control to the Reactor by starting an asynchronous operation, the Reactor takes care of putting the task in a waiting state, setting a mechanism to call the result handler associated with the task once the result is available, and continue executing another task.

The other important component of Twisted are Deferreds. Deferreds encapsulate asynchronous tasks, similar to the idea of Promise or Future in other frameworks. Asynchronous functions in Twisted return a Deferred, and the Deferred is used to control the execution of the task and access the result once it is available.

To interact with a Deferred we can attach a series of functions to be called (fired in Twisted terminology) when the result of the asynchronous task is available. These series of functions are known as callbacks or callback chain. We can also attach a list of functions to be called if there is an error in the asynchronous task, known as errbacks or errback chain. The first callback is called when the result is available, or the first errback if an error occurs.

Enough with the theory, lets get started with a few simple examples.

## Example: Getting a URL

Let’s start with a simple example, getting the contents at a URL and printing the returned HTTP code. For this we will use Treq, a library inspired by Python Requests written on top of Twisted.

```python
from twisted.internet.task import react
import treq

def get_url(url):

    def handle_response(resp):
        print "Got response code %d from %s" % (resp.code, url)

    def handle_failure(failure):
        print "Something failed: %s" % failure.getErrorMessage()

    print "Getting URL %s" % url
    d = treq.get(url)
    d.addCallback(handle_response)
    d.addErrback(handle_failure)
    return d

def main(reactor, *args):
    url = 'http://google.com'
    d = get_url(url)
    return d

react(main)
```

Let’s analyze this code by parts:

* ```react``` is a utility function provided by Twisted that starts the reactor, executes the provided function, in this case main, and stops the reactor once all tasks have finished.
* The ```main``` function takes care of calling ```get_url``` with a specific url.
* ```get_url``` calls ```treq.get```, that gets the contents at the requested URL. As it is an asynchronous function, ```treq.get``` returns a Deferred.
* We add a callback to the returned Deferred that prints the response code, and an errback that prints the error if something goes wrong.

## Example: Getting multiple URLs

The first example was quite similar to a normal single thread program, let’s see a bit more of Twisted by slightly modifying the first example in order to get multiple URLs concurrently. We only need to modify the main function for this:

```python
from twisted.internet.task import defer

def main(reactor, *args):
    urls = ['http://github.com', 'http://www.google.com', 'http://www.facebook.com']
    ds = map(get_url, urls)
    d = defer.gatherResults(ds)
    return d
```

Here we call the ```get_url``` function for multiple URLs, getting a Deferred for each call. Then we use the Twisted function gatherResults to create a new Deferred that gets fired when all the provided Deferreds have fired.

## Inline Callbacks

Finally I would like to tell you about inline callbacks, a Twisted utility that allows writing Deferred-using functions that look like regular sequential functions.

```python
from twisted.internet.defer import inlineCallbacks

@inlineCallbacks
def get_url(url):
    print "Getting URL %s" % url

    try:
        resp = yield treq.get(url)
        print "Got response code %d from %s" % (resp.code, url)    
    except Exception as e:
        print "Something failed: %s" % str(e)

```

This version of ```get_url``` has the same behavior as the previous one, but with the ```inlineCallbacks``` decorator we can use yield to wait for the result of a Deferred instead of using callbacks and errbacks, making the code look similar to a sequential program.

If everything goes well, the result of the Deferred will be the value returned with the yield, if there is an error, an exception will be raised.

## Conclusion

I hope this is enough to get you started with Twisted. If you want to know more I suggest looking here:

* Twisted documentation, I specially recommend the part about [Deferreds](https://twistedmatrix.com/documents/current/core/howto/defer.html): https://twistedmatrix.com/documents/current/
* A more in depth explanation of Asynchronous Programming and Twisted: http://krondo.com/wp-content/uploads/2009/08/twisted-intro.html
