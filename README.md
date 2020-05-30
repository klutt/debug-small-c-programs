# How to debug small C programs

It's likely that you came here from Stackoverflow or a similar site because you asked for debugging help.

This document is heavily inspired from the blog post "How to debug small programs" which is amazing, but this document only focuses on C. So if you want to learn how to debug your C programs, this is for you. However, most of it is applicable to other languages too.


## Types of error

If the code does not do what you want it to do, it's cause can be divided into these categories:

### Compile time errors (The program cannot run)
 - Syntax error - The code simply refuses to compile because you break the language rules.
 - Semantic error - Similar to, and could be considered a subset of syntax errors.
 - Linker error - The code compiles but does not link. Common reasons are forgotten includes or wrong path. Technically not during compilation. But it's a problem occuring during the build phase, and compiling is often used instead of building.

### Run time errors (The program can be compiled and run)
 - Logical error - The code compiles, links and runs, but yields wrong results. 

It can be worth noted that this classification is not the only one. Different people prefer different classifications, and the one most preffered differ from language to language. Java folks prefer to have runtime error as a completely separate thing than logical errors, even though most (all) runtime errors is due to a logical error.

The most important thing is to know if your bug is during compile time or run time.


## UB

Undefined behavior is a concept in C that can make debugging extremely hard. The phrase is used in the specification of the language. If you for instance divide by zero in Java, the Java standard says that an exception should be thrown, and if the exception is not catched, the program crashes. In C, the standard simply says that the behavior is undefined, which means that the compiler is allowed to do whatever it wants. Typical symptoms of undefined behavior is that the program works the way it should (yes, really), crashes instantly or related or unrelated variables gets overwritten. Furthermore, there's nothing that says that the program will run fine until it invokes undefined behavior. Nope, it can behave strange even before that. This often manifests itself by skipping print statements BEFORE the troublesome line. If a program invokes undefined behavior THE WHOLE PROGRAM is invalid. This leads to another very typical symptom of UB, and that is that the program behaves differently depending on optimization level or if you are debugging or not.

UB is VERY common. It's not just some strange thing that happens when you do strange things. It's the source of the majority of beginner bugs, and also pretty common in professional code too. In order to properly debug C, this concept is crucial to understand.


## Warnings

Warnings is the compilers way of telling you that you probably are doing something else than you want. Pay close attention to your warnings. First, always compile with at least -Wall and preferably -Wall -Wextra. Personally, I often use -Wall -Wextra -Werror -std=c11 -pedantic but the two first should catch most things. Not only gives the warnings good hints on what's wrong. They also gives you a searchable phrase. Yes, whatever warning you get, someone else have already asked a question with that warning. It's not guaranteed to help you, but very often it does, and if it doesn't you now have something to add to your question that makes it much better and easier to answer. And other people will find your question easier.

For a small program, like a typical school project, you should in general not accept a single warning from -Wall -Wextra. For larger code bases you may have to just deal with them, but for small projects no. So address ALL warnings you get. In many cases, that will solve your problem. If not, you solved a problem you didn't know you had.

However, make sure that you understand the warning and don't just do something that magically makes it go away. And I'm especially talking about casting. Shortly, casting is a way to say to the compiler "I know this is iffy, but trust me. I know what I'm doing so shut up with your warnings."


## RTFM (Read the f*****g manual)

The documentation for C functions is widely available, so there's no excuse for not reading about the functions you're using. Warnings will often tell you if you have given a function the wrong number of arguments or arguments of the wrong type. If that's the case rtfm and find out what arguments the function expects.


## Search

Most of your problems have been asked by others. Searching the web is a crucial debugging tool, and I solve most of my problems that way. And if you have acticated warnings, it will be many times easier to find something relevant. If you're asking something really basic, it can be a good idea to search for "c for loop tutorial" or something like that. It can actually be good to classify what resources you're searching for. Try them all in a loop. For example, they could be classified as:
- Documentation
- Tutorials
- Examples
- Questions on QA-sites

The three first are easy to search. Searching for questions can be a bit tricker, and you will probably need to rephrase your search query a few times to find what you want.

## Error check your functions

