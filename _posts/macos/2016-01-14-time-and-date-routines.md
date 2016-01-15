---
layout: post
title: Time and Date Routines
tags:
- UNIX
comments: true
---

* toc
{:toc}
---

This blog is a note of Chapter 6 (System Data Files and Information) in APUE.
<br><br>

The basic time service provided by the UNIX kernel counts the number of seconds that have passed since the Epoch: 00:00:00 January 1, 1970, Coordinated Universal Time (UTC). These seconds are represented in a `time_t` data type, and we call them *calendar times*. These calendar times represent both the time and the date.

The figure below shows the relationship of various time functions:
![relationship of the various time functions](../images/relationship_of_time_functions.png)

Conversion specifiers for `strftime`:
![conversion specifiers for strftime](../images/conversion_specifiers_for_strftime.png)

Conversion specifiers for `strptime`:
![conversion specifiers for strptime](../images/conversion_specifiers_for_strptime.png){: width="60%"}
