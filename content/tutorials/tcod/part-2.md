---
title: "Part 2 - The generic Entity, the render functions, and the map"
date: 2019-06-15T06:37:21-07:00
draft: false
---

Now that we can move our little '@' symbol around, we need to give it
something to move around *in*. But before that, let's stop for a moment
and think about the player object itself.

Right now, we just represent the player with the '@' symbol, and its x
and y coordinates. Shouldn't we tie those things together in an object,
along with some other data and functions that pertain to it?

Let's create a generic class to represent not just the player, but just
about *everything* in our game world. Enemies, items, and whatever other
foreign entities we can dream of will be part of this class, which we'll
call `Entity`.

Create a new file, and call it `entity.py`. In that file, put the
following class:

```py3
class Entity:
    """
    A generic object to represent players, enemies, items, etc.
    """
    def __init__(self, x, y, char, color):
        self.x = x
        self.y = y
        self.char = char
        self.color = color

    def move(self, dx, dy):
        # Move the entity by a given amount
        self.x += dx
        self.y += dy
```

This is pretty self explanatory. The `Entity` class holds the x and y
coordinates, along with the character (the '@' symbol in the player's
case) and the color (white for the player by default). We also have a
method called `move`, which will allow the entity to be moved around by
a given x and y.

Let's put our fancy new class into action\! Modify the first part of
`engine.py` to look like this:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
import tcod
import tcod.event

+from entity import Entity
from input_handlers import handle_keys


def main():
    screen_width = 80
    screen_height = 50

-   player_x = int(screen_width / 2)
-   player_y = int(screen_height / 2)

+   player = Entity(int(screen_width / 2), int(screen_height / 2), '@', tcod.white)
+   npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', tcod.yellow)
+   entities = [npc, player]
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod
import tcod.event

<span class="new-text">from entity import Entity</span>
from input_handlers import handle_keys


def main():
    screen_width = 80
    screen_height = 50

    <span class="crossed-out-text">player_x = int(screen_width / 2)</span>
    <span class="crossed-out-text">player_y = int(screen_height / 2)</span>

    <span class="new-text">player = Entity(int(screen_width / 2), int(screen_height / 2), '@', tcod.white)
    npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', tcod.yellow)
    entities = [npc, player]</span>
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

We're importing the `Entity` class into `engine.py`, and using it to
initialize the player and a new NPC. We store these two in a list, that
will eventually hold all our entities on the map.

Also modify the part where we handle movement so that the Entity class
handles the actual movement.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
                    if movement:
                        dx, dy = movement
-                       player_x += dx
-                       player_x += dy
+                       player.move(dx, dy)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                    if movement:
                        dx, dy = movement
                        <span class="crossed-out-text">player_x += dx</span>
                        <span class="crossed-out-text">player_x += dy</span>
                        <span class="new-text">player.move(dx, dy)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Lastly, update the drawing functions to use the new player object:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-           root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)
+           root_console.print(x=player.x, y=player.y, string=player.char, fg=player.color)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            <span class="crossed-out-text">root_console.print(x=player_x, y=player_y, string='@', fg=tcod.white)</span>
            <span class="new-text">root_console.print(x=player.x, y=player.y, string=player.char, fg=player.color)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Now we need to alter the way that the entities are drawn to the screen.
If you run the code right now, only the player gets drawn. Let's write
some functions to draw not only the player, but any entity currently in
our entities list.

Create a new file called `render_functions.py`. This will hold our
functions for drawing and clearing from the screen. Put the following
code in that file.

```py3
def render_all(console, entities):
    # Draw all the entities in the list
    for entity in entities:
        console.print(x=entity.x, y=entity.y, string=entity.char, fg=entity.color)
```

There should be no surprises here: we're just doing the same drawing function from `main`,
except we're doing it for a list of entities, one at a time. We just need to call this
function in `main` now:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
        ...
        while True:
-           root_console.print(x=player.x, y=player.y, string=player.char, fg=player.color)
+           render_all(root_console, entities)

            # Show the console.
            tcod.console_flush()
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        while True:
            <span class="crossed-out-text">root_console.print(x=player.x, y=player.y, string=player.char, fg=player.color)</span>
            <span class="new-text">render_all(root_console, entities)</span>

            # Show the console.
            tcod.console_flush()
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Be sure to import the function at the top of `engine.py`:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
...
from entity import Entity
from input_handlers import handle_keys
+from render_functions import render_all
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity
from input_handlers import handle_keys
<span class="new-text">from render_functions import render_all</span>
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

If you run the project now, you should see your '@' symbol, along with a
yellow one representing our NPC. It doesn't do anything, yet, but now we
have a method for drawing more than one character to the screen.

It's time to shift gears a bit and get our map in place.

### Creating the game's map

Let's start by defining the width and the height of the map. We can do
so by putting them below the `screen_width` and `screen_height` variables:

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    screen_height = 50
+   map_width = 80
+   map_height = 45
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    screen_height = 50
    <span class="new-text">map_width = 80
    map_height = 45</span>
    ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

We'll create a class to hold the map, which we'll subclass from tcod's
`Map` class. Why? Because tcod has a number of useful features built in
using this class, but we'll want to do just a bit more with it. Therefore,
a subclass should suit our purposes nicely.

Create a new file called `game_map.py` and put the following class in it:

```py3
from tcod.map import Map


class GameMap(Map):
    def __init__(self, width, height):
        super().__init__(width, height, order='F')

        self.transparent[:] = True
        self.walkable[:] = True

        self.walkable[30:32, 22] = False
```


{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    map_height = 45

+   colors = {
+       'dark_wall': libtcod.Color(0, 0, 100),
+       'dark_ground': libtcod.Color(50, 50, 150)
+   }

    player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    map_height = 45

    <span class="new-text">colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150)
    }</span>

    player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)</pre>
{{</ original-tab >}}
{{</ codetab >}}


### Conclusion

<script src="/js/codetabs.js"></script>