Most functions that can go wrong does have a way of telling you that. In most cases, it is the return value. Do however note that the return value indicating an error differ from function to function. Common error values are 0, -1 or an unspecified negative number. For function returning pointers, a NULL pointer is usually an indication that something has gone wrong. But functions are different. scanf returns the number of successful assignments, so if you invoke scanf with 3 variables to assign and scanf returns anything less than 3, then something has gone wrong. Typically, continuing as if the function was successful will lead to undefined behavior. So if your code does not work, check ALL return values. Also, some functions also sets errno on error in addition to the error code. That makes it possible to extract even more information on what went wrong.


## Indent your code

Well indented code is MUCH more readable and makes it WAY easier to spot mistakes. You would be surprised how many bugs that become obvious by this simple step. Also, if you're using an editor that indents your code automatically (which you should) it may sometimes autoindent in an unexpected way. Very often, this is a good sign that you have made a mistake. And even if you don't find the error this way, it's common courtesy to make your code readable when other people should read it, irregardless if it is your colleges, class mates or people helping you on a forum.

Exactly what indentation style you go with does not really matter. As long as you avoid GNU indentation (ugh, the horror) you're fine. And make sure indentation is at least four spaces deep.


## Create a MRE

A minimal reproducible example is a wonderful thing when you debug. The principle is extremely simple. Remove everything you can from the code while it still reproduces your problem. It can be tedious and take a lot of effort, but it makes it far easier to spot the problem. Which is quite understandable. It's easier to find a needle among twenty hay straws than it is to find it in a big haystack. As with indentation, this is also a courtesy. Why should other people need to read more than necessary when helping you? Even if you cannot solve the problem, you can at least remove those parts that you know does not cause the problem.

But make sure that the code is complete before you post it. Don't guess what parts are necessary. Make sure that you can copy paste the code in your post into an empty file and that that is enough to reproduce the problem.

Personally, most of the times I'm creating a MRE, I never need to post it. The process of creating an MRE more often than not makes me see the problem.

Binary removal can be really efficient for quickly creating a MRE, and the process is simple. It's not always a suitable process, but when it is, use it. Start deleting stuff from the end of the main function. If you're still reproducing the same error, remove half of what's left. If it does not, put it back and remove half of the half the you removed the first time. This method can also be used within functions.

In many cases binary removal is not feasible to do to the letter. If you're within a function that returns a value, you can of course not remove the return statement. Use your own judgement. This method is basically what HR-people are doing when hiring new staff. A quick filter that removes a lot of unneccesary things that more skilled people (the bosses hiring or people helping you) does not have to consider. 


## Print variables

Printing variables is really a great tool. Print all variables involved at crucial events, like before, during and after a loop. Inspect the values carefully. Are they what you expect them to be? Also print values after reading them to make sure they are what you think.


## Remove input

C is notorius for being difficult and clunky when working with input. So try to remove it. Instead of making the user input a value, comment out the input line and assign a value to the variable. Same with files. Instead of reading data from files, hardcode the data in the code. After this, one of two things can happen.

First, we have the option that you reproduce the error, and then you can remove all code related to input. That makes it possible to post a compilable file that other users can compile without the need for creating additional files needed by the program. It also makes it possible to run it without entering tons of input every time, so this is especially important if you have a program that takes large amounts of input.

Secondly, it may happen (more often than you think) that you cannot reproduce the error this way. Gongratulations. Now you know that the error is in the input code.

With both outcomes, you have now come closer to a MRE, and also closer to the solution.


## Assert

It fills basically the same role as printing the values, but more silently. If a function expects a positive argument, then add an assert for that in the beginning. Writing asserts for the preconditions to functions the first thing that happens is a good practice. There are three main benefits with this

- It helps you catch errors
- It forces you to really think about what inputs are valid
- It gives information to other readers about what the function is doing, or more importantly when you ask for help, what you think it is doing.


## Refactor the code

Depending on your level, this might be a bit tricky, but it is still extremely useful. The concept of refactoring is simple. It's about changing your code to be more readable without changing it's observable bahavior. You don't solve bugs while refactoring. But often you discover them in the process. Examples of refactoring:
- Renaming variables and functions to more descriptive names
- Extracting code blocks to functions
- Restructuring the code in various ways


## Testing

Writing tests are awesome. Not only does the process force you to really think about what a function does, because how can you otherwise create an input/output pair to test? It does also give you a very easy way to test your function over and over again when you apply changes. It's a cruical part of refactoring. Testing is a whole science on it's own and beyond the scope for this article to be covered properly, but you can search for "unit test" to find some resources.


