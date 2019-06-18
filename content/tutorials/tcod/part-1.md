---
title: "Part 1 - Drawing the '@' symbol and moving it around"
date: 2019-06-15T06:30:00-07:00
draft: false
---

Welcome to part 1 of the **Roguelike Tutorial Revised**\! This series
will help you create your very first roguelike game, written in Python\!

This part assumes that you have checked [Part 0](/tutorials/tcod/part-0/)
and are already set up and ready to go. If not, be sure to check that page,
and make sure that you've got Python and TCOD installed, and a file called `engine.py`
created in the directory that you want to work in.

Assuming that you've done all that, let's get started.

### Hello, World!

Modify (or create, if you haven't already) the file `engine.py` to look like this:

{{< highlight py3 >}}
import tcod
import tcod.event


def main():
    screen_width = 80
    screen_height = 50

    # Set the font
    tcod.console_set_custom_font(
        "arial10x10.png",
        tcod.FONT_LAYOUT_TCOD | tcod.FONT_TYPE_GREYSCALE,
    )

    # Initialize the root console.
    with tcod.console_init_root(
        w=screen_width,
        h=screen_height,
        order="F",
        renderer=tcod.RENDERER_SDL2,
        vsync=True,
        title="Roguelike Tutorial"
    ) as root_console:
        # Print a basic test message
        root_console.print(x=0, y=0, string='Hello World!', fg=tcod.white)

        # Run the game.
        while True:
            # Show the console.
            tcod.console_flush()

            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
                if event.type == "QUIT":
                    raise SystemExit()


if __name__ == '__main__':
    main()
{{</ highlight >}}

Make sure that this basic program runs before going any further. It will
ensure that Python and TCOD are installed properly.

This may not be the most exciting program in the world, I admit. But
let's go over a few things here before proceeding.

```py3
import tcod
import tcod.event
```

These statements just import the `tcod` library, along with the `tcod.event`
part, since `tcod` alone does not import events. Events will be used to detect
things like keyboard and mouse input. You may want to use the `tcod` library
without using its events system, but for this tutorial, we'll be using the events
to capture all of our input.

```py3
def main():
    screen_width = 80
    screen_height = 50
```

The `main` function is where we'll run our game. You can technically call it
whatever you want, but calling this function `main` is considered good practice,
as it informs other people who might be reading your code what the entry point
into everything else is.

```py3
    # Set the font
    tcod.console_set_custom_font(
        "arial10x10.png",
        tcod.FONT_LAYOUT_TCOD | tcod.FONT_TYPE_GREYSCALE,
    )
```

TCOD needs a font file to run, so we'll use the `arial10x10.png` file downloaded
earlier in [Part 0](/tutorials/tcod/part-0/). Make sure that this file is located
in the same folder/directory as your `engine.py` file, or this will not work.

```py3
    # Initialize the root console.
    with tcod.console_init_root(
        w=screen_width,
        h=screen_height,
        order="F",
        renderer=tcod.RENDERER_SDL2,
        vsync=True,
        title="Roguelike Tutorial"
    ) as root_console:
```

This creates the `root_console`, which we'll be drawing to. The `w` and `h` are "width"
and "height", respectively. `title` can be set to whatever you want; it's the text that
will appear at the top of the console itself. The other values, `order`, `renderer`, and `vsync`,
are set to the recommended defaults according to [tcod's documentation](https://python-tcod.readthedocs.io/en/latest/index.html).

```py3
        # Print a basic test message
        root_console.print(x=0, y=0, string='Hello World!', fg=tcod.white)
```

We're printing the message "Hello, World!" to the top left of the console, with the
color being set to "white". Instead of passing `tcod.white` to the `fg` part, you could
set it to `(255, 255, 255)` as well (or any RGB color you want, really).

```py3
        # Run the game.
        while True:
```

This is our "game loop". Basically, we'll be stuck in this loop until the player decides to
"break" out of it (ending the game). This way, the game won't quit out prematurely.

```py3
            # Show the console.
            tcod.console_flush()
```

This command displays the console to the user.

```py3
            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
                if event.type == "QUIT":
                    raise SystemExit()
```

Here, we're grabbing any "events" (again, mouse movement, keyboard inputs, etc.) and handling them.
The only one we're doing anything with is the `QUIT` event at the moment (which is fired when you
click the red 'x' at the top right corner of the console), which, as you might expect,
exits the game.


{{< highlight py3 >}}
if __name__ == '__main__':
    main()
{{< /highlight >}}

So what does that do? Basically, we're saying that we're only going to
run the "main" function when we explicitly run the script, using `python
engine.py`. It's not super important that you understand this now, but
if you want a more detailed explanation, [this answer on Stack
Overflow](https://stackoverflow.com/a/419185) gives a pretty good
overview.

Confirm that the above program runs (if not, there's probably an issue
with your setup). Once that's done, we can move on to bigger and
better things.


### Let's get moving!
The first major step to creating any roguelike is getting
an '@' character on the screen and moving, so let's get started with
that.

We're going to need to keep track of the player's position on the screen,
so it would make sense to create two variables for that: one for the "x"
position, and one for the "y". Add these variables to the `main` function
like this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    screen_height = 50
+
+   player_x = int(screen_width / 2)
+   player_y = int(screen_height / 2)
+
    # Set the font
    tcod.console_set_custom_font(
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    screen_height = 50
    <span class="new-text">
    player_x = int(screen_width / 2)
    player_y = int(screen_height / 2)
    </span>
    # Set the font
    tcod.console_set_custom_font(
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

We're setting the player's initial "x" and "y" values to the center of the screen.
We have to cast those values to `int`s, because it wouldn't make sense for the player
to be halfway between one space and another (not in this context, anyway).

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    ) as root_console:
-       # Print a basic test message
-       root_console.print(x=0, y=0, string='Hello World!', fg=tcod.white)
+       root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)

        # Run the game.
        while True:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    ) as root_console:
        <span class="crossed-out-text"># Print a basic test message</span>
        <span class="crossed-out-text">root_console.print(x=0, y=0, string='Hello World!', fg=tcod.white)</span>
        <span class="new-text">root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)</span>

        # Run the game.
        while True:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Now instead of printing out "Hello World!", we're displaying the "@" symbol (by far the most
