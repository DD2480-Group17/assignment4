# Assignment 4 - Issue Resolution in Terasology

## Project

Name: Terasology
URL: [Terasology GitHub](https://github.com/MovingBlocks/Terasology)

One or two sentences describing it

## Onboarding experience

Did it build and run as documented?
    
See the assignment for details; if everything works out of the box,
there is no need to write much here. If the first project(s) you picked
ended up being unsuitable, you can describe the "onboarding experience"
for each project, along with reason(s) why you changed to a different one.

## UML class diagram and its description

Optional (point 1): Architectural overview.

Optional (point 2): relation to design pattern(s).

## Issue [Add new "Controller Settings" page #3648](https://github.com/MovingBlocks/Terasology/issues/3648)

This issue has already had some work on it, as seen in [PR #3705](https://github.com/MovingBlocks/Terasology/pull/3705).
It does not resolve the entire issue however, and so we will continue work on the issue with the PR as a starting point.

### Requirements

#### 1. Rebinding: Implement rebinding of controller buttons
There is currently functionality to rebind key buttons in-game, but this does not work for controller buttons.
The GUI element responsible for detecting the new rebind does not respond when a controller button is pressed.
Adding this functionality would allow users, whose controller does not bind correctly by default, to use their controller.
It would also allow all users to configure their controllers as they wish.


#### 2. Add setting for controller axis rotation speed
The speed when rotating the camera with the controller is very slow, to the point of the game almost being unplayable.
An axis sensitivity slider for each controller axis should be added to the controller settings page. This setting should
control the camera rotation speed.

#### 3. Implement menu navigation with controller

#### 4. Handle detection of plugging in and out controllers

## Issue [Item tooltip on tool changing #1514](https://github.com/MovingBlocks/Terasology/issues/1514)

### Requirements

#### 1. Display the name of the tooltip when switching tooltip
Currently, the Terasology only has a toolbar where you can see the available tooltips and if you hover them you see the name of the toolbar. The community of Terasology now wants a small display that shows the name of the tooltip when you switch slots from 0...9.

#### 2. Fix the location of the display message
After we re-used and implemented some of the code from an old PR (that the community of Terasology closed 2018) we saw that the location of the display was not adjustable relative to the screen.

#### 3. Fade the display after a couple of seconds if we don't switch tooltip
After we re-used and implemented some of the code from an old PR (that the community of Terasology closed 2018) we saw that the display does not fade after a time, which would be nice because otherwise it might affect or disturb the player.

### Workflow

Terasology uses modules to apply new packages and functionality. This is to avoid faulty code to be pushed to the core game and can be tested before.
Somenone tried to solve issue 1514 but failed, we used his code to get inspiration on how we could solve this issue, however his code did not work. Thus we 
corrected his code to be able to use it as base by implementing missing classes and values in to the code. The work flow can be illustrated by following image: ![workflow](/images/1514.png).

We will modify the file inventoryhud.ui to display the name of the item at a correct place and modify InventoryHud.java to enable fading text.

### Requirements affected by functionality being refactored

Optional (point 3): trace tests to requirements.

### Existing test cases relating to refactored code

### Test results

Overall results with link to a copy or excerpt of the logs (before/after
refactoring).

### Patch/fix

The fix can be copied or linked to (git diff).

Optional (point 4): the patch is clean.

Optional (point 5): considered for acceptance (passes all automated checks).

## Effort spent

For each team member, how much time was spent in

1. plenary discussions/meetings;

2. discussions within parts of the group;

3. reading documentation;

4. configuration and setup;

5. analyzing code/output;

6. writing documentation;

7. writing code;

8. running code?

For setting up tools and libraries (step 4), enumerate all dependencies
you took care of and where you spent your time, if that time exceeds
30 minutes.

## Overall experience

What are your main take-aways from this project? What did you learn?

Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?
