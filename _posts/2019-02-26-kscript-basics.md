---
layout: post
date: 2019-02-26 01:00:00
title: Scripting with Kotlin - Kscript
description: kscript, an easy way to write and execute scripts in Kotlin. This post covers what kscript is, how to install it on a Mac, and offers some kscript examples.
permalink: 2019/02/scripting-with-kotlin-kscript/
categories: 
- Android
---

*This post discusses kscript, an easy way to write and execute scripts in Kotlin. This post covers what kscript is, how to install it on a Mac, and offers some kscript examples.*

## What is Kscript
> Enhanced scripting support for Kotlin on *nix-based systems.

While Kotlin has some support for scripting already, it lacks a few things which would help its adoption. Chief among them for me, is the extra friction caused by having to run it using `kotlinc`. 

That's not really a big deal, I know. And there's probably a way around it that I don't yet know about. But I wanted something as simple and familiar as executing a `.sh` bash script. 

`Kscript` offers not only a convenient way to run a script, but also a bunch of other stuff on top.

## Installing Kscript
There are few options for installation, but I usually prefer the `brew` option when it is available. If that's not your thing, see its [GitHub](https://github.com/holgerbrandl/kscript) page for more options.

To install using Brew, run: 

    brew install holgerbrandl/tap/kscript

## Your first kscript

1. Create a new file. Let's call it `hello-world.kts`. The `kts` suffix is optional; feel free to use something else or omit it if you don't like it.

2. Add the following to the file

    ```
    #!/usr/bin/env kscript
    
    println("Hello world!")
    ```

3. Make the file executable.

    ```
    chmod +x hello-world.kts
    ```


4.  Execute the script

    ```
    ./hello-world.kts
    ```
    

## Add functions
As you can see from the example above, you don't need any structure in the script. You don't need a `main` method and you don't need to put your code in functions. But just because you don't have to use functions, doesn't mean you can't.


```kotlin

    val greeting = personalizedGreeting("Craig")
    println(greeting)

    fun personalizedGreeting(name: String) : String {
        return "Hello $name!"
    }  

    // Hello Craig! 


```


## Passing arguments
You have access to a variable called `args` in the script, which will be magically populated for you with String arguments passed in when invoking the script.


```kotlin

println("Script is running with ${args.size} args passed")

for(arg in args) {
    println("arg: $arg")
}

```


Pass in arguments after the script name; separate with spaces.

    ./hello-world.kts foo bar 'one with spaces'
    
 This will output:

    // Script is running with 3 args passed
    // arg: foo
    // arg: bar
    // arg: one with spaces

## Accessing Other Classes
Need access to other classes? You can add an import statement before you use them. 

    import java.io.File

## Gradle / Maven Dependencies
If your code depends on dependencies, you can even reference these directly in the script file. It will automatically pull them down the first time the script is run (meaning you'll need internet access for that first time).

Use the normal Gradle-style dependency declaration, for declaring your `groupId`, `artifactId` and `version`.

```kotlin

    @file:DependsOn("com.squareup.retrofit2:retrofit:2.5.0")

    import retrofit2.Retrofit

    val retrofit = Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .build();

```

## Invoking a bash script from kscript
You can run a bash script using this handy function. 

```kotlin

    import java.io.File

    fun String.exec(dir: File? = null): Int {
        return ProcessBuilder("/bin/sh", "-c", this)
            .redirectErrorStream(true)
            .inheritIO()
            .directory(dir)
            .start()
            .waitFor()
    }

```


You should copy this into your script. To make use of it, define a string which matches your script name, and call the `exec` extension function on it.

For example, if you had this bash script called `bash-script.sh`

```
#!/bin/bash

echo "Hello from bash"
```

You can execute this script (if you've included the extension function above) like this:

    "./bash-script.sh".exec()



## Links
[kscript on GitHub](https://github.com/holgerbrandl/kscript)