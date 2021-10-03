**tl;dr: Use SonarLint and optionally StyleCop.**

The JavaScript ecosystem has amazing tools for formatting and statically analyzing your code: [Prettier](https://prettier.io/) and [ESLint](https://eslint.org/). Both tools have nearly universal adoption and deliver significant value.

But what about linting and formatting C# code?

For formatting, there's Visual Studio's autoformatter (Edit > Advanced > Format Document), but it only formats a single file at a time and mainly fixes indentation and whitespace issues. It doesn't split up long lines, refuses to format certain language constructs, and is generally far less opinionated than Prettier.

For linting, the C# compiler already has a few lint-like warnings, such as reminding you when you've forgotten to `await` an async method. These are helpful but only scratch the surface when compared to ESLint.

Given the limitations of these built-in solutions, I set out to find something better.

## Formatting: dotnet-format 🤔

[dotnet-format](https://github.com/dotnet/format) is a formatting tool that's being included in the upcoming .NET 6 SDK. If you're not on .NET 6 yet, you can still easily install dotnet-format with `dotnet tool install dotnet-format` (pass the `-g` option for a global install). Johnny Reilly has a [great post](https://blog.johnnyreilly.com/2020/12/22/prettier-your-csharp-with-dotnet-format-and-lint-staged/) about setting up dotnet-format together with lint-staged.

I was really excited about dotnet-format when I heard about it, but sadly, it did not live up to the hype. I used it to reformat a medium C# 9 codebase and inspected the diff. And... **all dotnet-format did was remove extraneous whitespace.** dotnet-format did not

- Break up long lines of code
- Enforce consistent placement of curly braces
- Add or remove blank lines
- Reformat method signatures & calls with inconsistent argument formatting such as
  ```c#
  void AddDocument(string name, FileId fileId, long fileSize,
     DocType type,
     bool discoverable,
     CategoryId? categoryId
  );
  ```

In dotnet-format's defense:

- It was extremely easy to set up.
- It can be completely automated — no manual intervention required.
- My test codebase was already formatted pretty well.

## Formatting: StyleCop 🙂

[StyleCop.Analyzers](https://github.com/DotNetAnalyzers/StyleCopAnalyzers) is a open source suite of C# code analyzers that is installed via a [NuGet package](https://www.nuget.org/packages/StyleCop.Analyzers). When you first install StyleCop and rebuild your solution, you'll likely see 10,000+ warnings. Wow! But don't worry; StyleCop has automatic fixes for most of these, and many of warnings are about trivial things like sorting your `using` statements alphabetically.

### Setup Instructions

These instructions are adapted from [this blog post](https://medium.com/@michaelparkerdev/linting-c-in-2019-stylecop-sonar-resharper-and-roslyn-73e88af57ebd).

Create a `Directory.Build.props` file next to your solution file and paste in the following XML.

```xml
<Project>
  <PropertyGroup>
    <CodeAnalysisRuleSet>$(SolutionDir)MyRuleset.ruleset</CodeAnalysisRuleSet>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference
      Include="StyleCop.Analyzers"
      Version="1.2.0-beta.354"
      PrivateAssets="all"
      Condition="$(MSBuildProjectExtension) == '.csproj'"
    />
  </ItemGroup>
</Project>
```

This adds the StyleCop.Analyzers NuGet package to every project in your solution. You should update the version number to whatever the latest is on NuGet.org. 1.2.0 introduces support for the latest C# features, so I recommend it even though it is still technically in beta.

Next, add a `MyRuleset.ruleset` file, also in the same directory as your solution file. I recommend opening the ruleset file with the XML editor rather than Visual Studio's ruleset GUI. [Here's my ruleset file, which you're welcome to copy.](https://gist.github.com/srmagura/d7b4df132d48087b25243fbdd1d44b37) If you'd like to build your own ruleset, simply copy my ruleset and delete all of the `<Rule />` elements.

Finally, rebuild the solution. StyleCop's suggestions will appear in the Error List and you'll get green Intellisense squigglies.

### Fixing StyleCop's Suggestions

**StyleCop's default ruleset is extremely opinionated and I recommend disabling rules that don't bring value to your team.** For example, the [SA1200](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/documentation/SA1200.md) rule requires that `using` statements be placed within the `namespace` declaration. While there's nothing wrong with this coding style, it's not widely used since Visual Studio puts `using` statements outside the namespace when you create a new C# file. Both conventions are [equally valid](https://stackoverflow.com/q/125319/752601), so I recommend disabling this rule. You can do this by adding an element the to `<Rules>` section of your `.ruleset` file:

```xml
<!-- Using directive should appear within a namespace declaration -->
<Rule Id="SA1200" Action="None" />
```

As you go through each StyleCop violation, you will encounter rules that are actually helpful. In every instance, I was able to automatically fix the violations across my entire solution via the quick fix menu.

### My Review

**StyleCop is a good tool and I recommend using it.**

The initial setup is not the most intuitive and I didn't agree with many of the rules (27 to be precise), _but_, it really cleaned up my codebase and the automatic code fixes are awesome.

As the name suggests, StyleCop is primarily concerned with code **style**, not correctness. If we want to fix bugs in our code, we'll have to keep looking...

## Linting: SonarLint 🤩

[SonarLint](https://www.sonarlint.org/) is an IDE extension and [NuGet package](https://www.nuget.org/packages/SonarAnalyzer.CSharp/) that analyzes your code for bugs, vulnerabilities, and code smells. The core product is open source, though the company seems to be pushing the IDE extension which integrates with their commercial products.

I tried out both the extension and the NuGet package and liked the NuGet package much better. Both options gave me Intellisense warnings, but only the NuGet package, SonarAnalyzer.CSharp, immediately showed me all the rule violations across my codebase. Moreover, a NuGet package is superior in a team setting since it will be downloaded automatically during build.

### Setup Instructions

SonarAnalyzer.CSharp is a collection of Roslyn analyzers just like StyleCop, so the setup is very similar. I'll be showing configuration files that include both StyleCop and SonarLint, but you can totally use SonarLint on its own.

Edit the `Default.Build.props` file next to your solution to install SonarAnalyzer.CSharp into all projects:

```xml
<Project>
  <PropertyGroup>
    <CodeAnalysisRuleSet>$(SolutionDir)MyRuleset.ruleset</CodeAnalysisRuleSet>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference
      Include="StyleCop.Analyzers"
      Version="1.2.0-beta.354"
      PrivateAssets="all"
      Condition="$(MSBuildProjectExtension) == '.csproj'"
    />
    <PackageReference
      Include="SonarAnalyzer.CSharp"
      Version="8.29.0.36737"
      PrivateAssets="all"
      Condition="$(MSBuildProjectExtension) == '.csproj'"
    />
  </ItemGroup>
</Project>
```

Multiple ruleset files are not supported, so your SonarLint rule customizations will go in the same `.ruleset` file we created earlier. [Here's my completed ruleset file.](https://gist.github.com/srmagura/73617725775de060800625d3fc466443) You can safely delete the StyleCop stuff if you're not using it.

Once that's done, rebuild the solution and you should see new warnings.

### Fixing SonarLint's Suggestions

As you go through each warning, you'll notice that some have automatic fixes while others don't. If you encounter a rule you don't like, you can disable it using the same method I showed for StyleCop. I generally agreed with SonarLint's suggestions and only disabled 8 out of the 409 rules.

When you encounter a rule violation you don't know how to fix, consult [SonarLint's extensive rule database](https://rules.sonarsource.com/csharp/RSPEC-112).

### My Review

**The SonarLint NuGet package was everything I hoped for and I strongly recommend it.**

By addressing each SonarLint warning, I was able to

- Learn about a [C# pitfall](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7irajf4fxriwwe926epv.png) that can easily result in bugs
- Identify several weak tests that made no assertions
- Have TODOs in the code appear as warnings in the Error List
- Fix a ton of inconsistently-named variables
- Fix methods that weren't using one of their arguments
- Add missing `static` modifiers to classes that only had static methods
- And more!

## Linting: ReSharper 🙁

[ReSharper](https://www.jetbrains.com/resharper/) is a Visual Studio extension from JetBrains that adds advanced refactoring and static analysis. I used ReSharper for the first two years of my professional career, and **found it to be a very helpful tool for learning C#'s advanced features.** That said, I stopped using ReSharper due to its multiple downsides and I haven't looked back.

- It's expensive, starting at $299/year.
- Visual Studio + ReSharper is a slow and buggy mess.
- Visual Studio's built-in refactorings have been steadily catching up to ReSharper's.
- Once you're experienced with C#, ReSharper's warnings can be annoying rather than helpful.

JetBrains does have their own .NET IDE, [Rider](https://www.jetbrains.com/rider/), which may be worth checking out.

## Conclusion

I highly recommend SonarLint for identifying bugs and code smells.

I'll be using StyleCop to enforce code formatting best practices, though dotnet-format is also a viable option. It's a tradeoff: StyleCop is more powerful but requires more setup and babysitting. dotnet-format is easy to install and can be completely automated as part of a precommit hook, but it won't fix many common style issues.
