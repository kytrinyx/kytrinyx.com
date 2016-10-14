+++
title = "The Scandalous Story of the Dreadful Code Written by the Best of Us"
slug = "scandalous-story"
date = "2016-10-10"
description = "There are overlooked corners of our codebases. Ignored, unloved. Unimportant. Or so we thought. What was once inconsequential has—somehow—grown into ghastly mess. This talk tells the story of one such mess, and the taming of it."
video = ""
+++

**I will post the video when it becomes available**

## Transcript

We got this bug report. It said: I never get the notifications when we run out of stuff.

It went on to explain that they were warned when they were running low on something, so they knew to make plans to replenish, but then they'd try to use the resource, and they'd already be out.

The context here is a game. It's an online multi-player zombie apocalypse role playing game, and running out of stuff is problematic. Almost every decision you make in this game depends on what you have and what you need.

The notifications get triggered on various events, for example when zombies are spotted, or people join or leave tribes, and of course anything to do with gaining or losing resources. The notification in question is resource depleted.

The cause of the bug was that for all of our notifications we had a text version of the notification that goes to some notification channels and an html version of the notification that goes to other notification channels. They were defined separately. As you might have guessed, this is a copy/paste error. We copied resource scarce to create resource depleted and forgot to edit the HTML.

You might be thinking shouldn't the tests have caught this? And the truth is, yes, they should have. And they probably would have, too, if we had written any. So yeah. The notify package is kind of awful and there are no tests.

We were actually OK with this. When we first implemented the notifications, they weren't really all that important. We didn't have very many of them, and they were really straight forward. Over time they started becoming more and more central to the mechanics of the game. We implemented more notifications, and they got more a lot more complex.

This bug report served as a signal that it was time to get this part of the codebase under control, and the first step was to add tests.

Here's what makes it tricky to test. The functions in the notify package call functions on our comms package which in turn talk to a 3rd party pubsub service, and that service is devilishly difficult to test. There might be a way to configure the comms package to let us test it, but we're not trying to test the comms package, we're trying to test the notify package. We don't care about all of this complexty. These functions are completely incidental to our purpose.

What we want to know is _did the message say the right thing_ and _did the right people get the message_?

It turns out there's a very careful dance you can do to make it utterly trivial to test this feature. With this maneuver you'll never be more than 30 seconds away from a potential production deploy at all times, throughout the entire process. It's called the **no-op no-op no-op interface maneuver**.

## The No-op No-op Interface Maneuver

What the no-op no-op no-op interface maneuver lets us do, is move things around in tiny increments until we can tell the notification functions who to talk to for their communication needs. Then in production we'll use a real transmitter, and in test we can pass in fake transmitters that just keep track of what happens.

Almost every single step in this maneuver is safe, and "safe" in this context has a precise, technical definition. Safe means that you can reverse the step with a single undo. So a safe step is one edit, and the code will work both before and after that edit. And if it doesn't, you can back it out with one undo.

There's one moment where the code won't work, but it's very brief, and you can lean on the compiler to fix it.

Here we go. Three no-ops and an interface.

### No-op #1

The very first thing you're going to do is not change any existing code. This is very important. All the existing code works and working code pays the bills. So here are the existing comms functions which you're not going to change... yet.

In this package, define a new type. It kind of doesn't matter what this type is, because it's not going to have any fields or data or values. It's just a placeholder. It's a hook where you can attach some methods.

Since our notification code calls three functions, we're going to define three methods on this type. One for each function. Each method needs to have the same signature as the original function. And each method should be empty.

Back in each of the old functions where you already have mountains of code, you are going to add a single line to each function that calls the equivalent method on a value of type Signal. That value is a package variable that's just a nil pointer. Since the methods are defined on that pointer, this works without ever assigning anything to it.

So that's no-op number one. We created a new type with empty methods, and updated the code to call those methods.

### No-op #2

The original comms functions are doing two things. First they're doing all their actual work. Then they're calling a method which does nothing.

Move all of the work from the old function to the new method.

And that's no-op number two. The notification is still calling the old function, the old function is calling the new method, and the new method is now doing the heavy lifting.

### No-op #3

Now we want to cut the middleman out of this transaction. We have this new type, and we want our notify package to talk to it directly. To do this we need to change the notify function signatures to accept a new argument which is a value of this new type. We do this in three small steps.

First add the parameter to the function signature. This is where the code won't compile.

Then update all the callers to pass the package variable that we defined. Now the code compiles again and everything works.

The body of the function is still talking directly to the comms package. Update this to use the com argument instead.

And now the original comms functions are no longer being called, so we can delete them.

And that's no-op number three. We injected the dependency into the notify function and got the function to talk to the injected dependency.

### Interface

We added all this code in order to keep doing exactly what we were already doing. We still can't add tests. This is where the magic happens. In the comms package, define an interface that corresponds to the three methods on the Signal type. Then in the notify package change the type of the parameter to be the interface type instead of the concrete type. Everything still compiles, and everything still works.

And now, finally we can implement a fake transmitter type in the notify package, defining the methods that it needs in order to satisfy the transmitter interface. These methods should be trivial. Capture whatever information you need, and then in your tests, verify it however you want.

So. Three no-ops. One interface.