## Paper and pen

Very underrated tool. Execute the code in your head and write values for variabels at different states. If using pointers and arrays, draw boxes and arrows.


## Use a debugger

A debugger is obviously a very good debugging tool, hence its name. I may write more about this later. At least for a beginner, the most useful task for a debugger is to find which line that is causing a segmentation fault. If you're having a segfault, fire up a debugger and find the line. A debugger is also great when creating a MRE, since you can step through the code line by line, and that can give you very good clues on what line you can safely remove.


## Post a question on a forum

This should be your last resort. Use all of the above (to the best of your abilities) before doing this. Not only will it solve a vast majority of your problems without even posting a question. If it does not, it will make your question far easier to answer and far easier for future programmers to find the question.

So spend a lot of time before resorting to this. Exactly how much time depends on you and the problem. Be short and consise, but include everything that is relevant. Double check everything. Unless you're asking about a compiler error that prevents you from compiling, what you have posted should compile.

And before posting a question on a forum, read the forum rules and make sure that you follow the rules, that the question is on topic there and that it meets the requirements.

First, find out what kind of bug you have. Does it happen during runtime or compiletime? If compiletime, include the compiler (or linker) error. If you have a runtime error, include warnings, a sample output for a sample input and the expected output.

Loop:
  Make sure the code is well indented. This should be a no brainer.

  Create an MRE with the tools above.

  Add asserts and printouts and remove input by hardcoding if possible

  Refactor a bit if you know how to do it

  Use pen and paper, debugger, your brain and a rubberduck to try to figure out what the problem is

  Search the web

Loop the above. The order is not very important. Go back and forth among the steps. Continue until you have solved the problem or until you really have no clue about how to continue.
  
By now, you should have a code snippet suitible for posting on a forum. Make sure that the code really is enough to demonstrate your problem. Copy paste it straight from your editor. Don't type while reading it. Many typos finds their way to the questions this way. But before posting it, translate all variables, functions and strings to English. That helps A LOT. After this, check once more that it compiles and reproduces the error. You could also add some comments explaining what you think the problem is.

As mentioned before, include any output from the compiler. Also include output, input if any, and the expected output. If it is a segmentation fault or similar, include which line it occurs on. You can use a debugger for this.

Then write the body. Explain the problem briefly but clearly. Links are accepted, but should be references. The question should be understood without links. Avoid images unless very necessary. Pictures of code is a big nono. Be very clear about what the problem is. "It does not work" is not a problem description. If the program does not work, it has to be something that the code did or didn't that made you understand that it does not work. We need to know the expected behavior of the code. And give precise examples. 

Try to avoid XY-problems. That basically means that you're assuming that X is a suitable way to do Y and ask about X without even mentioning Y. Often, this leads to people thinking the question is very strange, and it also often leads to suboptimal (with very big emphasis on sub) solutions for you.

If it is a school assignment, please state so, but most importantly, post the question as it is. Nothing is so annoying as giving a perfect solution by only getting "oh, but I forgot to mention that the solutions may not use <whatever>. If your question have certain requirements, state them. And of course, that goes for all questions and not just school assignments. It's very common that the asker have misinterpreted the question.

It can be a good thing to link to similar questions and explain why the answers there did not help and how you noticed that they don't.

Read the question many times from an outsiders perspective. Would you understand the question if you did not know what you know about the question? This is actually tricky, so be careful to not miss anything vital.

Create a title, and make it describe the problem well. Also, make sure that the title is not needed to understand the question. All the information the title contains should be found in the body.

Double check the language. It's ok to not know the language perfectly, but do your best. Use spell checks and care for grammar, formatting, paragraphing and such. Make it as easy to read as possible. Adding a "sorry for my bad english" is ok if you're not a native speaker, dyslectic or whatever, but don't use it as an excuse to not improve the language to the best of your abilities.

On many forums, you can use tags. Make sure you read about the tags so you know they are appropriate. Don't tag spam. If you're coding C, don't als tag C++, Java or Python unless the question also contain those other languages. I want to specifically think about not tagging both C and C++ unless you have a damn good reason. They are separate languages, and even if much C code is compilable as C++, you would never write proper C++ program that compiles as C. Not even for a simple Hello World program.