common way to represent the player character in roguelikes). It should appear right in the center
of the screen, since we're using the previously established `player_x` and `player_y` variables.

Perhaps you've already guessed how we're going to move the player around: by modifying the
`player_x` and `player_y` values. How might we accomplish that? Most roguelikes control movement
through the keyboard (though what keys are used can vary). We're already capturing keyboard input
in the `for event in tcod.event.wait():`, so by using the `event` variable, we can determine
what key was pressed and what to do with it.

Here's what the code to detect key presses and act upon them looks like:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
                if event.type == "QUIT":
                    raise SystemExit()

+               if event.type == 'KEYDOWN':
+                   key = event.sym

+                   if key == tcod.event.K_UP:
+                       player_y -= 1
+                   elif key == tcod.event.K_DOWN:
+                       player_y += 1
+                   elif key == tcod.event.K_LEFT:
+                       player_x -= 1
+                   elif key == tcod.event.K_RIGHT:
+                       player_x += 1
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
                if event.type == "QUIT":
                    raise SystemExit()

                <span class="new-text">if event.type == 'KEYDOWN':
                    key = event.sym

                    if key == tcod.event.K_UP:
                        player_y -= 1
                    elif key == tcod.event.K_DOWN:
                        player_y += 1
                    elif key == tcod.event.K_LEFT:
                        player_x -= 1
                    elif key == tcod.event.K_RIGHT:
                        player_x += 1</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Because we're dynamically changing where the player gets drawn to, we need to move
the functions that draws the player inside our loop, like this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    ) as root_console:
-       root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)
-
        # Run the game.
        while True:
+           root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)
+
            # Show the console.
            tcod.console_flush()
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>   ...
    ) as root_console:
        <span class="crossed-out-text">root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)</span>

        # Run the game.
        while True:
            <span class="new-text">root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)</span>

            # Show the console.
            tcod.console_flush()
            ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Now, the player will be redrawn on each loop, to wherever the current x and y positions happen to be.

