---
title: Changeling - A Feature Morphing Creature
category: Security
tags: [red-team, pentest, automation]
---


There has been a lot of good work over the past several months surrounding the idea of improving payload development and generation for pentests and red team assessments. This post will be the first post in a continuing series that aims to add new methods to your arsenal, allowing you to build more payloads with less effort on your own assessments. The feature that we'll be taking a look at today is Embedded Resources in C# projects. This is a feature that will allow us to compile code once, and reuse it on multiple assessments (yes, with different shellcode AND configurations).


## The Problem

One of the most annoying things to me personally on an assessment is payload generation. Depending on the assessment, I can have anywhere between one and twenty teamservers that I am building payloads for. If I need to create 2 or 3 listeners per server, generate 32-bit and 64-bit shellcode for all listeners on all servers, roll over to a fresh Windows VM and use Visual Studio to build the potential 120 versions of the same payload, that has the potential to eat up a ton of my time. Obviously, as an operator, you're going to choose to only build the payloads you think you need, but it would be nice to have everything possible at your disposal, just in case.

And then it becomes even worse if, for whatever reason, you need to make a modification to your payload. Now you have to rebuild everything all over again! It literally makes my eyes gloss over thinking of manually compiling the same thing with different shellcode over and over again. Convert shellcode, copy, paste, compile, move the binary, repeat. If you've ever had to re-roll payloads in the middle of a large operation, you know the feeling I'm talking about. I'm so glad that, as a community, we've finally started finding some answers to this seemingly trivial problem.


## A Better Way

