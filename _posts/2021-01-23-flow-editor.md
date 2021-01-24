---
layout: post
title:  "Flow Editor"
date:   2021-01-23
author: Mike Blackstock
comments: true
---

# Introduction

Motivaton:
- have always been impressed with Node-RED editor, but didn't fully understand how it worked.  The essence of the editor was hard to see by just browsing the code.  I decided to build a 'toy' version of the editor to get a better idea of how it worked.
- wanted to learn D3
- experiment with some new (to me) tools such as the lerna package manager, parcel, and Typescript.  I had started to use these tools
at another job and wanted to understand them better.
- Typescript is a statically type superscript of Java.  It has helped companies building large Javascript applications find bugs https://slack.engineering/typescript-at-slack/
- package manager - wanted to have the front end module and backend server in the same repository
- needed Parcel web appliation bundler to transcode and bundle types.  It is supposed to be faster and simpler to configure than other tools like webpack or browserfy.
- learn about web components using a web component library called lit

## Starting point
- browsed node-red code
- found example for building a graph editor (reference)
- sketched out requirements:
 - pallette to drag nodes from
 - needed a way to define different node types - a registry
  - support single input connector, multiple outputs

 - pallette to drag nodes from
 - canvas to draw flow
 - draw links between nodes
 - select nodes
 - select links

 Things I would leave out
 - everything in the right side bar
 - multiple node selection
 - node icons
 - cut and paste
 - multiple flows (tabs)

## What I've done so far

The project is nearly at the stage where I could use it for the development
of simple flows for a suitable runtime like Node-RED's, or a new one written in another language like Python or Go.

The code is available at (github)

Talk about how the code works

How it is organized.

## Lessons learned

Typescript
- with VS code is awesome (how)
- knew right away that a field hadn't been declared because of a typo.
- Struggles - hard to get type write for d3 elements

Started writing pretty generic Javascript, and then adding types and other qualifications later such as private fields and methods.

Web components
- Encapsulation of CSS in web components is nice.  Don't worry about class name collisions (refs).
- Not sure about how I can effectively reuse CSS across components yet.
- ensures plug ins can only extend components in defined ways.  There's no way for an external component to manipulate the DOM inside another.

D3
- what did I learn there?

Lerna
- haven't used it yet.  I hope it will provide some payback as the project grows.

Parcel

What next
With a better understanding of how to build a flow editor I hope to be able to contribute more effectively to the Node-RED code base.
