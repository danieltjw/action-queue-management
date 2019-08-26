# Action Queue Management

## Requirements Specification

## 1. Overall Description

### 1.1 Product Perspective

This product primarily solves limitations with key(board) and mouse inputs that are sent to a client application, with the overall aim to enhance the user experience through additional functionalities such as a comprehensive queue management and graphical overlay.

This program receives key presses and mouse inputs from the user and processes it to determine which actions, and when they, should be sent to the client applications.

It does this by having a model of how actions interact with other actions in the context of when those actions are invoked.

### 1.2 Product Functions

**Action Queue Management [AQM]:**

1. Queue Action (key presses)
2. Loop Action (indefinitely until either (a) interrupted (b) paused or (b) stopped by another action)
3. Chain Action (principal action adds a sequence of actions to the queue, loop-able)
4. Substitute Action (principal action chooses from a list of appropriate substitute action)
5. Action Prerequisites
6. Action Resources
7. Action Effects
8. Meta Action

**Graphical User Interface [GUI]:**

1. Progress bar (visual feedback proportional to the duration of the current action)
2. Queued actions display

---

## 2. External Interface Requirements

### 2.1 User Interfaces

**Keyboard & Mouse: User > Program**

User interacts with the program through key(board) presses and mouse inputs to (a) issue Actions and (b) switch Modes. Advanced configurations can be changed in a text(ini) file.

**GUI: User < Program**

User receive feedback from a GUI on action currently being performed and actions being queued. This GUI update should happen immediately once the key press is made to ensure feedback is prompt.

GUI will indicate the time taken in real time for the completion of the current action. Preferably this should be done through a visual medium like a progress bar.

The progress bar should be visually proportionate to the duration of the action to give users a better sense of when the action completes. (An action that takes 1 sec to complete should be 1/3 the visual length of an action that takes 3 secs to complete)

### 2.2 Hardware Interfaces

User interacts with the program though the use of key presses and mouse inputs:

- Single Key (pressing “1” for an action, "F12" to switch modes)
- Combination key (pressing a modifier key “Shift” + “1” for another action)
- Mouse inputs provide additional context that may modify actions

### 2.3 Software Interfaces

**Client Application**

The program will dispatch actions on behalf of the user, through virtual key presses and mouse clicks, to the client application.

The user's key presses and mouse inputs should not interfere with the virtual key presses and mouse clicks sent by the program. For example, if the user holds down the "Shift" key when the program sends a virtual key press of "1", the client application should register "1" and not a combination of "Shift" + "1".

The program should be able to simultaneously receive key presses and mouse inputs from the user and dispatch actions from the Action Queue Manager to the client application at all times.

The use of virtual key presses introduces additional limitations such as the client application can not register combination key presses (“Shift” + “1”) simultaneously but will require a slight delay (~50ms) between those key presses. Various methods can be used to address this issue.

### 2.4 Communications Interfaces

**Remote Server**

As the client application ultimately forwards those key presses (virtual or otherwise) to a remote server, network latency will also be an important factor. The program’s action queue manager may need to add a short buffer between actions to ensure a previous action that arrived at the remote server slightly late, due to network latency, will not block the current action. This buffer should be configurable in the text(ini) config file.

---

## 3 System Features

### 3.1 Queue Action

An action can be 1 of 2 types: (1) **Normal Action** (2) **Blocking Action**.

**Type 1: Normal Action**

- Integer: _rest_millisecond_

**Type 2: Blocking Action**

- Integer: _rest_millisecond_
- Integer: _blocking_rest_millisecond_

User initiates an action (next action) resulting in these possible outcomes:

1. No previous action is in progress and next action is started immediately
2. If next action is **Normal Action**, it will start after previous action's _rest_millisecond_
3. If next action is **Blocking Action** it will only start after both
   - previous **Blocking Action**'s _blocking_rest_millisecond_
   - previous **Normal Action**'s _rest_millisecond_

```
For example:

Blocking Action (rest_millisecond: 1000, blocking_rest_millisecond: 3000)
Normal Action (rest_millisecond: 1000)

Time elapsed:

0_________1_________2_________3_________4
 Blocking                      Blocking

0_________1_________2_________3_________4
 Blocking  Normal              Blocking

0_________1_________2_________3_________4
 Blocking  Normal    Normal    Blocking

This blocking action will prevent another blocking action from starting for 3 secs,
but will allow 2 normal actions to be performed before the next blocking action.
```