As I struggled with this problem, my team and I started to implement several solutions introduced to the community by people like Adam Chester ([@_xpn_](https://twitter.com/_xpn_)) and Steven Borosh ([@424f424f](https://twitter.com/424f424f)), who opened my eyes to the world of Azure DevOps (thank you). While experimenting with using DevOps to design frameworks for generating payloads, I stumbled across a file property in Visual Studio titled `Build Action`. Reading more on build actions, one of the options presented was `Embedded Resource`. I don't remember exactly what I was doing when I came across it, but it had reminded me of the ALPC privesc released by SandboxEscaper the year prior. In the privesc PoC released by SandboxEscaper ([@SandboxBear](https://twitter.com/SandboxBear)), there was an embedded DLL that needed to be swapped out with a DLL of your own making. This was as simple as using a tool to swap the resources out. So I thought to myself, surely there's a way to build a similar tool for .NET assemblies. Enter Changeling. Changeling is a scriptable .NET tool that can be used on Linux, Windows, or Mac OS to swap embedded resources into and out of .NET assemblies!


## Resource Embedding

In .NET projects, files can be added with specific build actions. One build action that you may be familiar with without even knowing it is the `Compile` action. A file's build action can be seen by selecting the file in the Solution Explorer, and then viewing the build action in the Properties window. In this case, we are interested in the `Embedded Resource` build action. This is what Microsoft has to say about marking a file with this action:

> The file is passed to the compiler as a resource to be embedded in the assembly. You can call System.Reflection.Assembly.GetManifestResourceStream to read the file from the assembly.

Perfect. So, according to this we can mark a file to be embedded into the assembly, and then use the Assembly class to read the resource back out. This sounds ideal for modifying something like shellcode in a payload. There were a few projects out there that touched on embedded resources, but none that I found people using in the offensive space. So, I decided to create Changeling, and here's how you can use it with your own payloads.  


## A Sample C# Project

For this walkthrough, we will create a simple shellcode executor that pops calc on a 64-bit Windows machine. For the C# portion, we'll be using the Process Injection (T1055) technique created by [@pwndizzle](https://twitter.com/pwndizzle) that can be found [here](https://github.com/pwndizzle/c-sharp-memory-injection/blob/master/thread-hijack.cs).

To start off, we'll create a new C# `Console App (.NET Framework)` project in Visual Studio with the above file as our source, and we'll call the project `ShellcodeLoader`.  We will also need shellcode to inject. We can get this from metasploit with:

```
+[root@coffee-kali: changeling]$ msfvenom --payload windows/x64/exec CMD=calc.exe -a x64 --platform windows -f raw -o calc.bin
No encoder or badchars specified, outputting raw payload
Payload size: 276 bytes
Saved as: calc.bin
```

Now that we have our shellcode, we include it in our project as an embedded resource by selecting `Project -> Existing Item` from the Visual Studio menu, and selecting our freshly created `calc.bin`. Now, we'll make sure we have the Solutions Explorer (`View -> Solutions Explorer`) and the Properties Window (`View -> Properties Window`) open in Visual Studio. When we select `calc.bin` in the solutions explorer, we are shown several properties of the file in the properties window. We'll set the Build Action property to Embedded Resource, as described earlier.

{% include figure image_path="/assets/images/2020-03-24-changeling-a-feature-morphing-creature/calc-bin-properties.png" width='30' %}

Now, we need code to read our embedded resource out. To do this, we simply replace the following line in pwndizzle's code:

_**original**_
```csharp
byte[] payload = new byte[112] {
    0x50,0x51,0x52,0x53,...
};
```

_**changeling-ready**_
```csharp
using System.Linq;

/* <snip> */

byte[] payload;
string resourceName = "calc.bin";

var assembly = System.Reflection.Assembly.GetExecutingAssembly();
resourceName = assembly.GetManifestResourceNames()
    .Single(str => str.EndsWith(resourceName));

using (System.IO.Stream stream = assembly.GetManifestResourceStream(resourceName))
{
    using (System.IO.BinaryReader reader = new System.IO.BinaryReader(stream))
    {
        payload = reader.ReadBytes((int)stream.Length);
    }
}
```

Now, you may be asking, "What's with that lambda function that resets my `resourceName` variable to some new value?". Great question, I'm glad you asked. When you create an embedded resource, such as `calc.bin`, it is actually stored as `<RootNamespace.ResourceName>`. In this case, that would be `ShellcodeLoader.calc.bin`. To make the code a little more reusable, this is the solution I've opted for.

Now, instead of having our shellcode converted to readable text and hardcoded, our project will simply read it from the raw resource we have embedded in it. This project is officially ready to be used with Changeling!


## Morphing Features with Creatures

Now that we have our newly generated `ShellcodeLoader.exe`, let's see what Changeling can do.

{% include figure image_path="/assets/images/2020-03-24-changeling-a-feature-morphing-creature/changeling-help-menu.png" %}

Changeling currently exists with three methods, `extract`, `list`, and `replace`, and these methods are pretty self-explanatory. They can be used to extract, list, and replace any embedded resource in the assembly you pass to Changeling. The implications of this are that you no longer have to recompile tools that use shellcode for each individual assessment. You can simply take the payload from a static place, use Changeling to swap out the embedded resource, and you're good to go!

## But What About Other Tool Settings?

Some of you are thinking, "That's great, but I have other settings that change from engagement to engagement." I would argue the same point. Sometimes, you want to build a payload with a certain set of "arguments". Maybe it's a persistence tool that writes to a registry key chosen per assessment. Or maybe you are using WMI event subscriptions for lateral movement on an engagement, and you want to keep the event names different between sets of infrastructure. You can use Changeling to solve this issue as well!

I like to use JSON, but XML would probably work as well. Here's how you can store JSON as a serializable Embedded Resource, and pull it back out during execution.

To start out, let's add a new JSON file to our project, and mark it's build action as `Embedded Resource`.

_**projectConfiguration.json**_
```json
{
  "TARGET_PROCESS":  "notepad"
}
```

Fairly simple. This will be the default argument to our tool. Next, we'll make sure the project is targeting .NET Framework 4.0+, and then add the `System.Runtime.Serialization` reference so that we can deserialize our embedded JSON blob. We'll also need some code to use that reference.

_**ProjectConfiguration.cs**_
```csharp
using System.IO;
using System.Text;
using System.Linq;
using System.Reflection;
using System.Collections.Generic;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Json;

[DataContract]
class ProjectConfiguration
{
    [DataMember]
    public string TARGET_PROCESS { get; set; }

    public static byte[] GetEmbeddedResource(string resourceName)
    {
        byte[] result;

        var assembly = Assembly.GetExecutingAssembly();
        resourceName = assembly.GetManifestResourceNames()
            .Single(str => str.EndsWith(resourceName));

        using (Stream stream = assembly.GetManifestResourceStream(resourceName))
        {
            using (BinaryReader reader = new BinaryReader(stream))
            {
                result = reader.ReadBytes((int)stream.Length);
            }
        }

        return result;
    }

    public static ProjectConfiguration GetEmbeddedSettings()
    {
        byte[] embeddedBytes = GetEmbeddedResource("projectConfiguration.json");
        embeddedBytes = Encoding.UTF8.GetBytes(Encoding.UTF8.GetString(embeddedBytes, 0, embeddedBytes.Length));

        ProjectConfiguration deserializedSettings = new ProjectConfiguration();
        var ms = new MemoryStream(embeddedBytes);
        var ser = new DataContractJsonSerializer(deserializedSettings.GetType());
        deserializedSettings = ser.ReadObject(ms) as ProjectConfiguration;
        ms.Close();

        return deserializedSettings;
    }
}
```

Through the use of the `System.Runtime.Serialization` reference, we can create a `DataContract` that is used to read in our JSON object from the resource it is stored as. Obviously, this is just an example that can be extended and used in ways I probably can't come up with, but I'm sure you can :)

Now, with just a small tweak, our main file can use this JSON configuration to determine what process to inject into!

```csharp
// Original way
// if (args.Length == 0) { Console.WriteLine("Please enter a process name"); System.Environment.Exit(1); }
// Process targetProcess = Process.GetProcessesByName(args[0])[0];

// With Changeling
ProjectConfiguration projectConfiguration = ProjectConfiguration.GetEmbeddedSettings();
Process targetProcess = Process.GetProcessesByName(projectConfiguration.TARGET_PROCESS)[0];
```

Now, when we compile this tool, we'll have a tool that loads our `calc.bin` file into the `notepad` process. If we ever want to change it, it's as easy as doing the following with changeling:

```bash
➜  coffeegist ls -l
total 584
-rw-r--r--  1 coffeegist  staff  286208 Mar 23 21:51 Changeling.exe
-rw-r--r--  1 coffeegist  staff   12288 Mar 23 21:48 ShellcodeLoader.exe

➜  coffeegist mono Changeling.exe list -a ShellcodeLoader.exe

Resource Name
-------------
ShellcodeLoader.calc.bin
ShellcodeLoader.projectConfiguration.json

➜  coffeegist mono Changeling.exe extract -a ShellcodeLoader.exe -r projectConfiguration.json

➜  coffeegist cat ShellcodeLoader.projectConfiguration.json.template
{
  "TARGET_PROCESS":  "notepad"
}%                                                                                                                                                                                                                          
➜  coffeegist vim ShellcodeLoader.projectConfiguration.json.template

➜  coffeegist cat ShellcodeLoader.projectConfiguration.json.template
{
  "TARGET_PROCESS":  "explorer"
}

➜  coffeegist mono Changeling.exe replace -a ShellcodeLoader.exe -r projectConfiguration.json -f ShellcodeLoader.projectConfiguration.json.template -o NewShellcodeLoader.exe

➜  coffeegist ls -l
total 616
-rw-r--r--  1 coffeegist  staff  286208 Mar 23 21:51 Changeling.exe
-rw-r--r--  1 coffeegist  staff   11776 Mar 23 21:53 NewShellcodeLoader.exe
-rw-r--r--  1 coffeegist  staff   12288 Mar 23 21:48 ShellcodeLoader.exe
-rw-r--r--  1 coffeegist  staff      39 Mar 23 21:53 ShellcodeLoader.projectConfiguration.json.template
```


#### BONUS #1

In case you haven't noticed yet, through the use of Mono, this tool can be run CROSS PLATFORM! No need to hop boxes just to do tool management anymore. You can use Changeling directly from your op box!


## Coffee Break!

And that just about wraps it up. Changeling was a simple project, but one that has added a drastic increase in productivity on engagements. We can now spend less time spinning up Visual Studio instance just to compile and recompile, and more time doing the fun stuff, like sipping on some Ethiopian coffee straight out of my Moccamaster :D

Keep an eye out for part 2, where we will explore combining Changeling with some other automation tools to get the most bang for our buck. Until then, happy hacking!


#### Bonus #2

If you're interested in more automation of Changeling with Cobalt Strike, I'll soon be adding a script to generate multiple copies of your payloads even faster. Be sure to follow along to be one of the first to see it.


### Resources

- [https://github.com/coffeegist/changeling](https://github.com/coffeegist/changeling)

- [https://github.com/coffeegist/changeling-demo](https://github.com/coffeegist/changeling-demo)

- [https://github.com/pwndizzle/c-sharp-memory-injection/blob/master/thread-hijack.cs](https://github.com/pwndizzle/c-sharp-memory-injection/blob/master/thread-hijack.cs)
