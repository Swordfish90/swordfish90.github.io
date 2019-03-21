---
title: JSON Profiles in Ubuntu Terminal App
layout: post
link: https://swordfishslabs.wordpress.com/2015/02/27/json-profiles-in-ubuntu-terminal-app/
date: 2015/02/27 15:30:37
mood: happy
---

If you were bothered by a missing command or key in the terminal-app keyboard we’ve finally got you covered. Since our last update (27-02-2015) it is possible to easily define custom json layouts.

In this post I’m going to walk you through the construction of a simple VIM profile.

For starters we create the custom layouts directory:

<code>mkdir ~/.config/com.ubuntu.terminal/Layouts/</code>

All the JSON files inside it will be parsed when the application starts, so let’s create a empty file with your favourite editor (I’m guessing VIM at this point 😉 ).

<code>vim ~/.config/com.ubuntu.terminal/Layouts/vim.json</code>

The root object contains some pretty basic stuff like the name, the short_name which is shown on the selector and the language (which will be used in the future).

```
{
    "name" : "Simple VIM",
    "short_name" :"VIM",
    "language" : "en_US",
 
    "buttons": []
}
```

All the actual controls will be stored in the “buttons” array. Every Button object is composed by a main_action (triggered by click) and other_actions (triggered by click and drag in a sub-menu). There are currently two types of action, a “key” action which simulates a keystroke (with optional modifiers) and “string” action which sends a whole string to the terminal. Each action will also have a text property which is shown to the user.

Let’s start by emulating a very useful vim control: Escape.
There won’t be any additional action, the type will be “key” and we arbitrarily decide to show “ESC” to the user. Our final button will be:

```
{
    "main_action" : {
        "type" : "key",
        "text" : "ESC",
        "key" : "Escape"
    }
}
```

The list of keys that can be used is taken from here: [https://doc.qt.io/qt-5/qt.html#Key-enum](https://doc.qt.io/qt-5/qt.html#Key-enum). Please remember only to use the name after the prefix “Key_”, so the name “Qt::Key_Escape” becomes just “Escape”. Modifiers can be specified with the “mod” attribute, and the valid values are: Control, Alt and Shift.

Now let’s move to close and save commands. In vim we generally type the strings “:q”, “:q!”, “:wq” to close the file dropping or storing the changes. Since the correlation is strong I decided to put them in the same semantic button with “:q” as the main action and the other two as secondary actions. This time we want to send a string instead of a single keystroke so type will be set to “string”.

```
{
    "main_action" : {
        "type" : "string",
        "text" : ":q",
        "string" : ":q\n"
    },
    "other_actions" : [
        {
            "type" : "string",
            "text" : ":wq",
            "string" : ":wq\n"
        },
        {
            "type" : "string",
            "text" : ":q!",
            "string" : ":q!\n"
        }
    ]
}
```

The complete sample VIM profile can be found here: [https://drive.google.com/file/d/0B2KuANIptmZhWlQzdnVGLV9CZ00/view?usp=sharing](https://drive.google.com/file/d/0B2KuANIptmZhWlQzdnVGLV9CZ00/view?usp=sharing).

That’s it. If your custom profile doesn’t load, you probably have a syntactical or semantical error in your json file; check the standard output to see what’s wrong.

We strongly encourage you to play with this feature and to share your achievement with us, we are eager to see new profiles or patches to the default ones.

Cheers!