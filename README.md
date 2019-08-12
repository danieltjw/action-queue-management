# Action Queue Management

## Basic Software Requirements Specification

### 1. Overall Description

#### 1.1 Product Perspective

This product solves limitations with key(board) presses that are sent to a client application.

This program receives key presses from the user and processes it to determine which actions, and when they, should be sent to the client applications.

It does this by having a model of how actions interact with other actions in the context of when those actions are invoked.

With the aim of improve the user experience of using the client application.

#### 1.2 Product Functions

Action Queue Management [AQM]:

1. Queue actions (key presses)
2. Loop action (indefinitely until either (a) interrupted (b) paused or (b) stopped by another action)
3. Chain action (principal action adds a sequence of actions to the queue, loop-able)
4. Queue interruption (new action interrupt or pause the queue and then seamlessly resume)
5. Actions frequency limit (ignoring action if used too recently)
6. Actions behavior modification (action can affect subsequent action’s behavior)
7. Mode switching (same key presses have different effects depending on the mode)

Graphical User Interface [GUI]:

1. Progress bar (visual feedback proportional to the duration of the current action)
2. Queued actions display