REQ-1: Queued Actions are started at the appropriate time based on properties of previous actions

REQ-2: New actions are added to the queue (LIFO by default) in real time

### 3.2 Loop Action

**Loop Actions** allow for an action to be repeated indefinitely with a single key press.

**Loop Actions** interact with other actions in 3 ways:

1. No previous **Loop Action** is in effect, **Loop Action** will continuously trigger when there are no pending actions
2. **Loop Action** itself is in effect, current **Loop Action** will stop (toggle on / off)
3. Another **Loop Action** is in effect, current **Loop Action** will stop and the new **Loop Action** will start when appropriate

REQ-1: Loop Actions are started at the appropriate time and continue indefinitely

REQ-2: Loop Actions can be interrupted or paused by new actions and resume seamlessly afterwards

REQ-3: Loop Actions are toggled off when users use the same key press

REQ-4: Loop Actions can be replaced by another Loop Action

### 3.3 Chain Action

**Chain Action** create a sequence of actions to be performed. While those actions can be started separately if needed, grouping them into a **Chain Action** can reduce the number of key presses.

REQ-1: Chain Actions add a sequence of actions to the queue (this decomposition only happens at the point the Chain Action is started and not while it is in queue)

REQ-2: Chain Actions can be looped

### 3.4 Substitute Action

**Substitute Action** allows a single key press to choose an action from a group of possible actions. This allows for the user to logically group separate actions that perform a similar function together.

REQ-1: Substitute Actions chooses an appropriate action from a list of possible actions

REQ-2: Substitute Actions can be looped

### 3.5 Action Prerequisite

Actions will only be performed if prerequisites are met. All Actions, and Substitute Action in particular, will use these prerequisites to evaluate which action to choose.

**Implicit prerequisites:**

Before the next action in the queue is started, the Action Queue will rest for _rest_millisecond_ (and _blocking_rest_millisecond_ if applicable).

**Explicit prerequisites:**

- Action may only be reused after some time have passed
- Action may require a certain number of resources to use
- Action may only be used within a specified duration of another action

These prerequisites may be combined in complex ways. For example, an Action {B} can only be used for 20s after Action {A} has been used. Action {B} will require 1 resource to use. Action {b} can only be used once every 5s.

REQ-1: Actions need to check if all prerequisites are

### 3.6 Action Resources

Resources can be created by:

- An action
- Automatically after some time has passed

REQ-1: The Action Queue Manager has to keep track of available resources

### 3.7 Action Effects

Action can change the implicit or explicit prerequisites of the next suitable action(s) for a:

- duration of time
- number of times

REQ-1: The Action Queue Manager has to keep track of effects, their duration and number of uses

### 3.8 Meta Action

Actions might in some cases also affect the Action Queue Manager's (AQM) behavior itself.

- Reset the Action Queue (clear the Queue of all actions)

- Pause the Action Queue (temporarily stop all queued actions but allow new actions through)

- Switch AQM Application (changes how the AQM maps key presses to actions)

- Switch AQM Mode (changes how the AQM processes actions on the queue)
  - LIFO (default: Last In, First Out)
  - FIFO (First In, First Out)
  - Prefer responsiveness (actions are preformed according to queue order)
  - Prefer throughput (non-blocking actions may be delayed if throughput can be increased)

REQ-1: The AQM will be able to modify its behavior / logic when Meta Actions are performed

## 4 Other Nonfunctional Requirements

### 4.1 Performance Requirements

1. Register new actions: < 2 ms (99% of the time)

The Queue Manager should register new actions inputs from the User within 2ms, 99% of the time. The default time resolution of 15.6 ms can be improved through the use of the hardware counter but at the cost of higher CPU usage.

2. Dispatch actions: < 0.1 ms

The Queue Manager action dispatch timer should have a resolution of under 0.1 ms. This will help avoid performance loss over time.

3. CPU usage: < 1 %

The program should use under 1 % of a typical CPU. High resolution time keeping needs to be balanced with its high CPU cost.
