---
title: "Getting Started"
subtitle: <h4 style="font-style:normal">GEO 200CN - Quantitative Geography</h4>
author: <h4 style="font-style:normal">Professor Noli Brazil</h4>
output: 
  html_document:
    toc: true
    toc_depth: 3
    toc_float: true
    theme: yeti
    code_folding: show
---

<style>
h1.title {
  font-weight: bold;
  font-family: Arial;  
}

h2.title {
  font-family: Arial;  
}

.figure {
   margin-top: 20px;
   margin-bottom: 20px;
}

</style>

<style type="text/css">
#TOC {
  font-size: 13px;
  font-family: Arial;
}
</style>

\

The purpose of this lab guide is to install and set up R and RStudio. You will be using R to complete all data analysis tasks in this class.  For those who have never used R or it has been a long time since you've used the program, welcome to what will be a life fulfilling journey! 


<div style="margin-bottom:25px;">
</div>
##**What is R?**
\

R is an environment for data analysis and graphics. R is an interpreted language, not a compiled one. This means that you type something into R and it does what you tell it.  It is both a command line software and a programming environment.  It is an extensible, open-source language and computing environment for Windows, Macintosh, UNIX, and Linux platforms, which  allows for the user to freely distribute, study, change, and improve the software. It is basically a free, super big, and complex calculator. You will be using R to accomplish all data analysis tasks in this class. You might be wondering "Why in the world do we need to know how to use a statistical software program?" Here are the main reasons:

<br>

1. We will be going through a lot of equations and abstract concepts in lecture and readings.  Applying these concepts using real data is an important form of learning.  A statistical software program is the most efficient (and in many cases the only) means of running data analyses, not just in the cloistered setting of a university classroom, but especially in the real world.  Applied data analysis will be the way we bridge statistical theory to the "real world." And R is the vehicle for accomplishing this.

2. In order to do applied data analysis outside of the classroom, you need to know how to use a statistical program. There is no way around it as we don't live in an exclusively pen and paper world.  If you want to collect data on soil health, you need a program to store and analyze that data. If you want to collect data on the characteristics of recent migrants, you need a program to store and analyze that data. Even in qualitative work, after you've transcribed your interviews, the information you collected are data, which you will then need a software program to store and analyze that data.

<br>

The next question you may have is "I love Excel [or insert your favorite program]. Why can't I use that and forget your stupid R?" Here are some reasons

<br>

1. it is free. Most programs are not; 
2. it is open source. Which means the software is community supported.  This allows you to get help not from some big corporation (e.g.  Microsoft with Excel), but people all around the world who are using R.  And R has **a lot** of users, which means that if you have a problem, and you pose it to the user community, someone will help you;
3. it is powerful and extensible (meaning that procedures for analyzing data that don’t currently exist can be readily developed);
4. it has the capability for mapping data, an asset not generally available in other statistical software;
5. If it isn't already there, R is becoming the de-facto data analysis tool in the physical and social sciences

<br>

R is different from Excel in that it is generally not a point-and-click program.  You will be primarily writing code to clean and analyze data.  What does *writing* or *sourcing* code mean? A basic example will help clarify.  Let's say you are given a dataset with 10 rows representing people living in Davis, CA. You have a variable in the dataset representing individual income.  Let's say this variable is named *inc*.  To get the mean income of the 10 people in your dataset, you would write code that would look something like this


```r
mean(inc)
```

The command tells the program to get the mean of the variable *inc*.  If you wanted the sum, you write the command `sum(inc)`. 

Now, where do you write this command? You write it in a script.  A script is basically a text file.  Think of writing code as something similar to writing an essay in a word document. Instead of sentences to produce an essay, in  a programming script you are writing code to run a data analysis.  The basic process of sourcing code to run a data analysis task is as follows. 

1. **Write code**. First, you open your script file, and write code or various commands (like `mean(inc)`) that will execute data analysis tasks in this file.
2. **Send code to the software program**. You then send your some or all of your commands to the statistical software program (R in our case).
3. **Program produces results based on code**. The program then reads in your commands from the file and executes them, spitting out results in its console screen. 
 
