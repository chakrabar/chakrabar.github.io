---
layout: post
title: "Windows Workflow Foundation 4"
excerpt: "For internal purposes only"
date: 2018-12-24
tags: [coding, programming, wf, wwf, workflow]
categories: articles
comments: true
share: true
published: false
---

#### Declarative programming

* Asynchronous requests
* Transaction & compensation semantics
* Applications are composed rather than written, reuse of components across all levels
* Self-describing

Workflow Foundation lets you build workflow with different type of activities. Activities are similar to methods, but they are actually combinatin of instructons. They can be `Sequence` or `Parallel`, and you can combine them together to build complex workflows and invoke them with `WorkflowInvoker.Invoke(w)`.

#### Reactive programming

Programs that _"react"_ to triggers or stimulus. That trigger can be an user action, method call or API invokation.

* Spends most of the time waiting for stuffs to happen
* Challenging to design for scalability as waiting eats up resources

Now, these type of reactive programs (e.g. a web application backend) are written as bunch of independent APIs (e.g. web APIs) that work on a push model than poll/pull model. When a request comes, a workflow is invoked.

## Windows Workflow Foundation

`Windows Workflow Foundation` or **`WF`** is the `.NET` way to _compose_ programs totally declaratively. The whole programming logic is composed with `Activities`. The flow is saved as a `XAML` file and there is no code behind. Programming is only involved as creating custom activities

`WF` can be run in any `.NET` application. In many cases, `Activities` can be invoked as normal methods, call it and get back a result.

So, indirectly, `WF` gives us a way to write reactive programming (see next section), with declarative control flow and state management. A `WF` is basically a hierarchy or tree of activities. Many a times the workflow itself is the root activity.

**State Management** is provided out-of-the-box in WF applications. Just by changing some configuration, Wf can automatically manage your state and persist it as necessary. That means, it'll store the required state, fetch and use when required and discard automatically when no more required.

#### Activity

An activity is like a method in traditional programming, they all derive from base class `Activity`. Activities can be 

1. `Basic` (like, assign a variable) that does one thing
2. `Composite` or flow-control activities (e.g. an `if-else` flow control or `for-each` loop), that basically contain other activities. Composite activities can contain other composite activities. And there are WF specific activities `Parallel` which is still a composite activity.

`WF` comes with a bunch of default activities like

* Sequence
* Parallel
* Flowchart
* If, While, Do-While etc.

For any custom or business specific work, you need to write an `Activity`. Apart from basic activity, you can also write composite activity. **`System.Activities.Activity`** is the base class all custom activities need to derive from

1. Override `Execute` method to do the real work
2. Coposite activities are meant to manage the executon of child activities

**Activities are serializable**. So, programs are still just data. In `WF`, Activities are represented as `XAML` which is a `XML` based hierarchical representational language that can represent user interface and actions (can be compared to `HTML` and `JavaScript`). But it can also be expressed in many other hierarchical strutures like `JSON` or Database tables or Excel worksheets.
{: .notice--info}

This `C#` _"Imperative"_ code is equivalent to the following

```cs
Activity sequence = new Sequence
{
    Activities =
    {
        new WriteLine { Text = "Hello" },
        new WriteLine { Text = "World" }
    }
};
```

_"Declarative"_ `XAML` code (side note: notice each activity has got a unique ID)

```xml
<Activity xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities">
    <Sequence sap2010:WorkflowViewState.IdRef="Sequence_1">
        <WriteLine Text="Hello" sap2010:WorkflowViewState.IdRef="WriteLine_1" />
        <WriteLine Text="World" sap2010:WorkflowViewState.IdRef="WriteLine_2" />
    </Sequence>
</Activity>
```

Because of this `declararive` and `serializable` nature or WFs, they can be _**passivated**_ when idle. That means, the runtime can serialize and persist it (by default in SQL Server), and later rebuild (_resumption_) the workflow and invoke it when needed. In the process, the system can store the program and state (data) on disk for later use! This makes it a very efficient and scalable **reactive** programming model. They can also be passed around threads and processes.
{: .notice--success}

