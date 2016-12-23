---
layout: post
title: "OPUXL - auto-discovered excel user-defined-functions"
description: "We created an Excel addin to enable auto-discovered UDFs published via a json server"
tags: [dev, excel, opensource]
---

## Opuxl, what?

Opuxl is an Excel Addin which connects on startup to a language-agnostic server (usually an application running on localhost), discoveres all published functions and registeres them within Excel.
The user is than able to execute those functions from within the Excel sheet and the result will be displayed in either the cell or as a result matrix.

Opuxl Server: The server which publishes the functions and calculates the result for incoming function requests
Opuxl Addin: The Excel Addin which will discover all published functions and will register them within Excel

This is similar functionality as the famous Bloomberg Excel addin with its BDH function.

### Demo

{% include image.html path="posts/opuxl-demo.gif" path-detail="posts/opuxl-demo.gif" alt="Opuxl Demo" %}

## Introduction

We (my current company, [Patronas Financial Systems](https://patronas.com)) had the following problem: our customers who used our desktop application needed data out of the system to create documents and reports. (think: queries and table-joins for the data of displayed in our desktop application)

We needed a solution which should be easy to use for the end-user and ideally create Excel sheets directly, as people in our domain really know how to use Excel. We found [XLLoop](http://xlloop.sourceforge.net/) ("XLLoop is an open source framework for implementing Excel user-defined functions (UDFs) on a centralised server (a function server). 
") and liked the idea and its approach, but it was hard to understand and extend their codebase (not maintained since 2011 anymore).
So we created our own application using C# and the open-source library [ExcelDNA](https://exceldna.codeplex.com/) and open-sourced our result.

## How does it work

The Opuxl Server is language-agnostic and will just provide a well-structured JSON api. There is an "Initialize" request for which the server will return all available function signatures and a particular "function request" (eg =Opuxl.GetLatestWeather("Berlin")) for which the server will execute the requested function with the given arguments.
The result (or error message) is sent back to the Excel sheet and will be displayed in either the cell, or will be rendered as a matrix.

In our [demo application](https://github.com/PATRONAS/opuxl/tree/master/demos/java) we created an example Opuxl server using java with a simple java.net.ServerSocket and made provided different annotations to make it really easy to register an opuxl function.
(demos using C# and Nodejs are work-in-progress)

### Example

{% highlight java %}
@OpuxlMethod(name = "Pow", description = "Pows the given number")
public OpuxlMatrix pow(
    @OpuxlArg(name = "base", description = "The Base", type = ExcelType.NUMBER) final Double a,
    @OpuxlArg(name = "exponent", description = "The Exponent", type = ExcelType.NUMBER) final Double b) {
    // ...
}
{% endhighlight %}

## Within Excel

The Excel addin will try to retrieve all available functions on startup and register those as UDFs (User Defined Functions).
If such a function is executed from within Excel it will send the given arguments as a payload to the opuxl server and display the calculated response.

The Excel addin is able to:
* display single cell results, but also matrix results
* make us of the built-in Excel types
* remembers the size of a matrix, to correctly resize on recomputation
* works with 32 and 64 bit Excel
* provides auto-completion and function-wizard
* works with any server which provides JSON

The code is open-source and all contributions are highly appreciated: [Github Opuxl Repo](https://github.com/PATRONAS/opuxl)
