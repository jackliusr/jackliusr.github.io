---
title: Mermaid flow with subgraph control
date: 2024-08-20T20:10:00+08:00
categories:
- tech
tags:
- diagram
- Mermaid
---

One thing constantly bothered me since I used mermaid for flowchart, how to correctly group elements in a group. Only today I have a thought about what will happen if I put node in subgraph without links between them. Finally I got it works to correctly group elements in a group. 

Another things, mermaid diagram is natively supported in asciidoctor now. This one is my first example to use it. 

```mermaid
%%{init: {"flowchart": {"htmlLabels": false}} }%%
flowchart TD
   subgraph developer
     cls(Entity Classes annotated with AttrA)
   end
   subgraph fhirnexus
      cg(Source Generator):::changes
      task( msbuild custom task):::changes
   end
   subgraph generated artifacts
    clsF(Custom FHIR Resource class)
    sd(StructureDefinition)
    capStmt(jsonpatch Fragment and \n patched some files)
   end

   cls(Entity Classes annotated with CustomFhirResource) --input---> cg(Source Generator):::changes
   cls --input---> task( msbuild custom task):::changes
   cg -- real time auto-gen --> clsF(Custom FHIR Resource class)
   task -- build time  auto-gen  --> sd(StructureDefinition)
   task -- build time auto-gen  --> capStmt(jsonpatch Fragment and \n patched some files)
   classDef changes fill:orange
```

