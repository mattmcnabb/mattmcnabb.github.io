---
layout: post
title: Challenge Yourself with Powershell Programming Tasks
categories: [Powershell]
---

[RosettaCode]: http://rosettacode.org
[Hailstone]: http://rosettacode.org/wiki/Hailstone_sequence#.7B.7Bheader.7CPowerShell.7D.7D
[100Doors]: http://rosettacode.org/wiki/100_doors
[DigitalRoot]: http://rosettacode.org/wiki/Digital_root
[SumDigits]: http://rosettacode.org/wiki/Sum_digits_of_an_integer
[IBeforeE]: http://rosettacode.org/wiki/I_before_E_except_after_C

### Scripting Outside the Box
If you are like me, you probably took up Powershell as a way to address a particular business need. For me, that need was managing some aspects of Active Directory and Office 365. This is a great approach to learning the language, but it has its drawbacks as well. If you are only using Powershell to deal with the systems that you normally manage then you can get caught up 'in the box' and your learning can stagnate. A challenge I faced early on was how to keep the pace of learning up and continue to challenge myself. What I began to look for were arbitrary challenges that could help me take my Powershell problem solving to the next level. One website I found for this is [Rosetta Code][RosettaCode].

Rosetta Code is a wiki community that focuses on comparing programming tasks in different languages. You can choose a programming task and then post your solution to the problem in the programming language of your choice. Some of the tasks are relatively complex math puzzles, others are word-based, and some are simply common tasks like reading text from a file or printing to the screen.

### A Sample Task
My favorite tasks on Rosetta Code are the number challenges that got me thinking about how to achieve a Powershell solution in a more 'programmer-y' way. Here is one of the solutions I found that uses recursion to achieve a goal - [The Hailstone Sequence][Hailstone]. The task's description goes like this:

*The Hailstone sequence of numbers can be generated from a starting positive integer, n by:*

- *If n is 1 then the sequence ends.*
- *If n is even then the next n of the sequence `= n/2`*
- *If n is odd then the next n of the sequence `= (3 * n) + 1`*

*The (unproven), Collatz Conjecture is that the hailstone sequence for any starting number always terminates.*

_**Task Description:**_

1. *Create a routine to generate the hailstone sequence for a number.*
2. *Use the routine to show that the hailstone sequence for the number 27 has 112 elements starting with `27, 82, 41, 124` and ending with `8, 4, 2, 1`*
3. *Show the number less than 100,000 which has the longest hailstone sequence together with that sequence's length. (But don't show the actual sequence!)*

To solve the first two steps of the task, I wrote a function called Get-Hailstone:

{% highlight powershell %}
function Get-HailStone {
  param($n)
  switch($n) {
    {$_ -lt 1}     {return}
    1              {$n; return}
    {$n % 2 -eq 0} {$n; return Get-Hailstone ($n = $n / 2)}
    {$n % 2 -ne 0} {$n; return Get-Hailstone ($n = ($n * 3) +1)}      
  }
}
{% endhighlight %}

This fairly concise piece of code uses a switch statement to implement a recursive Hailstone algorithm. Wikipedia defines recursion as "*a method where the solution to a problem depends on solutions to smaller instances of the same problem*" - in other words, a block of code that uses itself to define itself. You can see this in practice in Get-Hailstone which calls Get-Hailstone inside it's function body.

The switch statement that makes up Get-Hailstone has four conditions. The first is simply a validation to ensure that the number provided to the function is not negative or zero.

The second condition is known as the base case. In recursive programming, the base case describes a condition where the recursion will terminate. If a base case is not included then the recursion can loop forever - like the infinite mirror trick. For the Hailstone sequence we set the base case to return when $n equals 1.

The third and fourth conditions are 'recursive cases' and decide how to modify $n based on the requirements of the Hailstone sequence. I use the modulus operator (%) to determine if $n is even or odd, output $n, and then calculate a new $n to be passed back into the function. If you aren't familiar with the modulus operator, it simply performs division and returns the remainder instead of the quotient. Thus using 2 as the divisor is an easy way to determine whether a number is even or odd.

Get-Hailstone is quick! On my laptop it can calculate the hailstone sequence for a 50-digit number in less than 200 milliseconds.

``` console
PS c:\> Measure-Command {Get-HailStone -n 19287549016253467837652413987654200198276352456276}

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 198
Ticks             : 1989228
TotalDays         : 2.30234722222222E-06
TotalHours        : 5.52563333333333E-05
TotalMinutes      : 0.00331538
TotalSeconds      : 0.1989228
TotalMilliseconds : 198.9228
```

### Conclusion
Here are some other tasks I have provided solutions to:

[100 Doors][100Doors]

[Digital Root][DigitalRoot]

[Sum Digits of an Integer][SumDigits]

[I Before E Except After C][IBeforeE]

So what are you waiting for? Get out there and attach an unsolved task, or let me know how you would solve the problems I posted.

**P.S.** For fun and extra credit, Google 'recursion.' ;)