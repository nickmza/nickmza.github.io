---
title: Testable Architecture with ArchUnit
tags: design
excerpt: >-
  In this post I'm discussing how to test software designs using ArchUnit as
  well as why you might want to in the first place.
ogimage: test-arch2.png
date: 2023-10-27 10:27:17
---


In the ever-evolving landscape of software development, maintaining a well-structured and resilient codebase is paramount. [ArchUnit](https://www.archunit.org/) is a framework that allows you to test your design using unit tests. Each aspect of the design can be expressed as unit tests or even derived from a UML diagram. The application is then tested to see if its structure conforms to the specified design.

ArchUnit empowers developers and architects to impose strict rules governing various aspects of software architecture. These rules encompass package dependencies, class dependencies, containment, inheritance checks, annotations, layering, and the prevention of circular call dependencies. But before diving into the details of how ArchUnit works, it's essential to explore why it is crucial in the first place.

# Why Architectural Testing Matters

Misunderstandings in design documents and whiteboard diagrams are not uncommon, and their consequences can vary from glaring errors to subtle deviations that take time to surface. By documenting your design using ArchUnit and subjecting your code to these tests, you ensure that your codebase aligns precisely with your architectural vision.

Architectural testing acts as an early warning system, instantly detecting any changes that violate your design. For instance, consider a scenario where a new team member accidentally bypasses the Data Layer, disables security on a controller, or introduces an unintended dependency. Many teams heavily rely on code reviews to identify such issues, but this approach can delay feedback and potentially lead to significant rework. In contrast, architecture testing serves as an immediate safeguard, allowing code reviews to focus on higher-level concerns.

> Embarrassing anecdote -> I once accidentally disabled security on one of my applications because I removed the security attribute from the controller whilst debugging and forgot to put it back. A simple ArchUnit test would have immediately spotted my error.

Software is inherently dynamic, and designs seldom survive unscathed in the real world. They adapt in response to evolving requirements and a deeper understanding of the problem domain. Architectural testing offers traceability at the source code level, enabling you to understand when and why changes occur. Additionally, it simplifies the assessment of the impact of modifications. For example, if you plan to introduce a Facade pattern over multiple classes, you can begin by modifying the architecture tests. This action will provide an inventory of all the classes and references that require adjustment to accommodate the new design.

# Implementing ArchNote

I am going to add ArchNote to one of my SpringBoot Camel projects. For now I want to implement a couple of simple rules:
- I want all of my Routes to be contained in the 'routes' package. 
- I want all of my Routes to inherit from the RouteBuilderBase class.

## Setup
Start by adding the ArchUnit dependency. 
```
 testImplementation 'com.tngtech.archunit:archunit-junit5:1.1.0'
```

## Creating Architecture Tests
Here's the first test. We import the application's packages and then create a rule that ensures that all of my Routes are placed in the 'routes' package.

{% codeblock lang:java %}

    @Test
    void EnsureRoutesAreInCorrectPackage(){
        JavaClasses jc = new ClassFileImporter()
                .importPackages("mcb.camel.template");

        ArchRule r1 = classes()
                .that().areAssignableTo(RouteBuilder.class)
                .should()
                .resideInAPackage("..routes");

        r1.check(jc);
    }

{% endcodeblock %}

The next test ensures that everything in the 'routes' package must inherit from RouteBuilderBase. 

{% codeblock lang:java %}
    @Test
    void EnsureRoutesAreInRoutesUseCorrectBase(){
        JavaClasses jc = new ClassFileImporter()
                .importPackages("mcb.camel.template");

        ArchRule r1 = classes()
                .that().resideInAPackage("..routes..")
                .should()
                .beAssignableTo(RouteBuilderBase.class);

        r1.check(jc);
    }
{% endcodeblock %}

When the tests are run ArchUnit gives a listing of the rules that are broken and the offending classes. For example if I change the base class of the GetAccountsRoute:

```
java.lang.AssertionError: 
Architecture Violation [Priority: MEDIUM] - Rule 'classes that reside in a package '..routes..' should be assignable to mcb.camel.template.routes.RouteBuilderBase' was violated (1 times):
Class <mcb.camel.template.routes.GetAccountsRoute> is not assignable to mcb.camel.template.routes.RouteBuilderBase in (GetAccountsRoute.java:0)
```
 
## A picture is work a thousand asserts

As an alternative to describing your rules using code ArchUnit allows you to describe your rules using UML via PlantUML. Here's a simple component diagram for my solution.

> [PlantUML is a versatile component that enables swift and straightforward diagram creation. Users can draft a variety of diagrams using a simple and intuitive language.](https://plantuml.com/)

<img src="test-arch1.png"/>

This diagram is generated from the markup below.

```

@startuml

[Configuration] <<..mcb.camel.template.config..>>
[Routes] <<..mcb.camel.template.routes..>>
[Models] <<..mcb.camel.template.models..>>
[Filters] <<..mcb.camel.template.filters..>>
[Application] <<mcb.camel.template.CamelMqApplication>>
[Tests] <<mcb.camel.template.tests..>>

[Routes] -> [Configuration]
[Routes] -> [Models]
[Routes] -> [Filters]

[Filters] -> [Models]

[Tests] -> [Routes]

@enduml

```

In order to test your code against the diagram you simply load the diagram from a resource and add it as an assertion to the ArchRule. The 'consideringOnlyDependenciesInDiagram' parameter prevents ArchUnit from flagging dependencies not specified in the diagram.

{% codeblock lang:java %}
    @Test
    void EnsureClassesConformToDesign()
    {
        JavaClasses jc = new ClassFileImporter()
                .importPackages("mcb.camel.template");

        URL myDiagram = getClass().getClassLoader().getResource("architecture.puml");

        ArchRule r1 = classes().should(adhereToPlantUmlDiagram(myDiagram, consideringOnlyDependenciesInDiagram()));

        r1.check(jc);
    }

{% endcodeblock %}

# Next Steps
I can see how ArchUnit could fit into our development process. I can also see that it could be grossly misused in a misguided attempt to get a team to conform to someone else's vision. My suggestion would be an iterative approach focussing on the core principles of the design and specific constraints. You could start by reviewing comments in your Pull Requests and see if any of these could be modelled using ArchUnit. I'm curious as to whether or not I could generate UML from an existing application and then use this as the basis of ArchNet rules. Will try that next.
