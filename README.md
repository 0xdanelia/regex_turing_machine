I created a functioning Turing Machine using Notepad++ and its Find/Replace regular expression engine. Take a look at `busybeaver.txt` in Notepad++ or (almost) any other text editor with a regex find/replace option.

<img src="https://i.imgur.com/dO6JUGn.gif">

## What it looks like

The machine consists of the tape, the read/write head, the current instruction, and the instruction sets.

- The tape is a line beginning with a `!` followed by a string of `0`s and `1`s.
- The read/write head is a line beginning with a `[` followed by a preset amount of whitespace and ending with a `^`.
- The current instruction is a line beginning with a `#` and followed by the name of the current instruction.
- The instruction sets are the remaining lines, formatted like `>C.I:WMN` 
  - `C` current instruction name.
  - `I` the input from the current tape position. Either `0` or `1`. Each instruction has execution parameters for both inputs.
  - `W` the output to be written to the tape at the current position. Either `0` or `1`.
  - `M` the movement of the read/write head. A `0` moves the head one position to the left. A `1` moves it to the right.
  - `N` the name of the next instruction to be executed at the new tape position.

---

**Note**: For the machine to work properly, make sure to add at least one newline after your instruction set.

---

This is an example of a tape with one instruction set named `INST`

```
!0000000000000000000001000000000000000000000000
[                     ^
#INST
>INST.0:00HALT
>INST.1:01INST
```

This machine is currently on the instruction `INST` with the current tape position containing a `1`.

`INST.1` defines the following instructions: write a `0`, move the head to the right, go to instruction `INST`.

After overwriting the tape with a `0` and moving the head to the right, the new tape position will contain a `0`, and we will still be in instruction `INST`.

`INST.0` defines the following instructions: write a `0`, move the head to the left, go to instruction `HALT`.

Because the tape already contains a `0` at that point, the content of the tape does not change. After moving the head to the left, we go to instruction `HALT`. Because no instruction set is defined with this name, the machine will stop running.


## How to run it

To run the machine, plug the following into the find/replace box of Notepad++:

Find what:
```
^!0([01]{20})([01])([01]*\r?\n\[ +\^\r?\n#)(.*)\r?\n(?=(.*\r?\n)*^>\4\.\2:([01])((0)|1)(.*\r?\n))
```
Replace with:
```
!\8\8\1\6\3\9
```


Make sure the `Regular expression` option and the `Wrap around` box are selected.

Make sure the `. matches newline` box is unselected.

Then keep clicking the `Replace` button.

The machine halts once the regular expression no longer finds a match and the replace stops functioning.

TIP: to save yourself some clicking, you can click `Replace` once and then hold `Enter` on your keyboard to execute many instructions quickly.


## How it works

The regular expression matches on the tape, the read/write head, and the current instruction. All of these things are replaced with the contents of the find/replace box with each instruction iteration.
```
^!0([01]{20})([01])([01]*\r?\n\[ +\^\r?\n#)(.*)\r?\n
```
The regular expression then uses a lookahead to read information about the current instruction.
```
(?=(.*\r?\n)*^>\4\.\2:([01])((0)|1)(.*\r?\n))
```
Everything contained within parentheses in the regular expression is called a group. The fourth group of parentheses can be referenced later in the expression with `\4`. In this case, `\4` is the name of the current instruction set and `\2` is the value of the current tape position. The lookahead passes by every line of text until it finds the one that begins with `>\4\.\2:` which it can then parse for the necessary information to execute the current instruction.

The groups are defined as follows:
- `\1` tape from position 2 up to current position
- `\2` tape at current position
- `\3` rest of tape, line break, all of the read/write head line, line break, `#`
- `\4` name of current instruction
- `\5` all text up until the current instruction
- `\6` what to write at current position
- `\7` where to move (`0` left, `1` right)
- `\8` what to modify beginning of tape with in order to move the head (explained below)
- `\9` next instruction name

These groups are used to piece together the `replace` portion of the find/replace. Most of the information remains the same, but some key elements are modified based on what the lookahead finds inside the current instruction.
```
!\8\8\1\6\3\9
```
To move the head left or right, either a `0` is inserted at the beginning of the tape, or a `0` is deleted from the beginning of the tape.
This scrolls the tape left or right while the read/write head remains stationary. With the current implementation, the read/write head looks at the 22nd element of the tape whenever we want to read the current position. With complicated machines that utilize long lengths of the tape, you would need to increase this number so that you never delete a necessary tape element as you scroll along it. This can be done by increasing the `{20}` that appears at the beginning of the expression. To make everything display correctly, be sure to also add more length to your tape and move the `^` representing the read/write head on the next line.

The actual scrolling of the tape is done using this portion of the lookahead:
```
((0)|1)
```
This is essentially saying match on (`0` or `1`) at the position in the instruction representing movement. But with the way grouping works in combination with the `|` operator, this will result in the following:
- If the movement instruction is a `0`, then `\8` will contain a `0`.
- If the movement instruction is a `1`, then `\8` will contain the empty string, because there is no 8th group on the right side of the `|`.

We utilize this by taking the first element of the tape (which must be a `0`) and replacing it with `\8\8`. If we consider the following tape:
```
!0001000
[   ^
```
A movement instruction of `0` (left) will replace the first `0` on the tape with `00`:
```
!00001000
[   ^
```
A movement instruction of `1` (right) will replace the first `0` on the tape with two empty strings:
```
!001000
[   ^
```
This replacement of two `0`s or two empty strings results in the ribbon scrolling left or right above the read/write head.