I am skipping over many details, most of which are dependent on the type of statistical software program you are using, but the above steps outline the general work flow. You might now be thinking that you're perfectly happy pointing and clicking your mouse in Excel (or wherever) to do your data analysis tasks.  So, why should you adopt the statistical programming approach to conducting a data analysis? 

<br>

1. Your script  documents the decisions you made during the data analysis process. This is beneficial for many reasons.
* It allows you to recreate your steps if you need to rerun your analysis many weeks, months or even years in the future. 
* It allows you to share your steps with other people.  If someone asks you what were the decisions made in the data analysis process, just hand them the script.
* Related to the above points, a script promotes transparency (here is what i did) and reproducibility (you can do it too). When you write code, you are forced to explicitly state the steps you took to do your research.   When you  do research by clicking through drop-down menus, your steps are lost, or at least documenting them requires considerable extra effort. 
2. If you make a mistake in a data analysis step, you can go back, change a few lines of code, and poof, you've fixed your problem.
3. It is more efficient. In particular, cleaning data can encompass a lot of tedious work that can be streamlined using statistical programming.

<br>

Hopefully, I've convinced you that statistical programming and R are worthwhile to learn.

<div style="margin-bottom:25px;">
</div>
##**Getting R**
\

R can be downloaded from one of the “CRAN” (Comprehensive R Archive Network) sites. In the US, the main site is at http://cran.us.r-project.org/.  Look in the “Download and Install R” area. Click on the appropriate link based on your operating system.  

**If you already have R on your computer, make sure you have most updated version of R on your personal computer (3.6.3 "Holding the Windsock").**


<div style="margin-bottom:25px;">
</div>
###**Mac OS X**


1. On the “R for Mac OS X” page, there are multiple packages that could be downloaded. If you are running Mavericks, Yosemite, El Capitan, Sierra, High Sierra, Mojave or Catalina, click on R-3.6.3.pkg; if you are running an earlier version of OS X, download the R-3.2.1-snowleopard.pkg. 

2. After the package finishes downloading, locate the installer on your hard drive, double-click on the installer package, and after a few screens, select a destination for the installation of the R framework (the program) and the R.app GUI. Note that you will have to supply the Administrator’s password. Close the window when the installation is done.

3. An application will appear in the Applications folder: R.app. 

<div style="margin-bottom:25px;">
</div>
###**Windows**


1. On the “R for Windows” page, click on the “base” link, which should take you to the “R-3.6.3 for Windows (32/64 bit)” page

2. On this page, click either on the “base” or “install R for the first time links”.

3. On the next page, click on the “Download R 3.6.3 for Windows” link, and save the exe file to your hard disk when prompted. Saving to the desktop is fine.

4. To begin the installation, double-click on the downloaded file. Don’t be alarmed if you get unknown publisher type warnings. Window’s User Account Control will also worry about an unidentified program wanting access to your computer. Click on “Run”.

5. Select the proposed options in each part of the install dialog. When the “Select Components” screen appears, just accept the standard choices

Note: Depending on the age of your computer and version of Windows, you may be running either a “32-bit” or “64-bit” version of the Windows operating system. If you have the 64-bit version (most likely), R will install the appropriate version (R x64 3.5.2) and will also (for backwards compatibility) install the 32-bit version (R i386 3.5.2). You can run either, but you will probably just want to run the 64-bit version.


<div style="margin-bottom:25px;">
</div>
##**What is RStudio?**
\

If you click on the R program you just downloaded, you will find a very basic user interface. For example, below is what I get on a Mac

<br>