###### Included framework activities

* Control flow - While, ForEach, Sequence, Parallel, If, Pick, Switch
* Primitives & Collctions - Assign, Delay, Add/Remove from collection, InvokeMethod
* Flowchart - FlowDecision, FlowSwitch, FlowChart (_generally root level_ control flow with conditions). In flowchart, you can use it as container, and pull in other activities inside the body and then _connect_ them together to create a flow.
* Messaging - Send/ecieve WCF messages, correlation (working with WCF or exposing workflow as service)
* Tranaction & error handling - TransactionScope, TryCatch, Throw, CompensationScope, CancelationScope

#### More features or WF

The Workflow Foundation comes with some amazing bunch of features that makes it a super tool for building custom workflow, moving them around and also provide customized experiences for users to build their own workflows!

1. Declarative business rules - manage control flow and separate out rules from code
2. Tracking services - you can enable tracking & tracing across workflows just by changing config. It'll start putting debug trace and logs accordingly to already written activities. It provides enough information to trace them back to individual activities and verify.
3. Designers - comes with Out-of-box design surface that lets you build workflows interactively by dragging & dropping elements into the design surface. You can customize the designer user interface if you please. It also **lets you host the WF designer itself** inside `WPF` applications so that your users can build workflows with built-in and your custom activities with the designer!!You can select and customize even the built in activities for your custom designer.
4. Remember workflows can totally be created with `VB` or `C#` code, `XAML` is not mandatory. This is the imperative way of building workflows than the regular declarative one.

#### Arguments, Variables & Expressions

Workflows are nothing but activities. Most common root activities are workflows and sequences. They need to pass data through them, variables are used to store data & pass them within the workflow, while arguments are used to pass data in/out of workflow.

* Arguments - for passing data, with direction viz. In, Out, In/Out
* Variables - for storing data, scoped to a composite activity
* Expressions - for manipulating data, used in Activities

Variables & arguments can be of any `.NET` in-built type or custom type from referenced assemblies. 

Workflows take input **argument** in the form of `Dictionary<string, object>`, they `key` name must match the argument name in workflow definition. Output arguments are also returned in form of `Dictionary`. How the valu is returned depnds on how the workflow was invoked.

1. WorkflowInvoker.Invoke(new LeaveApprovalFlowchart()); - should run to completion without delays & input. The output is the return value of the method.
2. WorkflowApplication - runs episodic workflows, can run for long time, wait for inout etc. Gives access to the workflow instance and lifecycle events. Output arguments are passed to `completed` action.

```cs
//the workflow has In arguments name & age
//it returns an Out argument of type Person with name person
var name = "AC";
var age = 30;
var workflowresult = WorkflowInvoker.Invoke(
    new CreatePersonWorkflowWithArguments(),
    new Dictionary<string, object>
    {
        { "name", name },
        { "age", age }
    });
var outPerson = workflowresult["person"] as Person;
```

#### Creating custom activities

Activities can be created in 2 ways

1. Through designer -> Add new 'Activity' item to project -> create a normal workflow as activity -> use existing activities for logic
2. Through code -> Add new 'Code Activity' item to project -> write code for logic 

To the users of the activity, it does not matter whether it was created through designer or code, they behave just the same.

SO, any workflow you build generally on the designer with in-built or other activities, becomes custom activities themselves. Once the project builds successfully, they will appear as usable activities in the toolbox on the left hand pane of Visual Studio. You can drag and drop them in other activities and configure their arguments just like in-built activities.

For creating a `CodeActivity`, you write a class & derive from a suitable `Activity` base class. There are different options for the base class:

```fs
          |--> CodeActivity
          |
          |--> NativeActivity
Activity -|
          |--> AsyncCodeActivity   |--> CodeActivity<T>
          |                        |
          |--> Activity<TResult> --|--> NativeActivity<T>
                                   |
                                   |--> AsyncCodeActivity<T>
```

