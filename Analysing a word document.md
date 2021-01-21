---
title: Analysing a word document
layout: home
author: seKret
permalink: docanalysis.html
---

In this post im going to analyse the 1st stage of the apt attack called carbanak, its a .doc document with malicious macros.
The sha1 of this sample is `9c79e0e85be8c271c61cfe27b1f106b671ec83c3`.
I will use [OfficeMalScanner](http://www.reconstructer.org/code/OfficeMalScanner.zip) for extract the macros from the document.
Lets do this.
The first thing we do is the following:

![](https://i.imgur.com/nhCuvgm.png)

The tool extracted 2 macros.
UserForm1:

![](https://i.imgur.com/uWCI90O.png)


ThisDocument:

![](https://i.imgur.com/A2tS9tj.png)

The most interesting one is `ThisDocument`.

The first thing we see its a function called 'armando', and the next are variable declarations, the variable names are ophuscated.

![](https://i.imgur.com/Wcuw3kf.png)

The following part we will analyse is the following:

![](https://i.imgur.com/6ALgal6.png)

The first thing the macro is doing its creating a object of "Scripting.FileSystemObject", it does this thing for using somes 'special' functions.

![](https://i.imgur.com/94bKIDJ.png)

The first function from "FileSystemObject" the script are using its 'GetSpecialFolder'.

![](https://i.imgur.com/P2EaO4U.png)

What this function do?

![](https://i.imgur.com/EloO1sM.png)

The function returns to you  a special system folder (like the temp folder).
The script its trying to get the temporary folder specifying as parameter the 2.
The next thing its doing its reverse a string a concatetaning that one to another string.

![](https://i.imgur.com/7bEdxlV.png)

The variable `xotso` has the path of the system temporary folder and its trying to concatenate the string "ini.daphsarc" but reversed, this string reversed its "crashpad.ini", its crating a string like this `C:\Users\user\AppData\Local\Temp\crashpad.ini` and storing this in a variable.
The next thing its doing its retrieving something in the document and storing it in a variable.

![](https://i.imgur.com/91TKjqz.png)

Once the 'strange thing' its stored in the variable the next thing will do the malware its use the function "OpenTextFile", the writer of this macro gives as parameter of the functions the values "iqoda", 2 and true. iqoda its a variable name, this variable has the value `C:\Users\user\AppData\Local\Temp\crashpad.ini`.

![](https://i.imgur.com/blBsEBP.png)

The explaining of this function its the following:

![](https://i.imgur.com/TZT9lQ8.png)

It creates a file in the path you specified in the arguments and returns a file pointer to a variable, in this case the variable 'anzes' will have the file pointer. The '2' in the parameters specifies 'writing mode'.

The next thing the malware will do its a for each loop in the contents of the variable that has the strange text, the goal of this loop its delete every '|*|' in the text.

![](https://i.imgur.com/L1HePKh.png)

The text inside this variable is the following:

![](https://i.imgur.com/VrPWZeI.png)

I beautify a little bit the code.

Ok, now we know its writing in to 'crashpad.ini' this script.

The next instructions the malware will execute are: 

![](https://i.imgur.com/n17yP8o.png)

The first instruction is:

![](https://i.imgur.com/iKKhx7R.png)

Its storing in that variable the string 'd.exe /c wsc'.
The next instruction its:

![](https://i.imgur.com/VMiW2Y2.png)

Its storing in another variable the string 'ript.exe //b /e:jsc'

![](https://i.imgur.com/L9MpMqP.png)

The following 2 instructions are storing the string "ript " in one variable an "c" in another diferent variable.
The next instruction is:

![](https://i.imgur.com/VGKWIpl.png)

Its storing in that variable the string 'cmd.exe /c wscript.exe //b /e:jscript C:\Users\user\AppData\Local\Temp\crashpad.ini'.

And the last instructions of the function are:

![](https://i.imgur.com/xitWXdB.png)<

The explanation of the 'Shell' function its:

![](https://i.imgur.com/F0cocRz.png)

Its executing 'crashpad.ini' as 'invisible mode', no cmd will be spawned.
And the last instruction its a simple msgbox saying "Decryption error".

I fully analyse this document in this post, the only part i dont analyse its that strange text, but maybe in later posts.
Now we know how the macro works, the first its locating where is the tmp path of windows, the next open a file in writing mode in that location and writes in the file a text that its 'hide' inside the document. The next thing the macro will do its crafted a command and execute it, this command will execute the file that the malware created in the temp path.

This is how you can analyse word documents.
Regards, seKret.

- seKret