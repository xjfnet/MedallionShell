MedallionShell
==============

MedallionShell is a lightweight library that vastly simplifies working with processes in .NET apps. 

[Download the NuGet package](https://www.nuget.org/packages/medallionshell) [![NuGet Status](http://img.shields.io/nuget/v/MedallionShell.svg?style=flat)](https://www.nuget.org/packages/MedallionShell/)

Built on top of the powerful, yet [clunky](http://www.codeducky.org/process-handling-net/) [System.Diagnostics.Process API](http://msdn.microsoft.com/en-us/library/system.diagnostics.process(v=vs.110).aspx), the MedallionShell API streamlines common use-cases, removes [pitfalls](http://www.codeducky.org/process-handling-net/), and integrates Process handling with .NET [async/await](http://msdn.microsoft.com/en-us/library/hh191443.aspx) and [Tasks](http://msdn.microsoft.com/en-us/library/dd460717(v=vs.110).aspx).

```C#
// processes are created and interacted with using the Command class
var cmd = Command.Run("path_to_grep", "some REGEX");
cmd.StandardInput.PipeFromAsync(new FileInfo("some path"));
// by default, all output streams are buffered, so there's no need to worry
// about deadlock
var outputLines = cmd.StandardOutput.GetLines().ToList();
var errorText = cmd.StandardError.ReadToEnd();

// by default, the underlying Process is automatically disposed 
// so no using block is required

// and complex arguments are automatically escaped
var cmd = Command.Run("path_to_grep", "\\ some \"crazy\" REGEX \\");

// we can also do this inline using bash-style operator overloads
var lines = new List<string>();
var cmd = Command.Run("path_to_grep", "some REGEX") < new FileInfo("some path") > lines;
cmd.Wait();

// and we can even chain commands together with the pipe operator
var pipeline = Command.Run("path_to_grep", "some REGEX") | Command.Run("path_to_grep", "another REGEX");

// if you don't like using operators, you can use the equivalent fluent methods instead
var cmd = Command.Run("path_to_grep", some REGEX").RedirectFrom(new FileInfo("some path")).RedirectTo(lines);

// we can check a command's exit status using it's result
if (cmd.Result.Success) { ... }

// and perform async operations via its associated task
await cmd.Task;

// commands are also highly configurable
var cmd = Command.Run(
	"path_to_grep",
	new[] { "some REGEX" },	
	options: o => o
		// arbitrarily configure the ProcessStartInfo
		.StartInfo(si => si.RedirectStandardError = false)
		// option to turn a non-zero exit code into an exception
		.ThrowOnError()
		// option to kill the command and throw TimeoutException if it doesn't finish
		.Timeout(TimeSpan.FromMinutes(1))
		...
);

// and if we want to keep using a common set of configuration options, we 
// can package them up in a Shell object
var shell = new Shell(o => o.ThrowOnError()...);
shell.Run("path_to_grep", "some REGEX");
```

## Release Notes
- 1.2.1 Adds .NET Core support (thanks [kal](https://github.com/kal)!), adds new fluent APIs for each of the piping/redirection operators, and now respects StandardInput.AutoFlush when piping between commands
- 1.1.0 Adds AutoFlush support to StandardInput, and fixed bug where small amounts of flushed data became "stuck" in the StandardOutput buffer
- 1.0.3 Fixed bug with standard error (thanks <a href="https://github.com/nsdfxela">nsdfxela</a>!)
- 1.0.2 Fixed bug where timeout would suppress errors from ThrowOnError option
- 1.0.1 Allowed for argument ommission in Command.Run(), other minor fixes 
- 1.0.0 Initial release

## Building The Code
You will need:
- VisualStudio 2015 or higher (community edition is fine) [download](https://www.visualstudio.com/vs/community/)
- .NET Core Tooling for VisualStudio [download](https://www.microsoft.com/net/core#windows)
- .NET Core SDK for Windows [download](https://www.microsoft.com/net/core#windows)

MedallionShell can be built from VisualStudio or from the command line using [Cake](http://cakebuild.net/):

```
powershell ./build.ps1
```

Running the cake build script will build the solution and execute the tests against both .NET Core and the .NET Framework (this is key, since the XUnit runner in VS does not allow you to pick which build to use when running unit tests). Cake will also create the NuGet package.