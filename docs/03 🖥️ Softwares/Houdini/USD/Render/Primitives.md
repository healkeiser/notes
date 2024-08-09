---
title: Primitives
icon: material/cube
slug: primitives
tags:
  - houdini
  - usd
  - render
publish: true
---

_How to use the [UsdRender](https://openusd.org/dev/api/usd_render_page_front.html) primitives in order to output an image._

## Relationship

In USD, all you need to render is a couple of primitives:
- RenderSettings
- RenderProduct
- RenderVar

Their relationship is described as:

```mermaid
erDiagram
    RenderSettings {
        relationship products "/Render/Products/renderproduct"
    }

    RenderProduct {
        relationship orderedVars "RenderVar A, RenderVar B, RenderVar C"
    }

    RenderVarA {
        string sourceName "Beauty"
    }

    RenderVarB {
        string sourceName "CombinedDiffuse"
    }

    RenderVarC {
        string sourceName "CombinedEmission"
    }

    RenderSettings ||--o| RenderProduct: "produces"
    RenderProduct ||--|| RenderVarA : "consumes"
    RenderProduct ||--|| RenderVarB : "consumes"
    RenderProduct ||--|| RenderVarC : "consumes"
```

Which translates this way into Houdini Solaris:

![](../../../../_attachments/houdini_5LFLMNdtXs.png)
> [!warning]
> **Do not** forget to modify the parameter of each primitive: add the RenderVars through **Ordered Render Vars** inside the RenderProduct, then add the RenderProduct to the RenderSettings through the **Ordered Products** parameter.

> [!tip]
> As described in [[Render Globals#Variants]], you can **order** the RenderVars. Taking Arnold as an example, where the **Beauty** is called **RGBA**, you'll use `/Render/Products/Vars/RGBA /Render/Products/Vars/**` inside the RenderProduct **Ordered Render Vars** parameter to have **RGBA** prepended to the RenderVars list, so switching to the Arnold Hydra delegate will immediately show the right RenderVar.

