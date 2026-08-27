# Linux Needs A Universal Back Action

Android got "Back" right.

You can be four pages deep in Settings, reading something in Firefox, inside a chat history, whatever. then swipe from either edge and the current thing goes back. You don't stop to find where that app put its arrow(hmmm hmm ios). You don't have remember this specific apps shortcut. You just go back.

Linux has all the individual versions of this but no system version really. Firefox has history. File managers have previous locations. Qt and GTK apps have page stacks. Mouse buttons can send Back keys. Then the shell has its own gestures sitting above all of it with no shared way to say the user wants the current thing to go back.

I think there should be one (or two lmxp).

```text
app knows:
"where does Back go?"

compositor knows:
"the user invoked Back"

protocol connects them
```

The exact input is desktop managed, Mobile Linux could use the Android-style edge swipe, a desktop could use a mouse button, keyboard key, trackpad gesture, accessibility control, or some literal Back button in the shell, could be mapped however.

They should all reach the same action.

Let me define the action first because this gets stupid fast if we don't define Back.

## What Back Is

Back returns to the state that led to the current state.

In Firefox that comes from browser history.

```text
Google -> Wikipedia -> article -> current page
                                <- Back
```

In Settings it *probably comes from a page stack.

```text
Settings -> Network -> Wi-Fi -> My network
                              <- Back
```

A file manager already tracks the previous location, a chat app knows the conversation list is under the conversation you opened, and if a menu, dialog, drawer, or temporary page needs to disappear first, the app knows that too.

But The compositor has none of this information. 
we should keep history in the app.
But the compositor still owns the gesture, so there needs to be a crossing point.

## The solution

My first thought was a small Wayland protocol. Something like `xdg_navigation` or `ext_navigation`, naming bikeshed can come beat me later along with the wayland forums.

The app should tell the compositor whether its active window currently has a Back handler, Then compositor sends Back when the user invokes the shell's configured action.

```text
             APPLICATION
                  |
                  | can_go_back
                  v
        +---------------------+
        | navigation protocol |
        +---------------------+
                  ^
                  |
                  | back
                  |
             COMPOSITOR
```

 The compositor does not pop anything itself, it just sends the action and the app runs the same handler its own back button already uses.

Linux does have Back key symbols already kinda, GTK exposes `GDK_KEY_Back`, Qt has `Qt::Key_Back`, and mouse or keyboard hardware can send that input to the focused apps That part is useful and should feed the same handler.

But a key only says a key happened. It does not tell the shell whether Back is available right now, and it cannot represent a gesture beginning, moving, cancelling, or committing. You can fake `Alt+Left` too, but then it works in Firefox, probably works in a file manager, then lands in some editor where it moves a cursor or hits a random binding and now the "universal" solution doesn't work.

## The Toolkits Already Have The State

Qt Quick Controls has `StackView`. It exposes `depth` and `pop()`. Kirigami has page stacks and `goBack()`. Libadwaita has `AdwNavigationView`, which already connects navigation state to its own buttons, shortcuts, mouse Back input, touchscreen swipes, and touchpad gestures.

This is not new state.

The toolkit would need a window-level dispatcher because one window can have nested stacks, a dialog, a drawer, maybe a temporary layer over all of it. You cannot grab the first `StackView` you find and call `pop()` unless you enjoy debugging UI state at 3 AM.

```text
window Back dispatcher
        |
        +-> temporary layer
        +-> active dialog
        +-> inner page stack
        +-> outer page stack
```

Topmost active handler goes first. Then the next one.

For most apps, using normal toolkit navigation should be enough. The toolkit advertises Back and routes the event. Apps with custom UI register a handler themselves.

If every app developer has to manually synchronize some Wayland boolean whenever they push a page, this shit is dead on arrival. The toolkit integration is the feature.

Now the layer question.

## Does This Belong In Wayland

This was the first thing people pushed back on when I brought the idea up.

One response was basically "I like it, not sure this belongs in low-level Wayland." Another person said they were already doing something similar in Hyprland userspace. Someone else thought it needed a Wayland base and then UI-specific behavior on top.

Fair.

The navigation model is high-level because it lives in an app. The trigger can be low-level because a compositor owns edge gestures and system input policy. A GTK-only solution does not give a Qt desktop a universal action, and a Qt-only solution does nothing for Firefox. The Global Shortcuts portal is aimed at app actions that can fire while the app is unfocused, whilst Back belongs to whichever window is active and changes availability every time its UI moves.

So I keep landing at Wayland for the crossing point.

```text
app state -> toolkit -> Wayland -> compositor policy
```

I don't know if that exact placement survives implementation. Thats the part I actually want feedback on. Build one compositor path and one toolkit path, then if the location feels like ass we know what failed instead of making up fifteen theoretical versions of it on a mailing list (hhmmm hmm xdg).

And there are enough ugly cases to test.

## Where im strugglin a bit

Wayland does not have one generic focus. Keyboard, pointer, and touch focus can differ. Popups can grab input. Modal child windows exist. One process can own multiple windows and every window can have different navigation state.

So I think the navigation object belongs to a toplevel window, not the whole process. The toolkit handles menus, dialogs, temporary layers, and page stacks inside that scope. Im less sure about grabbed popups and whether the shell dismisses those before Back reaches the toplevel, so yah that needs code.

Rapid input is another problem.

Say the app has one page left and advertises `can_go_back = true`. The user invokes Back twice before the compositor sees the new state. A fire-and-forget event could pop once and then do something stupid at the root.

The first version may need one outstanding Back at a time.

```text
app                         compositor
 |                              |
 | can_go_back = true           |
 |----------------------------->|
 |                              |
 |<---------------- back(id) ---|
 |                              |
 | handled(id), now false       |
 |----------------------------->|
```

The app handles the event, reports the new state, then another Back can happen. Maybe there is a cleaner Wayland-shaped way to do it, I don't know yet, but the ordering has to exist somewhere.

Root behavior is separate. When an app cannot go back, a phone shell might return home. A desktop might show an overview, minimize the window, or do nothing. `can_go_back = false` cannot just mean close the app because ts don't work.

Also Back is not Up.

If I search Settings and open Wi-Fi details directly, Back returns to the search results because thats where I came from. Up can go to the Wi-Fi category because thats the structural parent.

```text
Back: Wi-Fi details -> search results
Up:   Wi-Fi details -> Wi-Fi
```

im here to argue about Back. Forward, Up, Home, and the rest can be someone elses problem.

Then after the one-shot action works, the fun version can happen.

## Predictive Back

Android now lets Back act like a gesture lifecycle. Start, progress, commit, cancel.

```text
back_begin(id)
back_progress(id, 0.20)
back_progress(id, 0.65)
back_commit(id)
```

The current page can follow your finger while the previous page appears under it. If you change your mind, slide back and cancel.

```text
back_begin(id)
back_progress(id, 0.40)
back_cancel(id)
```

I want that on Plasma Mobile and Phosh badly. It would be nice on desktop trackpads too.

But predictive Back adds gesture identity, ordering, cancellation, frame timing, client stalls, and animation synchronization. Trying to solve all of that in version one is how version one never exists.

So we should start smaller.

## The First Test

a universal way to interface with everything as i have detailed it,
If Wayland is the wrong spot,should become obvious. If the protocol is wrong, same thing. I mostly want to know whether people agree with the split itself.