Run `engine.py` now and see what happens. While you can now move the `@` symbol around, there's an
obvious problem: the symbol doesn't get removed from its previous position! Unless you're planning on
making "Snake: The Roguelike" (not saying you shouldn't!), this isn't desired behavior.

Here's how to fix it:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
            ...
            # Show the console.
            tcod.console_flush()
+            
+           # Clear the root console
+           root_console.clear()
+
            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            # Show the console.
            tcod.console_flush()
            
            <span class="new-text"># Clear the root console
            root_console.clear()</span>

            # Wait for an event, such as a mouse event or a key press
            for event in tcod.event.wait():
            ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

This will "clear" the root console, which will ensure that once our player has moved,
the previous image of the player in the previous position will be cleared. Try running
the program again, and you should get the expected behavior.

This is great, but there's room for improvement. Needless to say, the movement keys are
not going to be the only keyboard commands that we accept in our entire game. Far from it!
The problem is that if we add them all in the `main` function, it'll grow extremely long.

Up until now, this tutorial hasn't deviated all that much from the
original one, but here's a critical turning point. We're about to define
a function, called `handle_keys` to take care of keyboard input. We
*could* put this in our `engine.py` file... but should it be there? I
would argue no. The engine (game loop) captures input and should do
something with it, but translating from one to the other is not
something it needs to know about.

Let's create a new file, called `input_handlers.py` (you can call it whatever you like; just
be sure to remember to change the name in your program when you reference it). Put the following
in that file:

```py3
import tcod.event


def handle_keys(key):
    # Movement keys
    if key == tcod.event.K_UP:
        return {'move': (0, -1)}
    elif key == tcod.event.K_DOWN:
        return {'move': (0, 1)}
    elif key == tcod.event.K_LEFT:
        return {'move': (-1, 0)}
    elif key == tcod.event.K_RIGHT:
        return {'move': (1, 0)}

    return {}
```

This is very similar (nearly identical, really) to the code we were using in the `main` function,
but with a key difference: Instead of modifying `player_x` and `player_y` directly, we're opting to
return a dictionary instead. Why is that? Simply put: a *lot* of things can potentially happen
when you press a key. Instead of doing all those operations in this function, we'll just tell the
game the "result" of the key press. In this case, the movement keys result in a `"move"` action,
which is a tuple of an `x` and `y` coordinate changes. So if the player hits the up arrow key, we
expect the player wants to move 0 spaces in the x direction and -1 in the y direction.

So how do we utilize this function? First, we need to import it at the top of `engine.py`, like this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
import tcod
import tcod.event
+
+from input_handlers import handle_keys


def main():
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
import tcod
import tcod.event

<span class="new-text">from input_handlers import handle_keys</span>


def main():
    ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Then, we can replace our previous movement section with this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
                if event.type == 'KEYDOWN':
                    key = event.sym

-                   if key == tcod.event.K_UP:
-                       player_y -= 1
-                   elif key == tcod.event.K_DOWN:
-                       player_y += 1
-                   elif key == tcod.event.K_LEFT:
-                       player_x -= 1
-                   elif key == tcod.event.K_RIGHT:
-                       player_x += 1
+
+                   action = handle_keys(key)
+
+                   movement = action.get('move')
+
+                   if movement:
+                       dx, dy = movement
+                       player_x += dx
+                       player_y += dy
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                if event.type == 'KEYDOWN':
                    key = event.sym

                    <span class="crossed-out-text">if key == tcod.event.K_UP:</span>
                        <span class="crossed-out-text">player_y -= 1</span>
                    <span class="crossed-out-text">elif key == tcod.event.K_DOWN:</span>
                        <span class="crossed-out-text">player_y += 1</span>
                    <span class="crossed-out-text">elif key == tcod.event.K_LEFT:</span>
                        <span class="crossed-out-text">player_x -= 1</span>
                    <span class="crossed-out-text">elif key == tcod.event.K_RIGHT:</span>
                        <span class="crossed-out-text">player_x += 1</span>

                    <span class="new-text">action = handle_keys(key)

                    movement = action.get('move')

                    if movement:
                        dx, dy = movement
                        player_x += dx
                        player_y += dy</span>
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Now, we're simply getting whatever "action" resulted from the keypress (if any) and acting
upon it. In the case of moving, we're getting the "move" action (again, if it's relevant)
and assuming that the "move" key corresponds to an x and y tuple. We then apply those values
to the player's x and y positions.

