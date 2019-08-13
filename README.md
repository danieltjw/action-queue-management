# Action Queue Management

## Requirements Specification

### 1. Overall Description

#### 1.1 Product Perspective

This product primarily solves limitations with key(board) presses that are sent to a client application, with the overall aim to enhance the user experience through additional functionalities such as a comprehensive queue management and graphical overlay.

This program receives key presses from the user and processes it to determine which actions, and when they, should be sent to the client applications.

It does this by having a model of how actions interact with other actions in the context of when those actions are invoked.

#### 1.2 Product Functions

**Action Queue Management [AQM]:**

1. Queue actions (key presses)
2. Loop action (indefinitely until either (a) interrupted (b) paused or (b) stopped by another action)
3. Chain action (principal action adds a sequence of actions to the queue, loop-able)
4. Substitute action (principal action chooses from a list of appropriate substitute action)
5. Queue interruption (new action interrupt or pause the queue and then resume seamlessly)
6. Action frequency limit (ignoring action if used too recently)
7. Action behavior modification (action can affect subsequent action’s behavior)

**Graphical User Interface [GUI]:**

1. Progress bar (visual feedback proportional to the duration of the current action)
2. Queued actions display

**Program Configuration [PC]:**

1. Mode switching (changes the context of the AQM and key presses)

---

### 2. External Interface Requirements

#### 2.1 User Interfaces

**Keyboard: User > Program**

User interacts with the program mainly through key(board) presses to (a) issue Actions and (b) switch Modes. Advanced configurations can be changed in a text(ini) file.

**GUI: User < Program**

User receive feedback from a GUI on action currently being performed and actions being queued. This GUI update should happen immediately once the key press is made to ensure feedback is prompt.

GUI will indicate the time taken in real time for the completion of the current action. Preferably this should be done through a visual medium like a progress bar.

The progress bar should be visually proportionate to the duration of the action to give users a better sense of when the action completes. (An action that takes 1 sec to complete should be 1/3 the visual length of an action that takes 3 secs to complete)

#### 2.2 Hardware Interfaces

User interacts with the program though the use of key presses:

- Single Key (pressing “1” for an action, "F12" to switch modes)
- Combination key (pressing a modifier key “Shift” + “1” for another action)

#### 2.3 Software Interfaces

**Client Application**

The program will dispatch actions on behalf of the user, through virtual key presses, to the client application.

The use of virtual key presses introduces additional limitations such as the client application can not register combination key presses (“Shift” + “1”) simultaneously but will require a slight delay (~50ms) between those key presses. Various methods can be used to address this issue.

#### 2.4 Communications Interfaces

**Remote Server**

As the client application ultimately forwards those key presses (virtual or otherwise) to a remote server, network latency will also be an important factor. The program’s action queue manager may need to add a short buffer between actions to ensure a previous action that arrived at the remote server slightly late, due to network latency, will not block the current action. This buffer should be configurable in the text(ini) config file.