The first no-op defined a new type, and hooked it into the code. The second no-op moved the work into the methods on this type. The third no-op eliminated the need for the original functions by getting the code to call these methods directly. And finally, defining an interface made it possible to inject fake transmitters, which made it trivially easy to implement tests for the feature.

## The Flocking Rules

The point of all of this is to make it safe to clean up and simplify the notify package. To clean this up I'm going to ignore everything that is known about good design, design patterns and design principles, because most of the time, when I'm looking at legacy code, I have absolutely no idea what it should end up looking like. I don't know where I'm going. It's just this giant, overwhelming mess.

But even though I don't know where I'm going, I do know how to get there. It's a process that Sandi Metz calls the Flocking Rules, named after the incredibly complex behavior exhibited in nature by flocks of birds or schools of fish where each individual is following a few simple rules.

The flocking rules for eliminating duplication go like this:

- find the things that are most alike
- select the smallest difference between them
- make the smallest change that will remove that difference

The beauty of these rules is found in cycling through them repeatedly and discovering what emerges.

Lets start with a notification that exhibits a lot of problematic duplication, the resource found notification. This has about a hundred lines of code, but we're only going to look at the bit that caused the bug we saw at the very start--the part where we build two versions of the same notification independently of one another.

Here's the text version. It's got a simple format string, and it's passing in some game values.

The HTML is doing basically the same thing, except there's more of everything. The format string hand-crafts HTML, and it passes in heaps of data.

Alright, so... the flocking rules.

_Find the things that are most alike._

It's not always obvious where to start. If you have a number of choices that all seem reasonable, just pick one. If you have no idea where to begin, just pick something and try it. You'll quickly figure out whether or not it was a useful place to begin. Here it seems like the two format strings are a plausible choice.

I'm going to collapse the data arguments for a bit to make it easier to focus on the format strings.

_Select the smallest difference between them._

Where the plain text version has simple formatting verbs, the HTML version has hard-coded HTML anchor tags with multiple format verbs for each.

_Make the smallest change that will remove that difference._

If we pass the entire anchor tags as arguments to Sprintf, then the format string becomes identical in both cases with the messy bits of the HTML extracted into new and smaller format strings. We can collapse the identical bit into a temporary variable.

Then repeat the flocking rules.

What's most alike? The anchor tag creation code that we just extracted suddenly seems much more repetetive than it did when it was embedded in the rest of the HTML notification. What's the smallest difference? Again, there are a few options here. Let's start with the href. How can we make all the hrefs identical? The cleanest thing would probably be to define a URL() method on the individual game types. And to be fair, we really shouldn't be hand-coding these all over the place anyway.

Once we've done that, and we call those URL methods, things are looking considerably less noisy.

Repeat the flocking rules.

That template escaping thing is really similar, the only difference between them is the argument we're passing. In the first two cases we're referencing a struct field, and in the third we're calling a method. There's no reason why we can't call methods everywhere. Define a String() method on the other two game types.

And then repeat the flocking rules.

The anchor tags are now almost identical. The only difference between them is the game value that we're calling the methods on. Notice that we're calling the same two methods on all of the game values: String() and URL(). These make up a tiny interface, it just hasn't been defined yet. We need a name for the interface, and because these are used to define links, I'm thinking linker.

Now we can extract the creation of the anchor tag to a function that takes a linker, and this completely removes the repetetive bits for making the HTML links, leaving us with just the repetition in creating the notifications themselves.

The duplication is more obvious, and the differences more subtle. Take a look at the way that we're processing the data arguments in the HTML notification. We're handling them all consistently. In the text version of the notification, we're not. We're referencing struct fields on a couple of arguments, and the String() method is being implicitly called on the last one. We already dealt with this for the linker. All the game types have a String() method, and that method would do the right thing here. We can stop referencing the struct fields altogether, and let Sprintf take care of the details.

We're very close to identical now. The difference is that the text version is using the game values straight up, whereas the HTML version is passing them to the link function first. We can make them even more similar by defining a text function that takes a linker. This seems a bit pointless, but it's worth being just a touch pedantic in following the flocking rules, because it can push you to do do things which are unintuitive, and which can lead to interesting outcomes.

Now the shape is the same everywhere. We're still working on this bit of duplication here. The difference is the name of the function that the arguments get passed to. The two functions have the same signature, so we can create a formatting function that takes a function with that signature as an argument. It would also need to take the format string and the game values that are being used as data.

In this function loop through the game values, and pass them to the function that we passed in before running it all through Sprintf. And finally, those two lines of code are identical.

This smells like a type. All of these things belong together. They deserve to be an actual thing, not a collection of seemingly unrelated things that happen to line up.

If we create a type that stores a template and some game values, along with a function that will make new messages, we can move the formatting function so that it is method on this type, and then with a couple of wrapper methods, the body of the notification function ends up looking like this.

We started out with an undifferentiated mass of stringy stuff, and ended up with two crisp abstractions: a message type, which knows how to render itself as plain text and HTML, and a linker interface, which allows us to represent game values agnostically in the notifications.

Applying the flocking rules systematically over and over again allows you to transform code with a series of seemingly insignificant increments. You don't have to know where you're going. The accumulation of these tiny changes brushes away the jumble that hides beautiful and simple abstractions.

Look for the tiny differences, eliminate them, and name what's left.

Thank you.