Let's say we want to add diagonal movement. We can do so by expanding our `handle_keys` function,
like this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
def handle_keys(key):
    # Movement keys
-   if key == tcod.event.K_UP:
+   if key in (tcod.event.K_UP, tcod.event.K_k, tcod.event.K_KP_8):
        return {'move': (0, -1)}
-   elif key == tcod.event.K_DOWN:
+   elif key in (tcod.event.K_DOWN, tcod.event.K_j, tcod.event.K_KP_2):
        return {'move': (0, 1)}
-   elif key == tcod.event.K_LEFT:
+   elif key in (tcod.event.K_LEFT, tcod.event.K_h, tcod.event.K_KP_4):
        return {'move': (-1, 0)}
-   elif key == tcod.event.K_RIGHT:
+   elif key in (tcod.event.K_RIGHT, tcod.event.K_l, tcod.event.K_KP_6):
        return {'move': (1, 0)}
+   elif key in (tcod.event.K_y, tcod.event.K_KP_7):
+       return {'move': (-1, -1)}
+   elif key in (tcod.event.K_u, tcod.event.K_KP_9):
+       return {'move': (1, -1)}
+   elif key in (tcod.event.K_b, tcod.event.K_KP_1):
+       return {'move': (-1, 1)}
+   elif key in (tcod.event.K_n, tcod.event.K_KP_3):
+       return {'move': (1, 1)}

    return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key):
    # Movement keys
    <span class="crossed-out-text">if key == tcod.event.K_UP:</span>
    <span class="new-text">if key in (tcod.event.K_UP, tcod.event.K_k, tcod.event.K_KP_8):</span>
        return {'move': (0, -1)}
    <span class="crossed-out-text">elif key == tcod.event.K_DOWN:</span>
    <span class="new-text">elif key in (tcod.event.K_DOWN, tcod.event.K_j, tcod.event.K_KP_2):</span>
        return {'move': (0, 1)}
    <span class="crossed-out-text">elif key == tcod.event.K_LEFT:</span>
    <span class="new-text">elif key in (tcod.event.K_LEFT, tcod.event.K_h, tcod.event.K_KP_4):</span>
        return {'move': (-1, 0)}
    <span class="crossed-out-text">elif key == tcod.event.K_RIGHT:</span>
    <span class="new-text">elif key in (tcod.event.K_RIGHT, tcod.event.K_l, tcod.event.K_KP_6):</span>
        return {'move': (1, 0)}
    <span class="new-text">elif key in (tcod.event.K_y, tcod.event.K_KP_7):
        return {'move': (-1, -1)}
    elif key in (tcod.event.K_u, tcod.event.K_KP_9):
        return {'move': (1, -1)}
    elif key in (tcod.event.K_b, tcod.event.K_KP_1):
        return {'move': (-1, 1)}
    elif key in (tcod.event.K_n, tcod.event.K_KP_3):
        return {'move': (1, 1)}</span>

    return {}</pre>
{{</ original-tab >}}
{{</ codetab >}}

Why did we change the cardinal direction keys? Because we're now allowing the player to use
multiple types of inputs to move around. The most common ways to allow players to move in
diagonals are VIM keys and the numpad. Both options are represented here. You should now
be able to move around in all 8 directions, using any combination of VIM keys and numpad keys
that you want.

Now that we're moving around in 8 directions, we could call it a day here, but let's add two
last little pieces of control: setting the game to full screen, and exiting using the `esc` key.

### Full screen and exit keys

We'll need to capture keyboard inputs for both exiting the game and setting it to full screen.
We can accomplish this by adding the code below to `handle_keys`:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-def handle_keys(key):
+def handle_keys(key, modifier=None):
    # Movement keys
    if key in (tcod.event.K_UP, tcod.event.K_k, tcod.event.K_KP_8):
        return {'move': (0, -1)}
    elif key in (tcod.event.K_DOWN, tcod.event.K_j, tcod.event.K_KP_2):
        return {'move': (0, 1)}
    elif key in (tcod.event.K_LEFT, tcod.event.K_h, tcod.event.K_KP_4):
        return {'move': (-1, 0)}
    elif key in (tcod.event.K_RIGHT, tcod.event.K_l, tcod.event.K_KP_6):
        return {'move': (1, 0)}
    elif key in (tcod.event.K_y, tcod.event.K_KP_7):
        return {'move': (-1, -1)}
    elif key in (tcod.event.K_u, tcod.event.K_KP_9):
        return {'move': (1, -1)}
    elif key in (tcod.event.K_b, tcod.event.K_KP_1):
        return {'move': (-1, 1)}
    elif key in (tcod.event.K_n, tcod.event.K_KP_3):
        return {'move': (1, 1)}

