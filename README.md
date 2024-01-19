# Unreal 5 Clang Tidy Guide

`Updated:` Jan 18, 2024

## Table of Contents
- [How to Use This Guide](#how-to-use-this-guide)
- [General Warning](#general-warning)
- [About Clang Tidy](#about-clang-tidy)
- [General](#general)
- [Helpful Links](#helpful-links)
- [Changing Unreal Code](#changing-unreal-code)
- [Setup](#setup)
- [Suppressing Checks](#suppressing-checks)
- [Trouble Shooting](#trouble-shooting)
  - [Config File Not Working](#config-file-not-working)
  - [Check Not Being Suppressed](#check-not-being-suppressed)
  - [Crashes](#crashes)
- [Tidy: Unreal Macros](#unreal-macros)
  - GENERATED_BODY
  - GENERATED_UCLASS_BODY
  - GENERATED_UINTERFACE_BODY
  - IMPLEMENT_PRIMARY_GAME_MODULE
  - IMPLEMENT_MODULE
  - check() and other check...() functions
- [Tidy: Check Warnings](#tidy-checks)

---
---

## How to Use this guide

The easiest way is to just search the name of the Check that you're getting the warning on.

See [Check Docs quick access](#check-docs-quick-access) on how to find the name to search for.

The Check warnings, listed below in this guide, follow this pattern:

  - Check: (name)
  - Warning message:
  - Example:
  - Notes:
  - Recommendation(s)

`Note:` The recommendations can have multiple choices which means it's up to you to pick the best choice.


[Top](#unreal-5-clang-tidy-guide)

---
---

## General Warning

- I'm not a C++, Unreal, or clang tidy expert. This is opinion based on what I currently know or researched.
- Some opinions might change with different context so the recommendations could be wrong for your situation
  - For example: There are a couple of times, with Lyra, I suppressed warnings because there were C++ comments that applied to the warning.

`Note:` Please use [Issues](https://github.com/boocs/unreal-clangd/issues) or [Pull requests](https://github.com/boocs/unreal-clangd/pulls) to help with any incorrect info or to add any warnings you get in your Unreal code

[Top](#unreal-5-clang-tidy-guide)

---
---

## About Clang Tidy

clang-tidy is a clang-based C++ “linter” tool. Its purpose is to provide an extensible framework for diagnosing and fixing typical programming errors, like style violations, interface misuse, or bugs that can be deduced via static analysis.

https://clang.llvm.org/extra/clang-tidy/


[Top](#unreal-5-clang-tidy-guide)

---
---

## General
- This was created with the free, Epic made, Lyra example project for Unreal 5.3
  - It can be used as a general guide for other projects
- This was tested with VSCode using the [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd) extension
- Other extensions/IDEs can also use Clang Tidy
    - But they `won't` be able to use the `.clangd` config file
    - You'll have to substitue it for `Comment` suppresion or `.clang-tidy` suppression

[Top](#unreal-5-clang-tidy-guide)

---
---

## Helpful Links
- [Unreal-clangd](https://github.com/boocs/unreal-clangd) 
  - My VSCode helper extension for Unreal/clangd
  - Helps setup a clangd config for Unreal (with clang-tidy option)
    - Tidy config is default off so you need to turn it on (or see [Setup](#setup))
  - Takes over `Intellisense` and `formatting` from the Microsoft's C++ extension
  - `note:` You can still use Microsoft's C++ extension for debugging
- [Clang Tidy docs](https://clang.llvm.org/extra/clang-tidy/index.html)
- [Clangd config docs](https://clangd.llvm.org/config) - More targeted suppression of Clang Tidy warngins using .clangd config
- [Epic C++ Coding Standard (UE 5.3)](https://docs.unrealengine.com/5.3/en-US/epic-cplusplus-coding-standard-for-unreal-engine/)


[Top](#unreal-5-clang-tidy-guide)

---
---

## Changing Unreal Code
- Be careful on changing code created by Epic.
- You never know if there's a strange Unreal or "Game Programming" reason on why it is the way it is.
- Here's a Check where I had an Exception when `changing the code`: [misc-use-anonymous-namespace](#check-misc-use-anonymous-namespace)

[Top](#unreal-5-clang-tidy-guide)

---
---

# Setup

If you're using my VSCode extension [Unreal-clangd](https://github.com/boocs/unreal-clangd) you can turn on `clang-tidy` by just creating the clang-tidy config file for your project.

`Note:` If your VSCode extension or other IDE supports clang-tidy, see their documentation on how to get it to work.

Setup:
1. Create a .clang-tidy file in your project's parent directory
2. Add this to your file(keep all formatting the same):
    ```
    ---
    Checks: |-
    -*,
    readability-*,
    cppcoreguidelines-*,
    bugprone-*,
    modernize-*,
    performance-*,
    misc-*
    CheckOptions:
    - key: readability-function-cognitive-complexity.Threshold
      value: 25
    FormatStyle: file

    # Keep blank line at end of file

    ```

The file has 3 sections: Checks, CheckOptions, and FormatStyle

### Checks 
This section is where you can enable and disable checks `Globally`.

The first line "-*," is used to first disable every default enabled check.

We then enable different categories and enable all their Checks using the '*' character,
```
readability-*,
cppcoreguidelines-*,
bugprone-*,
modernize-*,
performance-*,
misc-*
```

This is also where you can disable specific checks globally.

Example:
```
readability-*,
-readability-uppercase-literal-suffix,
cppcoreguidelines-*,
```
You can see the specific check, `readability-uppercase-literal-suffix`, is disabled by using a '-' prepended to it.

### CheckOptions
This is an example on how CheckOptions are formatted.

- In this example, Threshold is a option for the check.
- Threshold's value is the Check Option's default value.

CheckOptions are another way to suppress Check warnings using a specific option of a Check.

### FormatStyle
Used for Tidy 'Quick Fix' formatting

The file value means we're using a .clang-format file. My VSCode extension [Unreal-clangd](https://github.com/boocs/unreal-clangd) will auto create this file.

`Note:` If you're not using a .clang-format file you can change the value to `llvm` or `google`.


[Top](#unreal-5-clang-tidy-guide)

---
---
## Check Docs quick access

If you hover over a check, a window will pop up with the Check as a link you can click. Click the link to go to the documenation for the specific check.

- The link in blue is also the name you want to search this guide for

![image](/resources/warning-docs-quick-access.png)

`Note:` The check's page will sometimes show you the Check's advanced `Options` you can set in the `CheckOptions` section of .clang-tidy or .clangd config files.

[Top](#unreal-5-clang-tidy-guide)

---
---

## Suppressing Checks
We'll be using `four ways` to suppress clang-tidy checks

1. ### `Global` using .clang-tidy config file

    You can have multiple .clang-tidy files. Can be used for `Global` or `Directory` suppression of checks. Which ever .clang-tidy file is `closest` to your source file is the file that will be used.

    I'll only use one .clang-tidy file for `Global` suppression of checks.

    Example suppression of a Check using a prepended '-' character:
    ```
    readability-*,
    -readability-uppercase-literal-suffix,
    cppcoreguidelines-*,
    ```

2. ### `Per Check` using  .clangd config file

    You can have multiple .clangd files. Can be used for `Global`, `Directory`, `File(s)` or `Per Check`. We will be doing `Per Check`
    #### Example: Per Check
    By Adding more sections to your `.clangd` file you can suppress `per check` for multiple files.

    `Note:` .clangd files don't support backslash in paths

      ```
      # Rest of file above here
      ---  # 3 dashes create a new section
      If:
        PathMatch:
          - Source/LyraGame/LyraGameplayTags.cpp
          - Source/LyraGame/LyraGameplayTags.h
          - Source/LyraGame/LyraLogChannels.cpp
          - Source/LyraGame/LyraLogChannels.h
      Diagnostics:
        ClangTidy:
            Remove:
              - cppcoreguidelines-avoid-non-const-global-variables
      # Other Sections below here
      ```

3. ### `Comment` suppression using C++ comments

    You can use comments to prevent clang-tidy from parsing your code.

    - I'll be using '`// NOLINT`' , '`// NOLINTNEXTLINE`' , '`// NOLINT(Check-To-Suppress, ...)`' , and '`// NOLINTNEXTLINE(Check-To-Suppress, ...)`'

      - Using the `// NOLINT(Check-To-Suppress)` makes searching for suppressed Checks a lot easier.
      - I recommend using '(Check-To-Suppress)' on everything except macros

    - There's also `'// NOLINTBEGIN(Check-To-Suppress, ...)'` and matching `'// NOLINTEND(Check-To-Suppress, ...)'` but we won't be using them. See [Tidy docs](https://clang.llvm.org/extra/clang-tidy/index.html) for more info.

    `Note:` Many won't like `Comment` suppression, but on certain Checks it's unavoidable.
    
     For example, the Check that finds variables that haven't been initialized. You want to disable false Check warnings with `Comment` suppression and not by `Per Check` or `Global` suppression. Using `Comment` suppression allows Tidy to continue finding variables that haven't been initialized throughout the file or project.

4. ### `CheckOptions`

    Some `Checks` have options, that you can set, that also suppress. You can find these options in the documentation for the [specific check](#check-docs-quick-access).

    You can use CheckOptions in both config files:

    #### In .clang-tidy(Global):
      ```
          CheckOptions:
            - key: misc-non-private-member-variables-in-classes.IgnorePublicMemberVariables
              value: true
      ```
      

    #### In .clangd(Per Check):
      ```
      ---
      If:
        PathMatch:
          - Source/LyraGame/AbilitySystem/LyraAbilitySystemComponent.cpp
      Diagnostics:
        ClangTidy:
          CheckOptions:
            cppcoreguidelines-pro-type-member-init.IgnoreArrays: true

      ```
[Top](#unreal-5-clang-tidy-guide)

---
---

## Trouble Shooting

### Config File Not Working
- I've had trouble with .clangd/tidy config files `not` working when `not` putting a `empty line` at the `end of the file`.
- Mising comma after one or more of the entries in your `.clang-tidy` file (I've had this happen)

### Check not being suppressed
- Make sure there is a minus '`-`' sign in front of the check you're trying to suppress in your `.clang-tidy` file
- Some checks have `double checks` and come from different Tidy libraries. You might think you didn't suppress, but you did. It's just that the same Check from another category took over.
- On `Windows`, PathMatch section in .clangd needs to use '`/`' for paths.

### Crashes
It's rare, but clang-tidy can crash!

In the past, in my extension, I had added a `Global` suppression because of a crash in Lyra. This happened in UE 5.1 with Lyra.

```
readability-*,
-readability-static-accessed-through-instance,
cppcoreguidelines-*,
```

`This was wrong to do!`

If anything it should've been a `Per Check` or `Comment`(if a user were doing it). I believe the code could have also been changed.

`Note:` In Lyra using UE 5.3, this Check no longer crashes.


[Top](#unreal-5-clang-tidy-guide)

---
---
## Unreal Macros
For Unreal macros we use tidy `Comment` suppression to prevent Tidy parsing. 
  - This is because Macros can have multiple checks
  - This is also because Macros can cause errors that wouldn't normally occur in normal code. So you'd want to suppress these errors only for the Macro.
  

I use the general comment to suppress Check warnings in Macros:
```
GENERATED_BODY() // NOLINT
```

List of Macros that caused warnings and were ignored with Comment suppression:
- `GENERATED_BODY`
- `GENERATED_UCLASS_BODY`
- `GENERATED_UINTERFACE_BODY`
- `IMPLEMENT_PRIMARY_GAME_MODULE`
- `IMPLEMENT_MODULE`
- `check()` and other check...() functions
- `ensure` sometimes because of the parameters

`Note:` check() and other check...() functions are also mentioned below. They probably shouldn't have been though. I'll still leave them up since people might not realize they are macros.


[Top](#unreal-5-clang-tidy-guide)

---
---
## Tidy Checks
These are all the warning Checks found in Lyra(UE 5.3). Hopefully other devs will add warning Checks found in their code!

They follow a pattern of:
  - Check: (name)
  - Warning message:
  - Example:
  - Notes:
  - Recommendation(s)

### Check: `modernize-use-trailing-return-type`
#### Warning Message: 
    Use a trailing return type for this function

#### Example:
```
FString GetClientServerContextString(UObject* ContextObject)
{
```
#### Notes:
 UFUNCTION doesn't support this so we don't. Whenever this is supported then this will be categorized as project choice.

#### Recommendation: 
- `Global suppression`(.clang-tidy)

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-narrowing-conversions`
### Check: `cppcoreguidelines-narrowing-conversions`
#### Warning Message: 
    Narrowing conversion from 'double' to 'float'

#### Example: n/a

#### Notes: 
- Can be other types
- I believe Epic ignores these when Building. Ignore Globally or Per Check if you want.


#### Recommendation: Project choice.
- `Global` suppression
- `Per Check` suppression. Allows you to keep track of which files have this warning.

- `Special Comment Supression`

  Using Tidy allows us to suppress the Checks and also allows developers to find them easily.
Narrowing conversions can cause multiple Tidy warnings so we use `Tidy's special character` to suppress any warning ending with 'narrowing-conversions'.

  ```
  // NOLINT(*narrowing-conversions)
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-avoid-magic-numbers`
### Check: `readability-magic-numbers`
#### Warning Message: 
    100000.0f is a magic number; consider replacing it with a named constantclang-tidy
#### Example: n/a

#### Notes: 
  Tidy's default is to not count 0,1, and 100 as magic numbers. You can change this with a `CheckOption`.

  Magic numbers might not be that big a deal in the context of:

  https://docs.unrealengine.com/5.3/en-US/configuration-files-in-unreal-engine/ 

#### Recommendation:
- `Global` suppression
- `Per Check` suppression with a check option ignore. Allows you to keep track of files/values
  - This also allows you to use a `CheckOption` targetting a specific file or files with a custom list of allowed Magic Numbers
- `Comment` suppress 

  Using Tidy allows us to suppress the Checks and also allows developers to find them easily.
Magic Numbers can cause `multiple` Tidy checks so we use Tidy's `special` comment character '`*`' to suppress any Check that ends with 'magic-numbers'.

  ```
  // NOLINT(*magic-numbers)
  ```



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-auto`
#### Warning Message:
    Use auto when initializing with a template cast to avoid duplicating the type name

#### Example:
```
if (AActor* Actor = Cast<AActor>(ContextObject))
```

#### Notes: 

See here:

https://docs.unrealengine.com/5.3/en-US/epic-cplusplus-coding-standard-for-unreal-engine/#auto
#### Recommendation: Variable

- `Global suppression`(.clang-tidy)
- `Per Check`
- `Change code`(can Tidy quick fix)

  Tidy quick fix example:
  ```
  if (auto* Actor = Cast<AActor>(ContextObject))
  ```

`Note:` If we adhere to the Unreal Code Stardard this wouldn't be allowed. I believe it should be though since the Cast clearly shows the type.

For other context that don't clearly show the type then I wouldn't change the code.




[Top](#unreal-5-clang-tidy-guide)

---
### Check: `cppcoreguidelines-explicit-virtual-functions`
### Check: `modernize-use-override`
#### Warning Message:
    'virtual' is redundant since the function is already declared 'override'

#### Example:
```
virtual void StartupModule() override
{
```

#### Notes:
 Epic's projects still use virtual

#### Recommendation: Project choice
- `Global suppression`(.clang-tidy)
- `Change code` (Needs testing)


[Top](#unreal-5-clang-tidy-guide)

---
### Check: `cppcoreguidelines-avoid-non-const-global-variables`
#### Warning Message: 
    Variable 'TestVar' is non-const and globally accessible, consider making it const

#### Example: n/a

#### Notes: 
Of course you always want to try to make a var const if possible but with game dev sometimes you do things unorthodox. Lyra has this warning a lot.

#### Recommendation: Variable.
- For Lyra I would do it `Per Check` in .clangd file.

  This is because of Lyra's Gameplay Tags/Ability system
 
  Allows you do get the warning, acknowledge it, and then suppress it.
  
- `Use best judgement`. There are, of course, times though you may want to fix the code. 

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-vararg`
#### Warning Message:

#### Example:
```
UE_LOG(LogLyra, Display, TEXT("Could not find exact match for tag [%s] but found partial match on tag [%s]."), *TagString, *TestTag.ToString());
```

#### Notes: Happens on all UE_LOG lines

#### Recommendation: 
- `Global` suppression(.clang-tidy)


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-else-after-return`
#### Warning Message: 
    Do not use 'else' after 'return'

#### Example: from Source\LyraGame\LyraLogChannels.cpp
```
  if (Role != ROLE_None)
  {
	  return (Role == ROLE_Authority) ? TEXT("Server") : TEXT("Client");
  }
  else
  {
#if WITH_EDITOR
	  if (GIsEditor)
	  {
```

#### Notes: n/a

#### Recommendation: 
- `Use best judgement`

- There might be code where you suppress this but, for `this instance`, we `change` the code and remove the else (Tidy can auto fix this)
- There are other instances in Lyra that also give this check warning.


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-avoid-const-params-in-decls`
#### Warning Message: 
    Parameter 'WorldType' is const-qualified in the function declaration; const-qualification of parameters only has an effect in function definitions

#### Example:
![image](/resources/readability-avoid-const-params-in-decls.png)

#### Notes: 
    Doesn't affect your code

#### Recommendation: Project choice
- `Global` suppression
- `Change` the code

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-suspicious-include`
#### Warning Message: 
    Suspicious #include of file with '.cpp' extension

#### Example:
```
#include UE_INLINE_GENERATED_CPP_BY_NAME(LyraGamePhaseSubsystem)
```
#### Notes: In Lyra, Epic uses these a lot.

#### Recommendation:
- `Global`(.clang-tidy) suppression
- `Comment` suppression



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-equals-default`
#### Warning Message:
    Use '= default' to define a trivial default constructor

#### Example:

![image](/resources/modernize-use-equals-default.png)

#### Notes: 
Epic knows best? One of those where you don't question Epic but remind yourself to research some time in the future.

#### Recommendation: 
`Per Check` suppression to allow you to find and test these in the future

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-implicit-bool-conversion`
#### Warning Message: Implicit conversion 'FGameplayAbilitySpec *' -> bool

#### Example:
```
if (FoundSpec && FoundSpec->IsActive())
{
```

#### Notes: n/a

#### Recommendation: Project choice.
- `Global(.clang-tidy) suppression`
- `Change code`(can Tidy quick fix)

Here's what `Tidy quick fix` does:
```
if ((FoundSpec != nullptr) && FoundSpec->IsActive())
{
```


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-const-cast`
#### Warning Message:
    Do not use const_cast

#### Example:
![image](/resources/cppcoreguidelines-pro-type-const-cast.png)

#### Notes: 
One of those game dev is not the same as C++ dev things...

#### Recommendation: Variable
- `Per check` suppression to keep track of which files have these(.clangd)
- `Global` suppression(.clangd)

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `performance-unnecessary-value-param`
#### Warning Message:
    The parameter 'PhaseEndedCallback' is copied for each invocation but only used as a const reference; consider making it a const reference

#### Example: 
```
void ULyraGamePhaseSubsystem::StartPhase(TSubclassOf<ULyraGamePhaseAbility> PhaseAbility, FLyraGamePhaseDelegate PhaseEndedCallback)
{
```

#### Notes: 
- Context matters on this warning!
- Need to research/test if changing code is a viable option
- More than likely, in this Lyra instance the CheckOptions suppression is the right call
  - This is because FLyraGamePhaseDelegate is a delegate type created by the DECLARE_DELEGATE_OneParam macro
- In your own code it might not have this context. It could be that changing the code it the right call


#### Recommendation: Variable
- Context matters
- `CheckOption` suppression
  - The 'AllowedTypes' Check Option allows you to specificy a semicolon-sperated list of names of types allowed to be passed by value.
- `Change` code if the context warrants. Code that deals with custom types should be looked at more carefully and shouldn't be changed without proper thought.
- Always research and test which option is best

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-simplify-boolean-expr`
#### Warning Message:
    Redundant boolean literal in conditional return statement

    Boolean expression can be simplified by DeMorgan's theorem

#### Example:
```
if (ConnectionType == EOnlineServerConnectionStatus::Normal || ConnectionType == EOnlineServerConnectionStatus::Connected)
{
	return true;
}

return false;
```

#### Notes:
- Some Unreal `ensure...` macros give this warning and can be suppressed

#### Recommendation: Variable.
 - `Simplify the code` 
 - `Comment Suppression`. 
 
 You determine the best course of action. Most often I would simplify the boolean expression. There are times where I would keep it though so it's up to you. For example, one error where this occurred had some commented code inside the 'if' statement. I 'Comment Suppressed' the warning because of this.

 With another one of these errors, I did simplify.

 Example with quick fix:
 ```
 return ConnectionType == EOnlineServerConnectionStatus::Normal || ConnectionType == EOnlineServerConnectionStatus::Connected;
 ```

 `note`: It can also be choice:

 It will want to change this:

    if (RepList.RemoveFast(ActorInfo.Actor) == false)
 To this:

    if (!RepList.RemoveFast(ActorInfo.Actor))

 You might think it's more readable the first way.


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-use-anyofallof`
#### Warning Message: 
    Replace loop by 'std::ranges::any_of()'

#### Example: 
```
for (const auto& KVP : ActivePhaseMap)
{
```

#### Notes: 
Even though there has been some easing of the use of some of the standard library, 'For loops' are still being recommened to be done like the above example.

#### Recommendation: 
- `Global` suppression in .clang-tidy


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-function-cognitive-complexity`
#### Warning Message: 
    Function 'MyFunction' has cognitive complexity of 43 (threshold 25)

#### Example: n/a

#### Notes: 
This warning is about the complexity of a function.

#### Recommendation: 
1. Use a global `CheckOption` to up the `threshold` to whatever you think is best.

2. Comment Suppress any function that goes over the `threshold` setting so you can make note of functions that may need looking after.

Check Option example:
```
- key: readability-function-cognitive-complexity.Threshold
  value: 150
```
2. Comment Suppress example:
```
// NOLINTNEXTLINE(readability-function-cognitive-complexity)
void UMySubsystem::OnBeginComplexFunction(const UAbility* MyAbility, const FSpecHandle MyHandle)
{
```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `misc-unused-parameters`

#### Warning Message: 
    Parameter 'MyParam' is unused

#### Example: n/a

#### Notes: 
Function parameters are often unused with Unreal game dev

#### Recommendation: 
- `Global` suppression in .clang-tidy


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-redundant-access-specifiers`
#### Warning Message: 
    Redundant access specifier has the same accessibility as the previous access specifier

#### Example: 
```
protected:

	virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;
	virtual void EndAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, bool bReplicateEndAbility, bool bWasCancelled) override;

protected:

	// Defines the game phase that this game phase ability is part of.  So for example,
```
#### Notes: 
In Unreal, redundant specifiers are often used to organize

#### Recommendation: 
- `Global` suppression in .clang-tidy

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-non-private-member-variables-in-classes`
### Check: `misc-non-private-member-variables-in-classes`
#### Warning Message: 
    Member variable 'Foo' has protected visibility

#### Example: 
```
protected:
  int32 Foo;
```

#### Notes: 
Lyra other Unreal projects still use protected class vars.

#### Recommendation: 
- `Global` suppression in .clang-tidy 
  - Keep an eye out for change in coding styles. Maybe protected vars won't always be ok for Unreal game dev.


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-uppercase-literal-suffix`
#### Warning Message: 
    Floating point literal has suffix 'f', which is not uppercase

#### Example: 
```
float BaseHeal = 0.0f;
```

#### Notes: 
Lower case 'f' for floating point literals are the default choice in Lyra and most Unreal projects.

#### Recommendation: Project choice. 
- `Global` suppression in .clang-tidy
- `Change` the code(Tidy quick fix available)



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `misc-use-anonymous-namespace`
#### Warning Message: 
    Function 'HealStatics' declared 'static', move to anonymous namespace

#### Example: 
```
static FHealStatics& HealStatics()
{
	static FHealStatics Statics;
	return Statics;
}
```
#### Notes: 

I tried changing the code. Build was successful  but `caused an Exception!`.

#### Recommendation: 

- `Global suppression`(.clang-tidy)
- `Per Check suppression`(.clangd)

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-prefer-member-initializer`
#### Warning Message: 
    'bOutOfHealth' should be initialized in an in-class default member initializer

#### Example: 
```
ULyraHealthSet::ULyraHealthSet()
	: Health(100.0f)
	, MaxHealth(100.0f)
{
	bOutOfHealth = false;
```
#### Notes: 
Lyra seems to like both. 

There was a pattern building where UPROPERTY variables would be in the initializer list and non-UPROPERTY variables would be initialized in the constructor. I ran into a case where that pattern stopped. 

#### Recommendation: Project choice. 

- `Global` Suppress and just have a clear rule.

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-nodiscard`
#### Warning Message:
    Function 'GetLyraAbilitySystemComponent' should be marked [[nodiscard]]

#### Example:
 In file Source\LyraGame\AbilitySystem\Attributes\LyraAttributeSet.h
```
  ULyraAbilitySystemComponent* GetLyraAbilitySystemComponent() const;
```
#### Notes: 
Can't think of a reason not to change the code

#### Recommendation: 
- `Change` the code.
  ```
  [[nodiscard]] ULyraAbilitySystemComponent* GetLyraAbilitySystemComponent() const;
  ```
- `Global` suppression if you don't care about this warning

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-macro-usage`
#### Warning Message: 
    Function-like macro 'ATTRIBUTE_ACCESSORS' used; consider a 'constexpr' template function
    Variadic macro 'IMAGE_BRUSH' used; consider using a 'constexpr' variadic template function

#### Example: 
```
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```
#### Notes: n/a

#### Recommendation: Variable

- For most Unreal instances I'd use the `Global CheckOption` to exclude the macro name in .clang-tidy. 

  I would definitely add any other Unreal macros, that gives this error, to the value string. For `multiple macro names` in the value string, separate them using the '`|`' character.
  ```
  - key: cppcoreguidelines-macro-usage.AllowedRegexp
    value: "ATTRIBUTE_ACCESSORS|LOCTEXT_NAMESPACE"
  ```
- Could also do `Global` or `Per Check` suppression

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-using`
#### Warning Message: 
    Use 'using' instead of 'typedef'

#### Example: 
```
typedef TFunctionRef<bool(const ULyraGameplayAbility* LyraAbility, FGameplayAbilitySpecHandle Handle)> TShouldCancelAbilityFunc;
```
#### Notes:
- `Don't` use 'using' in the global scope
- `note:` See here for more 'using' rules https://docs.unrealengine.com/5.3/en-US/epic-cplusplus-coding-standard-for-unreal-engine/#namespaces


#### Recommendation: Project choice.
- `Per Check` suppression
- `Comment` suppression
- `Global` suppression

- `Change` code? Ceck the "Notes:" above


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-avoid-c-arrays`
### Check: `cppcoreguidelines-avoid-c-arrays`
#### Warning Message: 
    Do not declare C-style arrays, use std::array<> instead

#### Example: 
```
int32 ActivationGroupCounts[(uint8)ELyraAbilityActivationGroup::MAX];
```
#### Notes: 
Notice it's a small array of int32. Game dev will have you doing things different and this is one example. Lyra uses them for light weight arrays more than once.

#### Recommendation: Variable.
- `Per Check` in your .clangd file to keep track of which files have this warning
- `Comment` suppression


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-identifier-length`
#### Warning Message: Variable name 'PC' is too short, expected at least 3

#### Example: 
```
if (AController* PC = CurrentActorInfo->PlayerController.Get())
{
```
#### Notes: 
CheckOption has both `.IgnoredVariableNames` and `IgnoredParameterNames`. It also many more CheckOptions if you want to check the [docs quick access](#check-docs-quick-access)

#### Recommendation: Project choice
- `Change` code. You can have short names for variables that are known but should think about changing confusing short named variables.
- `CheckOption` regex that lets you add names to ignore(good for short names known to Unreal devs). 

  Example:
  ```
  - key: readability-identifier-length.IgnoredVariableNames
    value: "PC|C|LP"
  - key: readability-identifier-length.IgnoredParameterNames
    value: "Ar|PC|YL"
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-redundant-member-init`
#### Warning Message: 
    Initializer for base class 'FGameplayEffectContext' is redundant

#### Example: 
```
FLyraGameplayEffectContext()
	  : FGameplayEffectContext()
{
}
```
#### Notes: n/a

#### Recommendation: Project preference
- `Global` ignore
- `Change` the code(as long as Tidy is correct)


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-owning-memory`
#### Warning Message:
    Initializing non-owner 'FLyraGameplayEffectContext *' with a newly created 'gsl::owner<>'

#### Example: 
```
FLyraGameplayEffectContext* NewContext = new FLyraGameplayEffectContext();
```
Another from LyraMemoryDebugCommands.cpp:
```
delete OutputFile;
```
#### Notes: 
From Source\LyraGame\AbilitySystem\LyraGameplayEffectContext.h

I believe this gets put into Unreal's garbage collection system.

#### Recommendation: Depends
Epic sees no problem using new and delete for specific noncommon stuff

- `Per Check` in .clangd
- `Comment` suppression
- `Change` code if it's warranted. 
  - Use new/delete sparingly and carefully
  - You can search Lyra on how Epic used it
  - Most devs won't need to


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-easily-swappable-parameters`
#### Warning Message: 
    2 adjacent parameters of 'MyFunction' of similar type ('MyType') are easily swapped by mistake
#### Example: 
The two FGameplayTag parameteres are marked by Tidy

    HandleChangeInitState(UGameFrameworkComponentManager* Manager, FGameplayTag CurrentState, FGameplayTag DesiredState)

#### Notes: 

#### Recommendation: 
- Leave Epic's functions alone
- Think about how you design your own functions and if you care about this.
- `Check option`,  IgnoredParameterNames, to ignore certain parameter names (there are a bunch of Check options)
- Or just `Global suppression` in .clang-tidy


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-union-access`
#### Warning Message: 
    Do not access members of unions; use (boost::)variant instead

#### Example: n/a
#### Notes: 

#### Recommendation: 
- `Global` suppression


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-return-braced-init-list`
#### Warning Message: 
    Avoid repeating the return type from the declaration; use a braced initializer list instead
#### Example: 
```
if (ASC->HasMatchingGameplayTag(TAG_Gameplay_MovementStopped))
{
	return FRotator(0,0,0);
}
```
#### Notes: n/a

#### Recommendation: Project Choice (I like the code change TBH)
- `Global` suppression
- `Change` code(Tidy quick fix)

  ```
  if (ASC->HasMatchingGameplayTag(TAG_Gameplay_MovementStopped))
  {
	  return {0,0,0};
  }
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-named-parameter`
#### Warning Message: 
    All parameters should be named in a function
#### Example: 
```
void ALyraCharacter::OnDeathStarted(AActor*)
```
#### Notes: n/a

#### Recommendation: 

- `Change` code(can Tidy quick fix)

  Tidy Quick Fix pastes a /\*unused\*/ for the parameter which fixes the warning:

      void ALyraCharacter::OnDeathStarted(AActor* /*unused*/)

- `Global` suppression

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-isolate-declaration`
#### Warning Message: 
    Multiple declarations in a single statement reduces readability
#### Example: 
```
double AccelXYRadians, AccelXYMagnitude;
```
#### Notes: 
These are used as OUT variables and are set on the next line

#### Recommendation: Variable
- `Per Check`
- `Global`
- `Comment`
  

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-init-variables`
#### Warning Message: 
    Variable 'AccelXYRadians' is not initialized
#### Example: 
```
double AccelXYRadians, AccelXYMagnitude;
```
#### Notes: 
These are OUT variables that are set on the next line

#### Recommendation: 
- `Comment` suppression

  Unfortunately this will be a comment suppression recommendation.
You definitely want to be warned of variables that aren't initialized.

  The above example is a mistake since these are two OUT variables that are set on the next line. So we Comment suppress like so:

      double AccelXYRadians, AccelXYMagnitude; // NOLINT(cppcoreguidelines-init-variables)



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-make-member-function-const`
#### Warning Message: 
    Method 'MyFunction' can be made const
#### Example: n/a

#### Notes:

Use your own best judgement on changing your function to a const function.

Doesn't seem like it would hurt anything... 

#### Recommendation:
- `Change`code



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-convert-member-functions-to-static`
#### Warning Message: 
    Method 'GetDesiredThread' can be made static
#### Example: 
```
ENamedThreads::Type GetDesiredThread() { return ENamedThreads::GameThread; }
```
#### Notes: 

#### Recommendation: Use best Judgement
- `Change` code (Didn't seem like it hurt)
- `Per Check`
- `Global`


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-cstyle-cast`
#### Warning Message: 
    Do not use C-style cast to downcast from a base to a derived class; use dynamic_cast instead
#### Example: 
```
if ((BaseEffectContext != nullptr) && BaseEffectContext->GetScriptStruct()->IsChildOf(FLyraGameplayEffectContext::StaticStruct()))
{
	return (FLyraGameplayEffectContext*)BaseEffectContext;
}
```
#### Notes: 
- You aren't suppose to use dynamic_cast. If you need to use dynamic_cast you can use Cast<>()
- This isn't to say you should change this
- This could be one of those 'game programming' or Unreal situations
  - Example: It does check `IsChildOf()` in the `if` statement

#### Recommendation: Variable
- `Per Check` suppression. You want the warning, in other files, but don't want to use Comment Suppression.
- `Comment` suppression. You want to be warned. In this case it's probably fine since the 'If' statement does check IsChildOf
- `Change` code when warranted. 
  - Always research and test
  - Use Cast<>() instead of dynamic_cast

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-default-member-init`
#### Warning Message: 
    Use default member initializer for 'CartridgeID'
#### Example:

```
FLyraGameplayAbilityTargetData_SingleTargetHit()
		: CartridgeID(-1)
	{ }

virtual void AddTargetDataToContext(FGameplayEffectContextHandle& Context, bool bIncludeActorArray) const override;

/** ID to allow the identification of multiple bullets that were part of the same cartridge */
UPROPERTY()
int32 CartridgeID;
```

#### Notes: 

#### Recommendation: Project choice
- `Global` suppression
- `Change` code (Didn't seem like it hurt anything)

  ```
  FLyraGameplayAbilityTargetData_SingleTargetHit() { }

  virtual void AddTargetDataToContext(FGameplayEffectContextHandle& Context, bool bIncludeActorArray) const override;

  /** ID to allow the identification of multiple bullets that were part of the same cartridge */
  UPROPERTY()
  int32 CartridgeID{-1};
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-member-init`
#### Warning Message: 
    Constructor does not initialize these fields: ActivationGroupCounts
#### Example: n/a

#### Notes: 
You `don't` want to suppress this globally. You'll want to know if non-UPROPERTY variables aren't being initialized.

One time we got this because of an array. It was initialized in the constructor with FMemory::Memset

#### Recommendation: 
Multiple options depending on context
##### For Arrays:
- Per Check `Check option` to ignore arrays and mark files that have this warning.
- `Comment` suppression if you don't mind comments

  Per Check example in .clangd:
  ```
  ---
  If:
    PathMatch:
      - Source/LyraGame/AbilitySystem/LyraAbilitySystemComponent.cpp
  Diagnostics:
    ClangTidy:
      CheckOptions:
        cppcoreguidelines-pro-type-member-init.IgnoreArrays: true

    ```
##### For UPROPERTY Variables:
- `Per Check` suppression
- `Comment` suppression

  Per Check example:
  ```
  ---
  If:
    PathMatch:
      - Source/LyraGame/Camera/LyraUICameraManagerComponent.cpp
  Diagnostics:
    ClangTidy:
      Remove:
        - cppcoreguidelines-pro-type-member-init

  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-bounds-array-to-pointer-decay`
#### Warning Message: 
    Do not implicitly decay an array into a pointer; consider using gsl::array_view or an explicit cast instead
#### Example: 
```
FMemory::Memset(ActivationGroupCounts, 0, sizeof(ActivationGroupCounts));
```
#### Notes: 
ActivationGroupCounts is an array

#### Recommendation: Use best judgement since context matters for this one
I'm sure it's fine to suppress in the example context.
- `Per Check` suppression. To keep track of what files have this warning.
- `Global` suppression if you don't care to keep track
- `Comment` suppression


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-bounds-constant-array-index`
#### Warning Message: 
    Do not use array subscript when the index is not an integer constant expression
#### Example: 
```
ActivationGroupCounts[(uint8)Group]++;
```
#### Notes: n/a

#### Recommendation: 
- `Per Check` suppression in .clangd to keep track of all files that have the warning
- `Global` suppression if you don't care to track
- `Comment` suppression if you don't care about comments.

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-virtual-class-destructor`
#### Warning Message: 
    Destructor of 'ILyraAbilitySourceInterface' is protected and virtual
#### Example: 
```
class ILyraAbilitySourceInterface
{
	GENERATED_IINTERFACE_BODY()  // NOLINT
```
#### Notes:
 Has to do with GENERATED_IINTERFACE_BODY() macro. The macro defines the destructor as protected and virtual.

#### Recommendation: 
- `Per Check` suppression to keep track of files that have this warning
- `Global` suppression


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-special-member-functions`
#### Warning Message: 
    Class 'ILyraAbilitySourceInterface' defines a non-default destructor but does not define a copy constructor, a copy assignment operator, a move constructor or a move assignment operator
#### Example: 
```
class ILyraAbilitySourceInterface
{
	GENERATED_IINTERFACE_BODY()
```
#### Notes: 
  Has to do with GENERATED_IINTERFACE_BODY() macro.
#### Recommendation: 
- `Global` suppression. Epic designs these macros.

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-argument-comment`
#### Warning Message: 
    Argument name 'bCreateOnFail' in comment does not match parameter name 'bCreateProfileOnFail'
#### Example: 

Function arg:
```
  ..., /*bCreateOnFail=*/ false))
```
#### Notes: n/a

#### Recommendation: 

 - `Change` comment to correct name


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-avoid-do-while`
#### Warning Message: 
    Avoid do-while loops
#### Example: 
```
do
{
	NewIndex = (NewIndex + 1) % Slots.Num();
```
#### Notes: 
Of course if you don't have to use a do-while loops then don't

#### Recommendation: 
- `Global` suppression

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-inconsistent-declaration-parameter-name`
#### Warning Message: 
    Function 'ULyraEquipmentManagerComponent::EquipItem' has a definition with different parameter names
#### Example:
```
// .h
ULyraEquipmentInstance* EquipItem(TSubclassOf<ULyraEquipmentDefinition> EquipmentDefinition);

// .cpp
ULyraEquipmentInstance* ULyraEquipmentManagerComponent::EquipItem(TSubclassOf<ULyraEquipmentDefinition> EquipmentClass)
{
```
#### Notes: n/a

#### Recommendation: 
- `Change` code. Make parameter names the same. Doesn't matter which one you choose but choose one.


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-integer-division`
#### Warning Message: 
    Result of integer division used in a floating point context; possible loss of precision
#### Example: 
```
const float NumberYOffset = ((NumberIndex / FMath::Max(1, DamageNumberArrayLength - 1)) - 0.5f) * 2.f;
```
#### Notes: n/a

#### Recommendation: 
- `Per Check` suppression to keep track of which files have this warning
- `Comment` suppression for more precise monitoring
- `Change` code if you need to

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-qualified-auto`
#### Warning Message: 
    'const auto Component' can be declared as 'auto *const Component'
#### Example: 
```
for (const auto Component : OwningActor->GetComponents())
{
```
#### Notes:
Code change doesn't seem like it would hurt.

As always test your changes!

#### Recommendation: 
- `Change` code(Tidy quick fix)

  ```
  for (auto *const Component : OwningActor->GetComponents())
  {
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-const-return-type`
#### Warning Message: 
    Return type 'const float' is 'const'-qualified at the top level, which may reduce code readability without improving const correctness
#### Example: 
```
const float ULyraAimSensitivityData::SensitivtyEnumToFloat(const ELyraGamepadSensitivity InSensitivity) const
{
```
#### Notes: n/a

#### Recommendation: 
- `Change` code to remove first const from definition/implementation
- `Global` supppression if you don't care about this warning

Change code example:

.cpp
```
float ULyraAimSensitivityData::SensitivtyEnumToFloat(const ELyraGamepadSensitivity InSensitivity) const
{
```


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-avoid-const-or-ref-data-members`
#### Warning Message: 
    Member 'Options' of type 'TArray<FInteractionOption> &' is a reference
#### Example: 
```
private:
	TScriptInterface<IInteractableTarget> Scope;
	TArray<FInteractionOption>& Options;
```
#### Notes: 
This message appeared on a class called FInteractionOptionBuilder. A Builder class, especially in games, might be created differently.

#### Recommendation: Variable(context)
- `Per Check` suppress so you can keep track of which file the warning was
- `Change` code if you think it makes sense


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-pass-by-value`
#### Warning Message: 
    Pass by value and use std::move
#### Example: 

Constructor:
```
class FGameSettingEditCondition_VideoQuality : public FGameSettingEditCondition
{
public:
	FGameSettingEditCondition_VideoQuality(const FString& InDisableString)
		: DisableString(InDisableString)
```
#### Notes: 
- MoveTemp() is Unreal's std::move equivalent so use that instead
- Remember to always test.
- Quick Fix works but will use std::move so you'll have to change it to MoveTemp()

#### Recommendation:
- `Change` code if possible and warranted
- `Per Check` suppression to know what files have the warning
- `Comment` suppression for more precision


This seems to be fine:
```
class FGameSettingEditCondition_VideoQuality : public FGameSettingEditCondition
{
public:
	FGameSettingEditCondition_VideoQuality(FString  InDisableString)
		: DisableString(MoveTemp(InDisableString))
```


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `performance-no-automatic-move`
#### Warning Message: 
    Constness of 'AbsolutePath' prevents automatic move
#### Example: 
```
const FString AbsolutePath = IFileManager::Get().ConvertToAbsolutePathForExternalAppForRead(*LogFilename);
return AbsolutePath;
```
#### Notes:
Is Epic doing this on purpose or is it a bug?

#### Recommendation: Hard to say since don't know Epic's intent
`Change Code` and remove const?

Checking other Lyra functions that return FString shows that this is rare.
Doesn't seem like removing const would be a problem...

`Always test though!`



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-interfaces-global-init`
#### Warning Message: 
    Initializing non-local variable with non-const expression depending on uninitialized non-local variable 'GWorld'
#### Example: 
Lambda as a parameter:
```
FConsoleCommandWithWorldAndArgsDelegate::CreateStatic(
		[](const TArray<FString>& Params, UWorld* World)
{
```

GWorld:
```
const FString CollectionName = FString::Printf(TEXT("_LoadedAssets_%s_%s%s"), *FDateTime::Now().ToString(TEXT("%H%M%S")), *GWorld->GetMapName(), *CollectionNameSuffix);
```

LogLyra:
```
UE_LOG(LogLyra, Warning, TEXT("Wrote collection of loaded assets to %s"), *CollectionFilePath);
```
#### Notes: 
- I believe LogLyra is incorrectly marked. 
  - There's also a LogTemp that also seems incorrectly marked. Any errors with Log Categories can be ignored.
- GWorld is a different story.
  - This is the only use of GWorld in Lyra
  - If you hover over GWorld, it tells you to try not using it
  - World is sent as a lambda parameter and looks like you can replace GWorld with it

#### Recommendation: Variable
- `Per Check` suppression to keep track of files that have this warning
- `Comment` suppression for more precise tracking.
- `Change` Code for GWorld?
  - Maybe there's some reason it was done this way?


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `bugprone-branch-clone`
#### Warning Message: 
    Switch has 2 consecutive identical branches

#### Example: 
```
case ELyraPlayerConnectionType::Player:
case ELyraPlayerConnectionType::InactivePlayer:
	//@TODO: Ask the experience if we should destroy disconnecting players immediately or leave them around
	// (e.g., for long running servers where they might build up if lots of players cycle through)
	bDestroyDeactivatedPlayerState = true;
	break;
default:
	bDestroyDeactivatedPlayerState = true;
	break;
```
#### Notes: 
Show above, this happens because they both have the same "bDestroyDeactivatedPlayerState = true;"

`Note:` This is different though because of the comment signifying that a code change is coming.

#### Recommendation: Depends
`Because of the comment` about code changes, we suppress.
- `Per Check` suppression
- `Comment` suppression for more precise tracking

`If no comment` about upcoming code changes:
- `Change` the code


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-concat-nested-namespaces`
#### Warning Message: 
    Nested namespaces can be concatenated
#### Example: 
```
namespace Lyra
{
	namespace Input
	{
```
#### Notes: Seems ok to change code

#### Recommendation: 
- `Change` code(Quick fix available)

What quick fix does:
```
namespace Lyra::Input
{
```

`Note:` The namespace will need to be formatted after the quickfix. Make sure  you have "`NamespaceIndentation: All`" in your .clang-format file.


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `performance-for-range-copy`
#### Warning Message: 
    Loop variable is copied but only used as const reference; consider making it a const reference
#### Example: 
```
void ULyraSettingKeyboardInput::RestoreToInitial()
{
	for (TPair<EPlayerMappableKeySlot, FKey> Pair : InitialKeyMappings)
	{
```
#### Notes: 
- In this instance, it seems Tidy is correct and you can change the code
- You'll have to determine what's right for your code

#### Recommendation: (Context matters)
- `Change` code(Quick fix available)

  What quick fix does:
  ```
  void ULyraSettingKeyboardInput::RestoreToInitial()
  {
	  for (const TPair<EPlayerMappableKeySlot, FKey>& Pair : InitialKeyMappings)
	{
  ```


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-duplicate-include`
#### Warning Message: 
    Duplicate include
#### Example: n/a

#### Notes: n/a

#### Recommendation: 
- `Change` code(Quick fix available)

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-delete-null-pointer`
#### Warning Message: 
    'if' statement is unnecessary; deleting null pointer has no effect
#### Example: 
```
if (DefaultContextInternal)
{
	delete DefaultContextInternal;
}
```
#### Notes: 
Seems ok to change code

#### Recommendation: 
- `Change` code(Quick fix available)

Quick fix removes 'if' statement but leave the delete:
```
  delete DefaultContextInternal;
```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `misc-no-recursion`
#### Warning Message: 
    Function 'ProcessLoginRequest' is within a recursive call chain
#### Example: n/a

#### Notes: n/a

#### Recommendation: 
- `Per Check` suppression to mark files that have this
- `Comment` suppression for more precise marking

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-static-cast-downcast`
#### Warning Message: 
    Do not use static_cast to downcast from a base to a derived class; use dynamic_cast instead
#### Example: 
```
if (FGameplayAbilityTargetData_SingleTargetHit* SingleTargetHit = static_cast<FGameplayAbilityTargetData_SingleTargetHit*>(LocalTargetDataHandle.Get(i)))
{
```
#### Notes:
- You're not suppose to use dynamic_cast in Unreal projects. You can use Cast<>() instead.
- This isn't to say you should change the code.
- This could be one of those it's 'game programming' or Unreal situations
- This could have already been vetted by Epic and shown to be fine

#### Recommendation: Variable
- `Per Check` suppress to mark files
- `Comment` suppress for more precise targeting

- `Test` using Cast<>() instead? Testing should also include performance!


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `performance-unnecessary-copy-initialization`
#### Warning Message: 
    The const qualified variable 'RedirectorPackageName' is copy-constructed from a const reference; consider making it a const reference
#### Example: 
```
const FString RedirectorPackageName = Params[0];
const FString TargetPackageName = Params[1];
```
#### Notes: 
- Don't see any reason not to change code but there could be a good reason why it is the way it is
- Make sure to test

#### Recommendation: (Context matters)
- `Change` code but test
- `Per Check` to keep track of files with warning
- `Comment` suppression for more precise tracking of warnings



[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-nullptr`
#### Warning Message: 

#### Example: 
```
LoadedPackage = LoadPackage(NULL, *DestPackageName, LoadFlags);
```
#### Notes:
- Don't use NULL
- This is the only use of NULL in Lyra's main project code
- Docs of 'LoadPackage' mention the parameter is usually nullptr or ULevel->GetOuter()

#### Recommendation: 
- `Change` code to nullptr


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-use-bool-literals`
#### Warning Message: 
    Converting integer literal to bool, use bool literal instead
#### Example: 
```
bool LogLoadingScreenReasonEveryFrame = 0;
```
#### Notes: 
- Can't think of any reason not to change the code to 'false'
- I say that but ran into this:

      if (0)  
- Looked like a way to comment out test code?
- I used `Comment` suppression for that one

#### Recommendation: 
- `Change` code usually
 
  ```
  bool LogLoadingScreenReasonEveryFrame = false;
  ```


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `modernize-loop-convert`
#### Warning Message: 

#### Example: 
```
for (int32 VerticeIndex = 0; VerticeIndex < UE_ARRAY_COUNT(Vertices); ++VerticeIndex)
{
```
#### Notes: 
- Don't see a reason not to
- As always it could be some 'game programming' or Unreal reason
- Perhaps performance?

#### Recommendation: 
- `Change` code but test

  Quick fix example:
  ```
  for (const auto & Vertice : Vertices)
  {
  ```

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-type-reinterpret-cast`
#### Warning Message: 
    Do not use reinterpret_cast
#### Example: 
```
auto ThunkCallback = [InnerCallback = MoveTemp(Callback)](FGameplayTag ActualTag, const UScriptStruct* SenderStructType, const void* SenderPayload)
{
	InnerCallback(ActualTag, *reinterpret_cast<const FMessageStructType*>(SenderPayload));
};
```
#### Notes: 
- You'll have to recognize advanced code and let it do its thing
- For you're own code make sure you have a really good reason to use it

#### Recommendation: 
- Try not to use it in your own code
- `Per Check` suppression to mark files that have them
- `Comment` suppression for more precise marking


[Top](#unreal-5-clang-tidy-guide)

---

### Check: `cppcoreguidelines-pro-bounds-pointer-arithmetic`
#### Warning Message: 
    Do not use pointer arithmetic
#### Example: n/a
#### Notes: 
- This was from a Unreal macro so it's safe to ignore if from there

#### Recommendation: 
- `Per Check` to target files that have the error
- `Comment` suppression for more precise targeting
- `Note:` If not an Unreal macro, then make sure you really need pointer arithmetic

[Top](#unreal-5-clang-tidy-guide)

---

### Check: `readability-static-accessed-through-instance`
#### Warning Message: 
    Static member accessed through instance
#### Example: 
```
DisplayDebugManager.SetFont(GEngine->GetSmallFont());
```
#### Notes: 
See no reason not to change code but needs to be thoroughly tested.

#### Recommendation: 
- `Change` code

What Tidy Quick fix does:

    DisplayDebugManager.SetFont(UEngine::GetSmallFont());


[Top](#unreal-5-clang-tidy-guide)

---
---