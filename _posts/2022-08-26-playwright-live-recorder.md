Recently I've been programming, helping the test team transition to a new Automated Testing framework.
We're moving from Cypress to Playwright. Right away there are a lot of things I like!
But our biggest pain points weren't fixed.
I created an amazing lib.

The first problem we have, is loading our page is really slow. No lib required there, I set up recording a HAR and `routeFromHar` (fallback). Slashed the load time.
Next challenge was trickier.
When writing tests, I have to keep rerunning the test to get the next line correct.

Ideally I'd like to live-code the next line(s) of test code, at the currently running state in the browser.
quick note: after starting on this, I pulled down the latest version of Playwright and tried their new recorder. Although a welcome addition, no silver bullet.

Ok, live-coding.
I'm able to drop a line at the end of my test (or the middle for that matter) and have the headed browser wait until window close.
In the console I've exposed a method to pump code back to the test context to be run.
![startLiveCodingpng](https://user-images.githubusercontent.com/11726379/186935047-9e5234a7-f98c-4e30-bdc4-b9ad060ea3e5.png)



So from the browser console, I ought to be able to type something like `PW_eval('page.playwrightThings')` and it would try to do it.
Let's try... success!!
[PW_eval](https://user-images.githubusercontent.com/11726379/186935193-102c392d-cd6a-43f4-84e2-965e7ba01b9b.gif)

------

Next pain point. Figuring out those selectors is a pain. I wish I could find a recorder that was more configurable or programmatic about narrowing to more idiomatic testing code instead of throwaway recorder code.
Couldn't find anything good out there.
Began writing my own.

A couple days later, we have some event listeners for pressing key chords and moving and clicking the mouse. And we hook up that programmatic part.
This is using a lib called 'finder' to get css shortest-path. https://medv.io/finder/finder.js
![PW_recorder1](https://user-images.githubusercontent.com/11726379/186935360-44b54764-1948-409f-971f-043480a21b1d.gif)

</br></br></br></br>

That's helpful, let's plumb those recorder/selector rules into more of a match=>code pairs.
And let's make it easy to add the code to the test.
Originally I output the test code lines to the Debug Console (vscode), but eventually was able to emit them directly into the test file.
![PW_recorder2](https://user-images.githubusercontent.com/11726379/186935702-0b2826f2-11e9-4865-bac2-032533cab8dc.gif)

</br></br></br></br>

You may have noticed the PlaywrightRecorder.startLiveCoding got a second param... it's best to evaluate the code passed in not just in the NodeJS playwright testing context, but in the actual test function, so that locally scoped things are available.
This did the trick!

You'll also notice a key combo to re-write and re-run the last executed statement. With this in place you can just gesture to the control you want to interact with, then fill in the details, and it updates in file.
Great!
![PW_recorder3](https://user-images.githubusercontent.com/11726379/186935808-ffabb602-3cd4-4383-aca0-a8dd69daa2d0.gif)

</br></br></br></br>
## Recorder Rules
So, how about those rules... I said they're configurable/programmatic. Let's take a look.

The first rule is simplest, it just falls back to a finder lib that returns the css selector as a string. From there the code output takes the selector and puts it in a standard `await page.locator(...).click()` statement.
![PW_recorderRules1](https://user-images.githubusercontent.com/11726379/186935879-d470b96a-036a-41d1-ab19-860ce16f6a47.png)


The next rule is for matching on 'data-testid' attributes. We'll put it first since it's more specific, and fall through to the other. If there's no match, just return undefined and it'll try the next rule down the line.
![PW_recorderRules2](https://user-images.githubusercontent.com/11726379/186935950-b345e8b0-09b7-46cc-a4c3-ad5d375c00d2.png)


The third rule is for .nav-link elements, bootstrap stuff I think. In any case, we'd like to grab the .nav-link with the whatever the given text in on the element under the cursor.
![PW_recorderRules3](https://user-images.githubusercontent.com/11726379/186935994-38db2da7-ba2d-4925-a148-9b3c7995e3a5.png)


Let's revisit that data-testid rule. I've encountered issues where the same control is twice on the same screen, and as a user I can tell the two sets apart, but playwright will want only one selector.
Let's detect which nth matching data-testid it is and return that.
![PW_recorderRules4](https://user-images.githubusercontent.com/11726379/186936037-7a5404e8-3426-408c-9bd7-434fa5f044a0.png)


and here's a gif of that - I'm modifying the page to have one, then two elements with the same data-testid. At first it displays data-testid=d1, but once it's not unique it returns the object with { selector, nth } values. The output code handles both cases.
![PW_recorderRules4](https://user-images.githubusercontent.com/11726379/186936094-209b055a-1bf7-40a7-ac44-8d3b91b62548.gif)

</br></br></br></br>

## Debugging rules
Speaking of bugfixes and improvements, the recorderRules code is debuggable right in your browser.
![PW_recorderRulesDebug](https://user-images.githubusercontent.com/11726379/186936138-dc108bae-bd4c-457f-a746-42972d357c86.gif)

## Workbench for testing new rules in browser

Additionally there's a lot exposed at your fingertips to use right in the console, when trying to author these rules.
You can access 'el' and try matcher code out on it. When you think you're close, call PW_addRule('your matcher code') and it'll generate add it to the rules file.
![PW_recorderRulesCreateRule](https://user-images.githubusercontent.com/11726379/186936316-caf2380b-9ebe-449c-a5e8-7021341b3ab5.gif)

------

OK, so I created a dumb rule that takes precedence that returns the closest parent 'div' instead of the element under the cursor... not very useful, but it works.
oops! I hovered over something that didn't have a parent div. and it crashed the reload process.
Bugfixing time.

Normally it's watching and can hot-reload your rules file. The browser even updates in the source and you can add/move/remove breakpoints. It's pretty great.

That's all I have so far, but I'm starting to work on making this system page-object-model aware. I'd like to take the url slug and create a mapping to a page object file. From there I'm figuring out some good conventions that allow for a smooth workflow.

It's tricky being able to execute the code as it's being written. It may be re-running the test is required if you add to a page-object file.
I do intend to make the page-object generating and user editing work hand in hand. That way you're not painted into a corner from either side.
I'm working with my organization to be allowed to open source this. My hope is it'll get traction and Microsoft @playwrightweb will take notice!
Thanks!
