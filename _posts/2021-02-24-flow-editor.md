---
layout: post
title:  "Flow Editor"
date:   2021-02-24
author: Mike Blackstock
comments: true
---

# Introduction

As I've mentioned in previous posts, I am a HUGE fan of Node-RED.  I've always been particularly impressed with the flow editor UI, but didn't fully understand how it worked and wanted to understand how it implemented such a great drag and drop UI for creating and editing flows.  

I tend to learn more by doing so I decided to build a 'toy' version of the editor using some of the tools, languages and libraries I've started to use recently.  This bit of code and write up is the restult of that.  Hopefully you find [this code](https://github.com/mblackstock/flow-editor) interesting and useful.

You can try it out here:

<iframe width="650" height="400" src="/demos/flow-editor">
</iframe>

As you can see, this editor is very limited.  My goal was to start with a (very) small subset of what the Node-RED editor can do using a flow format that is a subset of Node-REDs.

* Drag and drop different node types from the palette onto the canvas.
* Nodes may have different colours, a single input connector, and different numbers of output connectors.
* Move nodes around and connect nodes to each other with links.
* Select and delete nodes and links.

To get going, I spent time time browsing the Node-RED code and I searched for some sample code to understand how the [D3 visualization library](https://d3js.org/) works.  This [example of a directed graph editor in d3](http://bl.ocks.org/rkirsling/5001347) by [Ross Kirsling](https://github.com/rkirsling) proved very helpful.

In the rest of this post I'll outline the code base, how it works, then some lessons learned.

## Code

You can check out the code [on github](https://github.com/mblackstock/flow-editor).

The editor code lives in a [lerna monorepo](https://github.com/lerna/lerna) with the following directory structure (even though there is only one module the plan is to add additional modules over time).

```
.
├── lerna.json
├── package-lock.json
├── package.json
└── packages
    └── editor
        ├── README.md
        ├── dist
        ├── package-lock.json
        ├── package.json
        ├── src
        │   ├── config-element.ts
        │   ├── d3.ts
        │   ├── editor.ts
        │   ├── flow-element.ts
        │   ├── flow.ts
        │   ├── index.html
        │   ├── index.ts
        │   ├── pallette-element.ts
        │   ├── registry.ts
        │   └── styles.ts
        └── tsconfig.json
```

The code is written in Typescript.  While there are some benefits related to static typing and strong IDE support in VS Code, it does add an addition step of transpiling.  I didn't think this was a big deal given that I wanted to use a web application bundler and give Typescript a try.

I decided to use the [Parcel](https://parceljs.org/) web application bundler. To build the code ensure you have node installed, then:

```
cd packages/editor
npm start
```

Parcel will serve up the code at [http://localhost:1234](http://localhost:1234)

## How it works

Each of the areas in the editor page: the palette, the editor canvas and the configuration area is a separate web component created using the [LitElement](https://lit-element.polymer-project.org/) library.

>Note that in the following discussion, each source file has a link to the code in github.

The [`index.html`](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/index.html) file contains the web page `<div id="demo">` placeholder for the web components that are added dynamically to this element in `index.ts`.  There I define the three web components: `palette-element`, `flow-element` and `config-element`.  The `index.ts` file also contains some test data: three node types added to the registry, and a simple flow to start with.

It then dynamically renders the contents of the div with the three web components using [lit-html](https://lit-html.polymer-project.org/), setting event handlers and properties in the render call:

```javascript
render(
  html`
    <palette-element .registry=${registry} dropSelector="flow-element"
      @dropNode=${(e: DropEvent) => flowElement.dropNode(e.detail.type, e.detail.x, e.detail.y)}></palette-element>
    <flow-element .flow=${testFlow} .registry=${registry}
      @configNode=${(e: NodeEvent) => console.log(`config ${e.detail.id}`)}
      @selectNode=${e => console.log(`select ${e.detail.id}`)}></flow-element>
    <config-element .registry=${registry}></config-element>
    `,
  document.querySelector('#demo'));
```

Here, we use our three new web components on the web page.  The palette needs a node registry, a selector for the drop target, and send drop node events when an element is dropped into the drop target.

When a `dropNode` event is received from the palette, we call `dropNode` on the flow-element component with the information needed to add it to the flow: type and position.

The `flow-element` is set up with an initial flow, and the node registry.  It can trigger two events: `@selectNode`, when a node is selected (single click), and `@configNode`, when a node is to be configured in the flow (double click).  Currently these events just output to the console.  The `config-element` is just a placeholder for a future node configuration UI.

The node registry [`registry.ts`](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/registry.ts) is a wrapper on a map of node types to node descriptions.

## The palette component

The palette web component [`palette-element.ts`](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/palette-element.ts) contains the registered nodes, and supports dragging nodes into an element.  The component uses mouse events to implement drag and drop as described [here](https://javascript.info/mouse-drag-and-drop).

The component `render` method renders the various node types in the Registry.  We add our `mousedown` event handler to each node.

### Mousedown event handler

In the `mousedown` event we clone the html element so we can move it during the drag. We then add a `mousemove` event to the document for the drag and a `mouseup` event to the cloned node to end the drag.

In the `mousemove` event handler, we change the location of the node.  We then get the element that is under the node.  If the element that is under the dragged node is in our assigned droppable element (typically the flow-editor web component) we save the element in case we lift the mouse up.

On `mouseup`, we remove the mousemove listener, we remove the cloned node.  If we were dropped in the flow-editor (`this.droppable`) we fire a custom `dropNode` event with the type and location of the node so it can be added to the flow-editor.

## The flow component

The flow editor web component [`flow-element.ts`](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/flow-element.ts) is a wrapper on the FlowEditor class in [editor.ts](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/editor.ts) where most of the work is done.

The Editor class has three public methods: `init`, `updateFlow`, and `addNode`.  The `init` method is called to set things up with the flow.  The updateFlow method is called when the flow needs to be redrawn.  The `addNode` method is called when a new node is added; it then calls `updateFlow`.

The `init` method creates groups for nodes and wires, adds a line to draw links between nodes, and sets up event handlers for dragging the link, and handling the delete key for links and nodes.

The `updateFlow` method called after init to draw the current flow.  It generates a list of wires from the flow to bind to the wires.  It draws the nodes, connectors, adds click and drag behaviours for the nodes.  It then draws the wires, adding click behaviour to select wires.

The click behaviour can detect a single click or a double click using timers.  When dragging nodes, we not only need to move the nodes but change the start and end position of the attached wires.

If you have any questions or see any bugs in the code, feel free to drop me a line, or leave a comment.

## Lessons learned

This has been a good project to get exposure to a number of technologies.  I found Typescript to be a pleasure to use with VS code.  I started with fairly generic Javascript and then 'tightened' things up by adding my own types.  I feel like I haven't benefited from types much given this is a small project, but overall a good experience.

Web components using `lit-element` was pretty easy to use.  I like how Web Components encapsulate CSS, so I don't need to worry about class name collisions, or having styles used in one component affect another.  At some point I need to look at the CSS to see what I can reuse between components and move that to the shared [`styles.ts`](https://github.com/mblackstock/flow-editor/blob/master/packages/editor/src/styles.ts) file.

I learned a lot about how D3 works by creating this simple editor.  Flows seem to map naturally to nodes since they are arrays of nodes, but I needed to create a new array of wires from the flow to bind the links to.  Even handlers for dragging and clicking got a little complex.

## What's next?

My plan is to spend more time browsing the Node-RED source code to compare and contrast the editor code with what I've done.  Stay tuned for updates.  From this exercise I've definitely gained a greater appreciation for the tremendous work done in creating Node-RED; I hope to be able to contribute more effectively to the Node-RED code base. I may build on this a bit more by adding a simple data flow runtime to the project.

I'd appreciate any comments or questions.  Feel free to fork [the code available on github](https://github.com/mblackstock/flow-editor) for your own projects, or cut and paste as you like.




