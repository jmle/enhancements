---
title: Gradle support
authors:
  - "@jmle"
reviewers:
  - "@pranavgaikwad"
  - "@rromannissen"
  - "@shawn-hurley"
approvers:
  - "@pranavgaikwad"
  - "@rromannissen"
  - "@shawn-hurley"
creation-date: 2024-05-10
last-updated: 2024-05-10
status: implementable
---

# Annotation inspection

This enhancement presents the possibility of matching on methods, fields or class declarations with specific annotations in the Java provider.


## Summary

Sometimes it is desirable to create a rule to match on an attribute of a class of a given type, only if it is annotated with a given annotation. For instance, when migrating from JMS to Quarkus, to detect the following `Queue` annotated with `@Resource`:
```java
@Resource(lookup = "java:/queue/HELLOWORLDMDBQueue")
private Queue queue;
```

Ideally it should also be possible to match on the field with the annotation *including a given attribute and/or value*, this is, to be able to further restrict the match to *only if the `lookup` field is present and/or has a given literal value*.

See the [Windup rule development guide](https://access.redhat.com/documentation/en-us/migration_toolkit_for_applications/5.2/html-single/rules_development_guide/index#javaclass_child_elements) for more information.

## Motivation

- Matching more accurately enables the provider to give better hints and recommendations.
- The existing rulesets inherited from Windup contain rules with these characteristics, whose value is lost otherwise.

### Goals

- The ability to match on a class declaration, method, function or class field annotated with a specific annotation. Ideally, to match on any symbol which can be annotated.
- The ability to match *only if* that annotation has some given properties. 
- **Potentially** the ability to extract values from those annotations as variables to be later interpolated.

### Non-Goals

N/A

## Proposal

The main usecase would consist of the user being able to specify an additional field in the `java.referenced` entry with the information to be matched.

### User Stories

- As a user, I want to be able to create a `java.referenced` condition which can match on a given type, with an "annotable" `location`, which is annotated with a specific annotation, so that I can further narrow down my rule.
- As a user, I want to get an error if the `location` on which I am trying to find an annotation is not annotable.
- As a user, I want to be able to convert my old Windup XML rules containing annotation inspection to the new Konveyor YAML format.
- **Potential** As a user, I want to be able to extract, in the form of interpolatable variables, the information from this annotation, so that I can use it in my rule message.

### Implementation Details/Notes/Constraints

#### New condition field
There would be a new YAML field called `annotated` in the `java.referenced` condition. For instance:

```yaml
  when:
    java.referenced:
      location: FIELD_DECLARATION
      pattern: java.util.Calendar
      annotated:
        pattern: org.package.MyAnnotation
```
Which would allow us to match on the following:
```java
@Myannotation
private Calendar calendar;
```

#### Further match on annotation properties
Ideally it would be desirable to be able to match on annotation properties, officially called [elements](https://docs.oracle.com/javase/tutorial/java/annotations/basics.html).
```yaml
  when:
    java.referenced:
      location: FIELD_DECLARATION
      pattern: java.util.Calendar
      annotated:
        pattern: org.package.MyAnnotation
        elements:
          pattern: element
          value: value
```
Which would allow us to match on the following:
```java
@MyAnnotation(element = "value")
private Calendar calendar;
```

#### XML rule conversion
The windup-shim project must be modified accordingly to be able to transform old Windup rules into this new format.

#### Konveyor rule update
Our generated rulesets must be updated to include this information from old Windup rules, which is currently not present in any form in our rulesets.



### Security, Risks, and Mitigations
N/A

## Design Details

### Test Plan
- The rules can be tested using the windup-shim utility and ensuring there are no false positives.

## Implementation History
TBD
