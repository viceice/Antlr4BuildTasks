# Antlr4BuildTasks

This package is a build tool for Antlr4 C# programs using Net Core 3.1 and the Antl4.Runtime.Standard package.
It is based on Harwell's excellent [Antlr4cs build tool](https://github.com/tunnelvisionlabs/antlr4cs/tree/master/runtime/CSharp/Antlr4BuildTasks),
but has been completely rewritten and extended to function as a wrapper for the official
Antlr4 Tool that uses Java to generate a parser for all targets, including C#.

To use this package, include PackageReference's for Antlr4BuildTasks and the Antlr4.Runtime.Standard
version you want to use, then set the Build Action for your grammar files to Antlr4.
The rest is taken care of by this tool.

# Installation of Prerequisites

The only prerequisite you need is to have Net Core 3.1 installed. The package will look at your CSPROJ
file to determine what runtime you have selected. Based on that, the appropriate Antlr Tool jar
will be used. You don't need to download the jar because the tool does it for you, but it caches the most
recent version with this package. Currently, this build tool will support versions 4.8 and 4.7.2 of Antlr4.Runtime.Standard.

If you must support another runtime, then you will need to set Antlr4ToolPath yourself.
Make sure you do not have a version skew between the Java Antlr tool and the runtime versions.

The tool also contains a JRE built for the antlr4-4.8-complete.jar using Java 11 and jlink.
If you want to use a different Java,
install Java tool chain, either [OpenJDK](https://openjdk.java.net/) or [Oracle JDK SE](https://www.oracle.com/technetwork/java/javase/downloads/index.html) and set JAVA_EXEC to the full path
of the Java executable.

If you are not taking the default Antlr runtime version, you should verify that you have these
variables set up as expected. Again, you don't need to do a thing if you are using the defaults.
Try
*"$JAVA_EXEC" -jar "$Antlr4ToolPath"*
from a Git Bash or
*"%JAVA_EXEC%" -jar "%Antlr4ToolPath%"*
from a Cmd.exe.
That should execute the Antlr tool and print out the options expected
for the command. If it doesn't
work, adjust JAVA_EXEC and Antlr4ToolPath. JAVA_EXEC should be the full
path of the Java executable; Antlr4ToolPath should be the full path of the Antlr
tool jar file. If you look at the generated .csproj file for the Antlr
Console program generated, you should see what it defaults if they
aren't set.

This package is a Net Standard assembly and works on Linux or Windows, and works only for Antlr4 grammars.

To set grammar specific options for the Antlr tool, use VS2019 file properties or set the options in the CSPROJ file.

Language support in Visual Studio 2019 itself--
e.g. colorized tagging of the Antlr grammar, go to definition, reformat, etc.--
is a separate product, not part of the build rules for Antlr grammar files,
which is what this package supports. You can use Harwell’s [Antlr Language Support](https://marketplace.visualstudio.com/items?itemName=SamHarwell.ANTLRLanguageSupport)
extension, my own [AntlrVSIX](https://marketplace.visualstudio.com/items?itemName=KenDomino.AntlrVSIX) extension, or another.

You can see the NuGet package in action [here on Youtube](https://www.youtube.com/watch?v=Flfequp_Dy4).
The Net Core code for the Antlr Hello World example is [here in Github](https://github.com/kaby76/AntlrHW).

# How to use

You can use the build plug-in independently of Visual Studio. You will need to add two
PackageReference items to your CSPROJ file (see below example).

The following example
will help you get started. Note, the &lt;Antlr4&gt; element is similar to that used
in Harwell's Antlr4BuildTasks, but uses different child elements as the wrapper programs
have different options. All these options can be viewed and set in Visual Studio for the
properties of the grammar file.

    <ItemGroup>
        <Antlr4 Include="ExpressionParser.g4">
            <Listener>false</Listener>
            <Visitor>false</Visitor>
            <GAtn>true</GAtn>
            <Package>foo</Package>
            <Error>true</Error>
        </Antlr4>
    </ItemGroup>
    <ItemGroup>
        <PackageReference Include="Antlr4.Runtime.Standard" Version="4.8" />
        <PackageReference Include="Antlr4BuildTasks" Version="8.0" />
    </ItemGroup>

You *must* include a PackageReference to Antlr4.Runtime.Standard and Antlr4BuildTasks.

For every Antlr grammar file you want to have the Antlr Tool run on, list
each as an element in an &lt;ItemGroup&gt; with the attribute Include set to the
name of the file. For all imported grammars by another grammar, do not add an &lt;Antlr4&gt;
entry for the imported grammar.

The tool takes the following parameters:

* &lt;Listener&gt; -- A bool that specifies whether you want an
Antlr Listener class generated by the tool.
* &lt;Visitor&gt; -- A bool that specifies whether you want an
Antlr Visitor class generated by the tool.
* &lt;GAtn&gt; -- A bool that specifies whether you want
Antlr to generate the ATN Dot diagrams.
* &lt;Package&gt; -- An C# identifier that specifies the namespace to wrap
the generated classes in.
* &lt;Error&gt; -- Use this to specify whether you want the tool to
flag warnings as errors and stop a build.
* &lt;LibPath&gt; -- A string that specifies the path for token and grammar files
for the Antlr tool.
* &lt;Encoding&gt; -- A string that specifies the encoding of the input grammars.
* &lt;DOptions&gt; -- A list of <option>=<value>, passed to the Antlr tool. E.g.,
language=Java.

Antlr generates files that may produce a lot of compiler warnings. To ignore those,
add the following &lt;PropertyGroup&gt; to you .csproj file.

    <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
        <NoWarn>3021;1701;1702</NoWarn>
    </PropertyGroup>
