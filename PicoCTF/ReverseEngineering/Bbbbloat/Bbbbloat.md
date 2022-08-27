# Bbbbloat

--------------------------------------------------------------------

**TOOLS USED**: Ghidra<br>
**TARGET BINARY**: Bbbbloat

--------------------------------------------------------------------

## Initial Run

Program asks for user input.

![Run](./imgs/run.png)

## Ghidra

Initial ghidra summary on file import<br>
![Initial Ghidra Summary](./imgs/initial_ghidra.png)

Since the program ask for user input we can go to "window -> defined strings" and find that string<br>
![Ghidra Defined Strings](./imgs/defined_strings.png)

Cross reference to another function<br>
Click on memory address<br>
![Ghidra Defined Strings](./imgs/defined_strings2.png)


In the decompile pane we can see the code (looks like a main function)<br>
![Ghidra Decompile](./imgs/decompile.png)

local_48 is the user input and it is checking if it is equal to 0x86187 (favorite number)<br>
0x86187 = 549255<br>

![Flag](./imgs/flag.png)

**FLAG:** picoCTF{cu7_7h3_bl047_44f74a60}
