# Introduction
Usually, when triggering analytics events, not much attention and care is given to it. Commonly the usual solution is to declare classes per event or even use a simple solution like mapping attributes and values with maps and not take the advantage of a strongly typed system.

The problem becomes worse when you have different clients, written in different programming languages. Then you have to repeat the definition of the events for every project and the chances of having inconsistencies are even bigger. For example, you can have events with different names per platform or even attributes changing slightly over platforms.

## Overview
The purpose of this specification is to describe a useful structure to define analytics events taking the advantage of an easy-to-read object annotation, like YAML.

You can use this structure as an input to generate code and to hide implementation details for different platforms, you can also use it as documentation for all analytics events since YAML is easier to read than any regular programming language.

## Goals
1. Save time by centralizing the place to define analytics events and not repeat them for every client
2. Provide a structure that is easy to be parsed and to generate code for different platforms
3. Provide documentation and source-of-truth for analytics events