So, what are these options for? Which one to use?

1. `CodeActivity`: The simplest for implementing code login within `Execute()` method
2. `NativeActivity`: More low level access. Can be used to create _composite_ activities that can control child activities. Can also be usedto manage bookmarks
3. `AsyncCodeActivity`: Asynchronous pattern with Begin-End pattern. Override the `BeginExecute()` and `EndExecute()` methods. The begin method needs to return an `IAsyncResult`. In the end method, call `End()` on the `IAsyncResult` and get the required result. Asynchronous activities are specifically useful when they are used inside a `Parallel` type of flows.
4. `Activity<T>` types: Syntactic sugar for activities with single return value. Result property is of type `T`, which can bereturned from any of the three activity types above.

The **`ActivityContext`** class provides the interface to the runtime. It can be used to manage variables & arguments. In `NativeActivity`, they can also be used to schedule & bookmark. Different `Activity` types have their different derived context like `NativeActivityContext`, `CodeActivityContext`, `AsyncCodeActivityContext`.

**Arguments** of an activity are defined as properties of type `InArgument<T>`, `OutArgument<T>` etc. This enables the user to use expressions for those arguments (e.g. `products.Sum(p => p.Price)`) in the designer or code. If they are defined properties of type `T`, then primitive values can be used in designer (e.g. `100`, `"hello"`), but not expressions.

**Variables** are similarly defined as `Variable<T>`. They are mostly used in `NativeActivity`. These variables' state are managed by runtime. means, they can be persisted (say, on a DB) and can be brought back when needed.

To get a value of an argument or variable, you use the context. For example, to get the argument value `totalPrice` which is `InArgument<int>`, you use `totalPrice.Get(context)`. This is because, they variable may not be a direct `int`, but an expression that evaluates to an integer!

###### Code-declarative activities

The activities that you build declaratively, i.e. by composing existing components/activities (e.g. Sequence, While, Decision etc.), but do that by writing code rather than dragging-and-dropping tools on the designer. They derive from `Activity` base class. They can come in handy when building activities that are dynamic in nature, like `SomActivity<T>` where an input is `T`.

#### Validations

Adding metadata to the activity data (mostly arguments), so that the users use them correctly. Like, not missing an argument that is not-optional.

1. Override `CacheMetadata` - add error messages etc.
2. Use attributes on arguments - `RequiredAttribute`, `OverloadGroup`. RequiredAttribute makes an argument required. OverloadGroup comes in handly when you want to make one or more arguments required, among a set of possible options. In the example below, for the location of the user, either _street address, city & zip_ or _longitude & latitude_ are required. Notice that, individually they all need to be required, if they are required for that group. It's not mandatory to have all members of a group to be required. Also, you get validation error if you provide input argument from different groups!

```cs
[RequiredArgument]
[OverloadGroup("Address")]
public InArgument<string> StreetAddress { get; set; }
[RequiredArgument]
[OverloadGroup("Address")]
public InArgument<string> City { get; set; }
[RequiredArgument]
[OverloadGroup("Address")]
public InArgument<string> PostalCode { get; set; }

[RequiredArgument]
[OverloadGroup("Location")]
public InArgument<decimal> Longitude { get; set; }
[RequiredArgument]
[OverloadGroup("Location")]
public InArgument<decimal> Latitude { get; set; }
```

**Constraints** are powerful constructs (activities) that apply validation logic. They run at validation-time. Either apply them inside the constructor, or use `ActivityValidationServices` and `ValidationSettings` - useful for validating activity form the host(?). They can also be used to validate in-built or third-party activities, or when re-hosting the designer.

#### References

* https://app.pluralsight.com/player?course=wf4-fundamentals&author=matt-milner&name=wf4-introduction&clip=6&mode=live
* https://app.pluralsight.com/library/courses/wf4-custom-activities/table-of-contents