<center>
![R's Interface.](/Users/noli/Documents/UCD/teaching/GEO 200CN/lab/geo200cn.github.io/rint.png)
</center>


We will not use R's direct interface to run analyses. Instead, we will use the program RStudio. RStudio gives you a true integrated development environment (IDE), where you can write code in a window, see results in other windows, see locations of files, see objects you've created, and so on. To clarify which is which: R is the name of the programming language itself and RStudio is a convenient interface.

To download and install RStudio, follow the directions below

1. Navigate to RStudio's download [site](https://rstudio.com/products/rstudio/download/#download)
2. Click on the appropriate link based on your OS (Windows, Mac, Linux and many others). Do not download anything from the "Zip/Tarballs" section.
3. Click on the installer that you downloaded.  Follow the installation wizard's directions, making sure to keep all defaults intact.  After installation, RStudio should pop up in your Applications or Programs folder/menu.


<div style="margin-bottom:25px;">
</div>
##**The RStudio Interface**
\

Open up RStudio.  You should see the interface shown in the figure below, which has three windows.

<br>

<center>
![R Studio's Interface.](/Users/noli/Documents/UCD/teaching/GEO 200CN/lab/geo200cn.github.io/rstudio.png)
</center>

<br>

* **Console** (left) - The way R works is you write a line of code to execute some kind of task on a data object.  The R Console allows you to run code interactively. The screen prompt `>` is an invitation from R to enter its world. This is where you type code in, press enter to execute the code, and see the results.
* **Environment, History, and Connections tabs** (upper-right)
    + **Environment** - shows all the R objects that are currently open in your workspace.  This is the place, for example, where you will see any data you've loaded into R. When you exit RStudio, R will clear all objects in this window.  You can also click on ![](/Users/noli/Documents/UCD/teaching/CRD150/Lab/crd150.github.io/broom.png) to clear out all the objects loaded and created in your current session.
    + **History** - shows a list of executed commands in the current session.
    + **Connections** - you can connect to a variety of data sources, and explore the objects and data inside the connection.  I typically don't use this window, but you [can](https://support.rstudio.com/hc/en-us/articles/115010915687-Using-RStudio-Connections).    
* **Files, Plots, Packages, Help and Viewer tabs** (lower-right)
    + **Files** - shows all the files and folders in your current working directory (more on what this means later).
    + **Plots** - shows any charts, graphs, maps and plots you've successfully executed.     
    + **Packages** - tells you all the R packages that you have access to (more on this later).
    + **Help** - shows help documentation for R commands that you've called up.  
    + **Viewer** - allows you to view local web content (won't be using this much).

There is actually a fourth window, but we'll get to it shortly (oh, the suspense!)


<div style="margin-bottom:25px;">
</div>
##**Using R**
\

R is really just a big fancy calculator. For example, type in the following mathematical expression next to the `>` in the R console (left window)


```r
1+1
```

Note that spacing does not matter: `1+1` will generate the same answer as `1      +       1`.  Can you say hello to the world?
\


```r
hello world
```

```
## Error: <text>:1:7: unexpected symbol
## 1: hello world
##           ^
```

Nope. What is the problem here?  We need to put quotes around it. 


```r
"hello world"
```

```
## [1] "hello world"
```

"hello world" is a character and R recognizes characters only if there are quotes around it. Characters are one of R's four basic data types. You will learn more about these shortly (more suspense!)

In running the few lines of code above, we've asked you to work directly in the R Console and issue commands in an *interactive* way.  That is, you type a command after `>`, you hit enter/return, R responds, you type the next command, hit enter/return, R responds, and so on.  Instead of writing the command directly into the console, you should write it in a script.  The process is now: Type your command in the script. Run the code from the script.  R responds. You get results. You can write two commands in a script. Run both simultaneously. R responds.  You get results.  This is the basic flow. 

In your homework assignments, we will be asking you to submit code in a specific type of script: the R Markdown file.  R Markdown allows you to create documents that serve as a neat record of your analysis. Think of it as a word document file, but instead of sentences in an essay, you are writing code for a data analysis.  


Now may be a good time to read through the class [assignment guidelines](https://geo200cn.github.io/hw_guidelines.html) as they go through the basics of R Markdown files.  After you've gone through the guidelines, go through Robert Hijman's [Introduction to R](https://rspatial.org/intr/index.html): Basic data types, Basic data structures, Indexing, Algebra, Read and write files, Data Exploration, and Graphics.  We will cover the rest of his intro next week.


***


Website created and maintained by [Noli Brazil](https://nbrazil.faculty.ucdavis.edu/)