+   if key == tcod.event.K_ESCAPE:
+       return {'escape': True}
+   elif key == tcod.event.K_RETURN and modifier & tcod.event.KMOD_LALT:
+       return {'fullscreen': True}

    # No key was pressed
    return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def handle_keys(key):</span>
<span class="new-text">def handle_keys(key, modifier=None):</span>
    # Movement keys
    if key in (tcod.event.K_UP, tcod.event.K_k, tcod.event.K_KP_8):
        return {'move': (0, -1)}
    elif key in (tcod.event.K_DOWN, tcod.event.K_j, tcod.event.K_KP_2):
        return {'move': (0, 1)}
    elif key in (tcod.event.K_LEFT, tcod.event.K_h, tcod.event.K_KP_4):
        return {'move': (-1, 0)}
    elif key in (tcod.event.K_RIGHT, tcod.event.K_l, tcod.event.K_KP_6):
        return {'move': (1, 0)}
    elif key in (tcod.event.K_y, tcod.event.K_KP_7):
        return {'move': (-1, -1)}
    elif key in (tcod.event.K_u, tcod.event.K_KP_9):
        return {'move': (1, -1)}
    elif key in (tcod.event.K_b, tcod.event.K_KP_1):
        return {'move': (-1, 1)}
    elif key in (tcod.event.K_n, tcod.event.K_KP_3):
        return {'move': (1, 1)}

    <span class="new-text">if key == tcod.event.K_ESCAPE:
        return {'escape': True}
    elif key == tcod.event.K_RETURN and modifier & tcod.event.KMOD_LALT:
        return {'fullscreen': True}</span>

    # No key was pressed
    return {}</pre>
{{</ original-tab >}}
{{</ codetab >}}

What's with the `modifier`? It's used to capture things like if the user is
holding down the `alt` or `ctrl` keys. In this case, if the user holds the
left `alt` key and hits `enter` (return), then we return the 'fullscreen'
action.

Of course, we'll need to capture that in the `main` function, along with
handling the 'fullscreen' and 'escape' actions that we're now returning.
Modify `main` in the following manner:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
                    key = event.sym
+                   modifier = event.mod

-                   action = handle_keys(key)
+                   action = handle_keys(key, modifier)

+                   escape = action.get('escape')
+                   full_screen = action.get('fullscreen')
                    movement = action.get('move')

+                   if escape:
+                       raise SystemExit()
+
+                   if full_screen:
+                       tcod.console_set_fullscreen(not tcod.console_is_fullscreen())

                    if movement:
                        dx, dy = movement
                        player_x += dx
                        player_y += dy
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                    key = event.sym
                    <span class="new-text">modifier = event.mod</span>

                    <span class="crossed-out-text">action = handle_keys(key)</span>
                    <span class="new-text">action = handle_keys(key, modifier)</span>

                    <span class="new-text">escape = action.get('escape')
                    full_screen = action.get('fullscreen')</span>
                    movement = action.get('move')

                    <span class="new-text">if escape:
                        raise SystemExit()

                    if full_screen:
                        tcod.console_set_fullscreen(not tcod.console_is_fullscreen())</span>

                    if movement:
                        dx, dy = movement
                        player_x += dx
                        player_y += dy</pre>
{{</ original-tab >}}
{{</ codetab >}}

The `if escape` block essentially does the same thing as the `'QUIT'` part earlier. The
`if full_screen` sets the console's full screen property to the opposite of whatever it
currently is: If it's full screen, it sets it to not full screen, and vice versa.

### Conclusion

That's all for part 1. In this section, we got a basic `@` symbol moving around the screen,
and... well, not much else, honestly. But this is an important first step to making a functioning
roguelike game; we're well on our way!

[Click here to move on to the next part of this
tutorial.](/tutorials/tcod/part-2/)

<script src="/js/codetabs.js"></script>