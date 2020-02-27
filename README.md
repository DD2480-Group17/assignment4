# Assignment 4 - Issue Resolution in Terasology

## Project

Name: Terasology
URL: [Terasology GitHub](https://github.com/MovingBlocks/Terasology)

Terasology is simply a Minecraft-inspired tech game.

## Onboarding experience

After assignment 3, we all wanted to change project because we had a lot of problems with setting up the project correctly. First, we tried to set up the OSS SirixDB. However, we did not manage to do that properly even when we asked for help from the teaching assistants. After a couple of hours without any success, we decided to go back to the OSS Terasology again. We felt that we did not have time to spend on the set up anymore, and some of the features in Terasology worked before during assignment 3. We found a more detailed description of how to set up Terasology, which we used this time. Now everything works as it should according to the documentation for Terasology.


## Architectural overview
Terasology is large project with 170k LOC. It is not feasible to understand all of its system in a project of this scale.
We have however taken a deeper look into some of its systems.

### [Modules](https://github.com/MovingBlocks/Terasology/blob/develop/docs/Modules.md)
Modules simply include everything that is not game engine. They include e.g. game content, gameplay mechanics.

### [Events](https://github.com/MovingBlocks/Terasology/wiki/Events-and-Systems)
Terasology's event system can send events to entities. Event processing methods can be annotated with `@ReceiveEvent`
to be called every time an event is sent.

Event processing methods must have the following method signature (From [Terasology wiki](https://github.com/MovingBlocks/Terasology/wiki/Events-and-Systems#processing-events)):
* They must have a `@ReceiveEvent` annotation
* They must be public
* The type of the first argument must implement `Event`
* The second argument must be of type `EntityRef`
* The rest of the arguments (if there are any) must implement `Component`

To define an event you extend the `Event` class and make every field private, but accessible via getters and a
constructor that supplies every field.

A priority attribute can be added to the `@ReceiveEvent` annotation. If several event processing methods are listening
to the same event they will be called in the order given by the priority attributes. Additionally, event processing
methods can consume the event, which can be useful to communicate to lower priority methods whether or not the event
has been processed or not.

## Relation to design patterns

### Controller button rebinding with events
The bind system already had functionality to rebind controller buttons, but yet nothing happened when you actually tried
to rebind a controller button. The reason for this was that the bind system was never informed that a controller
button was pressed. This was solved by creating a `ControllerButtonEvent`, which extends the existing class
`ButtonEvent`. Terasology uses this class for mouse and key events among others, so the `ControllerButtonEvent` follows
the same design pattern.

The event processing method to handle `ControllerButtonEvent` is placed in `NUIManagerInternal`, which already has event
processing methods for `KeyEvent` and others. This request is then sent to `UIInputBind`, which is the UI component
responsible for updating the button with the controller button that was just pressed. This follows the same design
pattern used for handling `KeyEvent` and others.

## Issue [Add new "Controller Settings" page #3648](https://github.com/MovingBlocks/Terasology/issues/3648)

This issue has already had some work on it, as seen in [PR #3705](https://github.com/MovingBlocks/Terasology/pull/3705).
It does not resolve the entire issue however, and so we will continue work on the issue with the PR as a starting point.


### Requirements
This issue had a large scope of changes and was open for interpretation. Therefore, we came up with the following concrete requirements.

#### 1. Rebinding: Implement rebinding of controller buttons
There is currently functionality to rebind key buttons in-game, but this does not work for controller buttons.
The GUI element responsible for detecting the new rebind does not respond when a controller button is pressed.
Adding this functionality would allow users, whose controller does not bind correctly by default, to use their controller.
It would also allow all users to configure their controllers as they wish.

#### 2. Add setting for controller axis rotation speed
The speed when rotating the camera with the controller is very slow, to the point of the game almost being unplayable.
An axis sensitivity slider for each controller axis should be added to the controller settings page. This setting should
control the camera rotation speed.

#### 3. Handle detection of plugging in and out controllers
Right now, when a controller is connected to a computer, one has to close the game and start it again for the controller to be recognized. The requirement is about making the game recognize the controller without re-starting it.

#### 4. Implement menu navigation with controller
Currently, even when the controller is connected and working, it cannot be used to navigate through the menu. One has to use mouse or keyboard to navigate through the menu. That is why the requirement is about making the controller able to do that. (Not solved due to time limitations).


### Workflow

#### Requirement 1
Input settings now allow controller buttons to be bound to actions in the same way as key and mouse input previously was.

This was done by creating a `ControllerButtonEvent`, which is sent to the event handling system in
`processControllerInput` via `sendControllerButtonEvent` inside `InputSystem`. Some additions in `JInputControllerDevice`
and `EventCopier` were required to facilitate the event. The event was then handled in
`NUIManagerInternal.controllerButtonEvent` and sent to `UIInputBind.onControllerButtonEvent` to do the binding.

There was an issue with one button that wouldn't work, and after some debugging that lead me into decompiled .class
files it turned out that the start-button on an XBOX 360 controller is actually not a button, but a key. This was easily
fixed when discovered.

Additionally, there was an attempt to allow binding of controller axises, but it is not yet finished. It was implemented
similarly to buttons, but did not work as expected due to how the bindings were created between actions and input. Some
actions, such as `forwardsMovement`, which was responsible for moving the character forwards, has a duplicate version
called `forwardsRealMovement`, which is responsible for handling controller input. The reason that these exist, and
simply aren't bound the same way as keys, is that these have a single binding for forwards and backwards, whilst the
key bindings consist of two bindings. There was no translation between these bindings available. Furthermore, the
input settings page did not display the singly bounded actions, such as `forwardsRealMovement`, and plainly adding them
in the list would be confusing to the user, since there would be duplicate bindings available for many actions.

Large parts of these functionalities rely on user input, and are thus difficult to test. The controller button event was
however tested in `InputSystemTest`.

#### Requirement 2
A setting for controller axis sensitivity was added to the controller settings page for each axis. Axis names
were made understandable. POV axis (D-pad) was removed, since it functions like a button.

The GUI changes were made in `addAxis` and `addInputSection`, inside `ControllerSettingsScreen`. The  slider was bound to the
property value `sensitivity` in `Axis`. Axis ids such as `rx` were mapped to proper names such as `Right Joystick X-Axis`
in `axisMap` in `ControllerConfig`.

##### Before & After
<img src="/images/controller-settings-before.png" width="420"> <img src="/images/controller-settings-after.png" width="420">

##### Requirement 1 & 2 UML

![](/images/uml-controller-settings.png)



#### Requirement 3

To handle automatic recognition of the controller on connecting and disconnecting it while the game was running, different methods were looked up. Moreover, there is a comment in the code that says that the **lwjgl** library has the interface for such a functionality. However, this interface has no implementation.

Furthermore, the method for discovering the controllers that are connected has a limited functionality in that the method does not allow you to poll for **changes** in the list of the connected controllers on **all** the well-known operating systems after the game had already been started. However, the library allows to poll the list of the connected controllers when the game is started and this list is fixed regardless of whether some changes in the controllers connection happen during the game.

These two points above made this requirement more challenging that was first thought. If the requirement is wanted to be implemented as is, using JNI (Java Native Interface) was the only option that was found to solve the problem. This needs to be done in order to listen for the event of plugging and unplugging the controller, and in order to poll a list of the controllers that are plugged in after the game had already been started. However, this would need us to implement the functionality in C and compile it for every desired operating system (such as windows, linux, macOS), then write code in java to load the compiled files depending on the type of the operating system that is running. This make the requirement more complicated.

Therefore, a compromise was chosen to be implemented. A button called "Refresh Controllers" was added on the "Main Menu". When this button is clicked, the game polls the list of the connected controllers and invalidates the "Controller Settings" menu that has the options for the controllers. The invalidation is done to force the AssetManager to reload the "Controller Settings" menu to get the list of the new controllers instead. Moreover, a cleanup is done for the threads spawned by the **lwjgl** library when creating a new environment to re-poll the connected controllers list in order not to exhaust operating system resources.

The changes that were done were the following:

* A `public` `void` method called `refreshControllerList` was added in `LwjglInput` class. This class handles, among others, initialization of other class instances that handle the keyboard, mouse and controllers at the start of the game and setting up the input system correctly in the game. `refreshControllerList` method refreshes the controllers list by re-initialization of the `JInputControllerDevice` instance that re-polls the new controllers list every time it is created, and resetting the input system in the game. The added method also performs a cleanup for the threads spawned by the **lwjgl** library in `JInputControllerDevice` every time a new `JInputControllerDevice` is created. This is done in order not to exhaust operating system resources. Moreover, the added method invalidates the current Controller settings instance from current `AssetManager` if that screen was already loaded and saved in the AssetManager.

* The constructor in `JInputControllerDevice.java` was edited to poll the list of plugged-in controllers every time an instance of `JInputControllerDevice` is created and to update the current controllers list.

* A button was added on the "main menu" screen by editing three files: `menu.lang`, `menu_en.lang` and `mainMenuScreen.ui`.

* The `MainMenuScreen` class was changed to make the "Refresh Controllers" button subscribe to the click event, and, when clicked, it calls `refreshControllerList()` method on the `LwjglInput` instances that exist in the engine.

Some notes:
* Re-polling the list of connected controllers works only for windows 10 for now due to the limitation of the **lwjgl** library as mentioned above.

* It was hard to write unit tests because it was hard to simulate plugging and unplugging the controller.

UML over that changes done to fulfill requirement 3 follows:
![UML](/images/uml1.jpg)

#### Requirement 4
Not solved due to time limitation.

#### Plan to continue working with the requirement and the issue in general
In order to complete working on requirement 4, the following is needed:
* Need to implement and test for other operating systems (e.g. linux and macOS) and other types of controllers.
* Need unit tests.
* Need to have a way to not need the "Refresh Controllers" totally. Maybe JNI is a good option. Or move "Refresh Controllers" button to be in "Controller Settings" menu instead.

Regarding the rest of the issue, the issue is very broad and is connected to other issues such as ([issue #2125](https://github.com/MovingBlocks/Terasology/issues/2125)), and has a lot of features that could be added and are related to this issue. One could continue by building upon what was already done by implementing new feature such as Requirement 4 here, and looking at the related issues.

However, it seems that some features are harder to implement if it is continued to use the same library **lwjgl**. One of these features is the automatic recognition of plugging and unplugging. It could be worth changing this library totally if this would offer more flexibility for the whole infrastructure.

---

## Issue [Item tooltip on tool changing #1514](https://github.com/MovingBlocks/Terasology/issues/1514)

### Requirements

#### 1. Display the name of the tooltip when switching tooltip
Currently, the Terasology only has a toolbar where you can see the available tooltips and if you hover them you see the name of the toolbar. The community of Terasology now wants a small display that shows the name of the tooltip when you switch slots from 0...9.

#### 2. Fix the location of the display message
After we re-used and implemented some of the code from an old PR (that the community of Terasology closed 2018) we saw that the location of the display was not adjustable relative to the screen.

#### 3. Fade the display after a couple of seconds if we don't switch tooltip
After we re-used and implemented some of the code from an old PR (that the community of Terasology closed 2018) we saw that the display does not fade after a time, which would be nice because otherwise it might affect or disturb the player.

### Project plan

To be able to fulfill the requirements we need to:
* Read necessary documentation from Terasology such as wikis for NUI and Modules
* Import and implement the module Inventory to our local Terasology.
* Modify the NUI inventoryHud.ui to display the name of the item at a correct place at the screen.
* Modify the class InventoryHud.java to be enable to fading text.
* Try to create tests or modify existing test for the changes we will made.

### Workflow
Terasology uses modules to apply new packages and functionality. This is to avoid faulty code to be pushed to the core game and can be tested before. Somenone tried to solve issue 1514 but failed, we used his code to get inspiration on how we could solve this issue, however his code did not work. Thus we corrected his code to be able to use it as base by implementing missing classes and values in to the code. The work flow can be illustrated by following image: ![workflow](/images/1514.png).
By adding functionality to the InventoryHud.java and inventoryHud.ui we can keep the original workflow and create the desired functions.

#### Requirement 1.
Requirement 1.

After we understood the architecture related to this issue and got some help from one of the members in the Terasology community,
we managed to implement some of the old code from an PR from 2018. The changes that we made affected the module Inventory, the class InventoryHud.java and the NUI inventoryHud.ui. To be able to display the name of the current tooltip, we needed to create a new core widget as a `UIText` and initialise it with the abstract method `initialise()`. We also needed to add the `UIText` as an JSON object in content of the NUI inventoryHud. To be able to know which item the localPlayer where currently holding, we used the class from PR 2018 called `CurrentSlotItem` and the method `get()` which returns the name of the current item. By doing that, we were able to show a static display message of the name of the current tooltip and make it change whenever we switch slots from 0...9. The display message where static.

The changes for Requirement 1 are following the design-pattern for the Inventory module. We use the method `find()` to initialise the `UIText` widget, which they had used for initialising the other widget `Crosshair`. The new class `CurrentSlotItem` that we implemented, is also following the design-pattern for the Inventory module where the classes are private and extend the suitable binding for the purpose of the class.

#### Requirement 2.
We manage to change the position of the display message so it is located in a more suitable place. In the NUI `inventoryHud`, we needed to create a new JSON-object for the `UIText` widget toolTipText and add it to the existing content array in the inventoryHud.ui. The type of the content is relativeLayout, which means that the elements in the array are relative to each other. Therefore, we needed to add offset to each of the other elements in the array. For instance, we needed to add an offset of 3 from the TOP of the other widget `toolbar`. This works very well when the game screen/the window of the game is large or in full size, but it is not adjustable for smaller windows.

To make the `UIText` adjustable, we tried to use different JSON attributes and search for similar cases in the other NUI files, but did not find any good solution for it. To make the display message adjustable, we think we will need to create a new NUI like the NUI `healtHud` (the hearts above the toolbar). The hearts are always an offset of 60 from the bottom of the window, and not related to any other widget in the game. If we add the toolTipText to the content array in inventoryHud.ui, the display message will be in the middle of the other two widgets, which makes it difficult to set a offset that will be accurate for both of them at the same time if we resize the game window. We created a class `InventoryText.java` and moved all the code from the `InventoryHud.java` which were related to the toolTipText and the animation of it. We also created a new NUI called `inventoryText.ui`. However, we did not manage to make it appear in the game, even if we used the guide for creating and changing NUI in Terasology. We will probably need a couple of more hours to make the new NUI appear properly in Terasology.

The changes made in Requirement 2 are following the design-pattern for the Inventory module and the structure of the NUI in Terasology. The new `UIText` were implemented as an JSON-object in the existing NUI `inventoryHud`. The new class `InventoryText.java` followed the same structure and design as the `InventoryHud.java`. For instance, the class `InventoryText.java` extends from `CoreHudWidget` and uses the abstract method `initialise()` to initialise the new `UIText` widget. The new NUI had the same JSON structure as `healtHud.ui` and `inventoryText.ui`.

#### Requirement 3.
We manage to make the toolbar to appear if you select an item and then disappear after 2 seconds if no new item is selected. This was done by implementing an `AnimationThread` that monitors the condition of the player and sets the `UIText` to invisible after 2seconds. However, we did not have time to finalize the code and the toolbar so it cannot fade out. The function also contains a bug which makes the toolbar static sometime and out-of-synq with the players change of items. In order to solve this problem we have to modify the `AnimationThread` to be able to listen to calls at the same time as it is asleep. This could be solved by using reentrant locks and condition-variables to make the thread wait for a certain event.

In order to make the toolbar fade we need to modify `UIText` and the `Lwjgl` shaders displaying the text to be able to support transparency fades. This is a huge task and we do not have time to implement changes to the core engine.

The implementation of `AnimationtThread` works well with the design-pattern because they are using several threads to do different animation tasks. The problem with the implementation is that it is similar to a spin-lock (rapidly checking one condition) which can effect performance. This was not noted during test so we assumed the effect being minimal. The class is also following the design patter of having brackets of all loops and if-statments, and it uses an already existing class to produce the wanted result.


## Requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

## Existing test cases relating to refactored code

* Issue [Add new "Controller Settings" page #3648](https://github.com/MovingBlocks/Terasology/issues/3648), requirement 3:  
No existing test cases were found for the three classes that were edited.

## Test results

Overall results with link to a copy or excerpt of the logs (before/after
refactoring).

* Issue [Add new "Controller Settings" page #3648](https://github.com/MovingBlocks/Terasology/issues/3648), requirement 3:  
  * Same test results were obtained before and after implementing the requirement. The failing test cases are not related to the requirement being implemented.  
  * The patch was also marked as "Successful in 14m — No new or fixed alerts" by the CI server connected to the base Terasology repo on github, which confirms that the failing test cases are not related to the requirement being implemented.
  * Test results (or logs) were saved before ([here](/test_reports/issue-3648-req-3/before-locally-develop-branch) - using `develop` branch) and after ([here](/test_reports/issue-3648-req-3/after-locally-RefetchControllersByClickingOnMenuButton-branch) - using `RefetchControllersByClickingOnMenuButton` branch) implementing the requirement.

## Patch/fix

The fix can be copied or linked to (git diff).
* [Issue #3648](https://github.com/MovingBlocks/Terasology/issues/3648) Requirement 3 patch: [PR #3838](https://github.com/MovingBlocks/Terasology/pull/3838).

Optional (point 4): the patch is clean.
* [Issue #3648](https://github.com/MovingBlocks/Terasology/issues/3648), Requirement 3, [PR #3838](https://github.com/MovingBlocks/Terasology/pull/3838): it is considered clean because changes were done in the code in a way that makes minimal changes to the design pattern of the whole project.

Optional (point 5): considered for acceptance (passes all automated checks).
* [Issue #3648](https://github.com/MovingBlocks/Terasology/issues/3648) Requirement 3, [PR #3838](https://github.com/MovingBlocks/Terasology/pull/3838): the patch was marked as "Successful in 14m — No new or fixed alerts" by the CI server that is connected to the base Terasology repo on github. Moreover, same test results were obtained before and after implementing the requirement. The CI server results confirm that the failing test cases are not related to the requirement being implemented.  

---

## Effort spent

For each team member, how much time was spent in

1. plenary discussions/meetings;

1 hour in total for each student ??

2. discussions within parts of the group;

30 min to 1 hour in total for each student?? or none??

3. reading documentation;

`Marcus`:
* 3h, A lot of the time went to read documentation about terasology and other projects, for example on how to set up Gradle and solve issues with different Java version. However, I estimate that i spent 3h reading
documentation about Terasology.


`George`:

About 4-5 hours in total including reading documentation. A lot of documentation, tutorials and information about projects, issues, pull requests were read. Following is a summary:
* Docker.
* Gradle.
* [SirixDB project](https://github.com/sirixdb/sirix) and its configuration information.
* [Terasology project](https://github.com/MovingBlocks/Terasology) documentation, wiki, tutorials, including the following: [Home](https://github.com/MovingBlocks/Terasology/wiki), information about the architecture including [Entity System Architecture](https://github.com/MovingBlocks/Terasology/wiki/Entity-System-Architecture), [Events](https://github.com/MovingBlocks/Terasology/wiki/Events), [Events and Systems](https://github.com/MovingBlocks/Terasology/wiki/Events-and-Systems), [TutorialNui Home](https://github.com/Terasology/TutorialNui/wiki), [NUI Quick Start](https://github.com/Terasology/TutorialNui/wiki/Quick-Start), [Nui Core Widgets](https://github.com/Terasology/TutorialNui/wiki/Core-Widgets), [Containers and Layout](https://github.com/Terasology/TutorialNui/wiki/Containers-and-Layouts), [Developing Modules](https://github.com/MovingBlocks/Terasology/wiki/Developing-Modules), [CONTRIBUTING.md](https://github.com/sirixdb/sirix/blob/master/CONTRIBUTING.md), and [issue #3648](https://github.com/MovingBlocks/Terasology/issues/3648) and [PR #3705](https://github.com/MovingBlocks/Terasology/pull/3705).
* [Elasticsearch Project](https://github.com/elastic/elasticsearch) as an alternative.
* Even reading and seeing videos about Intellij to know how to navigate in a better way through the project.
* Even looking up some things about UML to draw UML of the changes done in the project.

4. configuration and setup;

`Marcus`:
* 7h, alot of projects where attempted which are listed bellow:
* [SirixDB project](https://github.com/sirixdb/sirix), Updated to Java 13, updated to Gradle 6, nothing worked to get the program to work with CMD on Windows or Intellj. The project
could not find the variable var which should be included in Java 13. Thee problem where not solved and abandoned because all group members had different issues.
* Teasology took time to set up, it did not work with Intellij and I needed to downgrade to Java 1.8. After several hours bug fixing the following sequence enables the game
to be executed from CMD. gradlew, gradlew jar game, without jar bug's appeared in the game.

`George`:

About 4-5 hours. The following was tried:
* First [SirixDB project](https://github.com/sirixdb/sirix) was tried. I installed Docker, gradle and Java JDK 13 on linux VM on my computer (I have Windows) and tried to run the docker image of `SirixDB project`. However, it did not succeed. This needed reading about docker and gradle before actually doing the installation steps.

* Second, [Terasology project](https://github.com/MovingBlocks/Terasology) was tried. We had problems with this project as well in lab 3. There was some documentation missing in the README.me of the [Terasology project](https://github.com/MovingBlocks/Terasology) about running `gradlew jar` before running `gradlew game`, which lead to problems. Also, there was no success in trying to run the game fully from Intellij. It could only run fully from the terminal. Intellij could only be used for debugging purposes. Discovering `gradlew jar` and knowing the exact order of running `gradlew` commands took some time.

5. analyzing code/output;

`Marcus`:
* 5h, the structure for implementing the functionality where not explained in the documentation, because of that earlier pull-requests and code where studied in order to understand how the
solution should be implemented.

`George`:

See question 8.

6. writing documentation;

`Marcus`:
* Wrote documentation to InventoryHud.java and and animation thread.
* Wrote documentation about workflow and added images.
* TODO

`George`:

About 4-5 hours. The following was done:
* Contribution to README.md file.
* UML took some time to draw manually.
* Writing detailed text in [pull request #1](https://github.com/DD2480-Group17/Terasology/pull/1) of requirement 3 for [issue #3648](https://github.com/MovingBlocks/Terasology/issues/3648), and detailed Commit message for the PR.
* Writing code documentation in class `LwjglInput`.

7. writing code;

`Marcus`:
* 8 hours were spent on adding support for adjusting location on tooltipbar and added functionality that the toolbar diapper after 2seconds and reapper if item is switched.
This was a bit hard to get to work with the original code.

`George`:

See question 8.

8. running code?

`Marcus`:

* see question 8 and 5, running code where included there.

`George`:

About 10-11 hours in total of analyzing code, writing code, and running code. Following are more details:
* Writing code to add the function of the requirement. The design of the code was changed iteratively to make changes the do not break the design pattern of the project as much as possible. This was hard because there is not much documentation on how most of the classes in the project should be used.

* Therefore, a lot of tracking, tracing, debugging by breakpoints, reading through the code of commit in the [PR](https://github.com/MovingBlocks/Terasology/pull/3705) that we started from and a lot of analysis was done in order to know where to introduce the changes in the code.

* A lot of re-compiling and re-running the code to run the game was done to test if the requirement that was implemented works. Re-compiling and re-running the code took some time every time it was done, which slowed the process a little bit.

* The most correct ways of introducing the changes were not discovered all at once. By re-running the unit tests, one more test failed which revealed that the assumptions that I made at first about the design pattern of the project was wrong. Therefore, more searching through the code was done, and re-changing some parts of the code fixed the failed unit test case.

* Re-running the unit tests takes some time, which slowed the process re-running the tests a little bit.

For setting up tools and libraries (step 4), enumerate all dependencies
you took care of and where you spent your time, if that time exceeds
30 minutes.

## Overall experience

* What are your main take-aways from this project? What did you learn?

Learning how to contribute to an open-source project, and navigate through code that is not fully documented.

* Optional (point 6): How would you put your work in context with best software engineering practice?

* Optional (point 7): Is there something special you want to mention here?
