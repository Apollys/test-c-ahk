# C-style Syntax for AHK

Idea: design a c-style syntax, to be parsed in Python, that will be translated to AHK with the full power of the AHK language.

Motivation: AHK syntax is god awful, but the language is really useful.

---

### Syntax Brainstorming

Here I will go through simple AHK example scripts from tutorials or other places online and try to decide how I would design the
C-stlye syntax for them.

**Example 1: Simple Send on Hotkey**

```ahk
^+a::
    Send Hello World
    return
```

```c
void AHotkeyFunc() {
    Send("Hello World");
}

RegisterHotkey("^+a", AHotkeyFunc)
```

or maybe a more compact syntax?

```c
hotkey ("^+a") {  // space here to mirror if/while-style syntax, not a function
    Send("Hello World");
}
```

---

AHK also has these things called "hotstring"s apparently ([documentation](https://www.autohotkey.com/docs/Tutorial.htm#s2)).

What is a hotstring? Hotstrings are mainly used to expand abbreviations as you type them (auto-replace), they can also be used to launch any scripted action. For example:

`::ftw::Free the whales`

The difference between the two examples is that the hotkey will be triggered when you press Ctrl+J while the hotstring will convert your typed "ftw" into "Free the whales".

We could implement this as

```
RegisterHotstring("ftw", "Free the whales")
```

or using the second more compact-style syntax:

```
hotstring ("ftw", "Free the whales")
```

Hotstrings can also do multiple actions, and hotkeys can be one-liners, so really they can be treated exactly the same.
Perhaps for simplicity we could always use the "multi-line" syntax, but I'm not sure that's necessary.  For the current example, that would look like:

```
hotstring("ftw") {
    Send("Free the whales");
}
```

---

Any AutoHotkey function would be available with exactly the same name in the c-style syntax, for example:

```ahk
^j::
    MsgBox, Wow!
    MsgBox, There are
    Run, notepad.exe
    WinActivate, Untitled - Notepad
    WinWaitActive, Untitled - Notepad
    Send, 7 lines{!}{Enter}
    SendInput, inside the CTRL{+}J hotkey.
    return
```

is written as:

```c
hotkey ("^j") {
    MsgBox("Wow!");
    MsgBox("There are");
    Run("notepad.exe");
    WinActivate("Untitled - Notepad");
    WinWaitActive("Untitled - Notepad");
    Send("7 lines{!}{Enter}");
    SendInput("inside the CTRL{+}J hotkey.");
};  // <-- maybe we put a semicolon here?
```

---

All the special hotkey symbols will be used exactly the same as in AHK, for example:

```ahk
Numpad0 & Numpad1::
    MsgBox, You pressed Numpad1 while holding down Numpad0.
    return
```

is written as

```c
hotkey ("Numpad0 & Numpad1") {
    MsgBox("You pressed Numpad1 while holding down Numpad0.)
}
```

---

Directives:

```ahk
#IfWinActive Untitled - Notepad
#Space::
    MsgBox, You pressed WIN+SPACE in Notepad.
    return
```

(Note: `#Space` just means Windows Key + Space)

This can work the same way in the C syntax, at least for now, we will just use quotes for strings:

```c
#IfWinActive "Untitled - Notepad"
hotkey ("#Space") {
    MsgBox("You pressed WIN+SPACE in Notepad.");
}
```

---

Mixing `%`s and literal strings:

```ahk
; Run a program. Note that most programs will require a FULL file path:
Run, %A_ProgramFiles%\Some_Program\Program.exe
```

```c
// Run a program. Note that most programs will require a FULL file path:
Run(A_ProgramFiles + "\Some_Program\Program.exe");
```

Note: commands use "legacy syntax" (require `%`s around variables) while functions use normal expression syntax.
We will have to make sure to be clear about handling the context of these two.

---

Commands return values in output parameters, they don't have directly accessible return values.  For the c-style syntax, should we
have them actually return values, or the same syntax as in AHK?  I think it depends whether or not there can be multiple output parameters.

Let's assume each command only has one output parameter for now.  Then that output parameter is removed, and the value is returned instead:

```ahk
StringReplace, output, input, rianbow, rainbow, All
```

becomes

```c
output = StringReplace(input, "rianbow", "rainbow", "All");
```

---

Code blocks

In AHK, we are required to have a newline before the start of a code block, e.g.:

```ahk
if (my_var = 5)
{
    MsgBox, my_var equals %my_var%
    ExitApp
}
```

In c-style syntax we would be allowed/encouraged to instead place the opening brace on the preceeding line.
Also, double equals for equality check of course:

```
if (my_var == 5) {
    MsgBox("my_var = " + my_var);
    ExitApp();
}
```

---
