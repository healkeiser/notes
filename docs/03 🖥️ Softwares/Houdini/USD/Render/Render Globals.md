---
title: Render Globals
icon: material/image-size-select-small
slug: render-globals
tags:
  - houdini
  - usd
  - render
publish: true
---

_How to create render globals in USD, to use throughout a production._

## Goal

In a production environment, you might want to give artists a base of render settings they can use pretty much anywhere. In Solaris, we can take advantage of the variant system, and author a set of variants applied on the `/Render` primitive to dictate which variant (preset) will be used.

Ideally, you'd publish different global presets against a specific pipeline step: one for lookdev, one for lighting, etc. 

> [!note]
> Even with publishing different globals against their own pipeline step, they could still inherit from the same "master" global which could contain show resolution, aspect ratio, render engine, etc.

We could also have have multiple RenderSettings pointing to multiple RenderProducts, but I like the idea of having a single standardized primitive which always resides in the same location: it just makes your life easier.

## Setup

In Solaris, the setup would look like this: 

![](../../../../_attachments/houdini_Y3wS1c5Cwr.png)
You can see there are two "main" variant branches: **beauty** and **technical**. This is a personal production choice, where a technical pass would contain the same renderable objects as the beauty pass, but without lighting, a very low sample count and what's defined as technical AOVs: **depth**, **P**, **N**, etc.

Let's now dive into deeper into each section of the above graph:
### AOVs

Here's the AOV part of the setup, where we define our RenderVar primitives:

![](../../../../_attachments/houdini_kTjFauu0IH.png)

Each **Karma Standard Render Vars** node define RenderVars for both the **beauty** and **technical** branch. The trick is found in the **Configure Primitive** and **Edit Properties** nodes: We select the freshy created RenderVar primitives, and we apply a [custom API schema](../Plugins/Custom%20Schemas.md) to them. This adds a new set of attributes on the primitives, in this instance a `beacon:renderVarType` token, which can be set on **beauty**, **technical**, **light**, **crypto** and **extra**.

> [!question]
> Why the hell do we do this? Well, appplying an extra attribute to the RenderVar primitives allows you to group them neatly. In Houdini, you can then use a primitive pattern to only select the ones you want: `/Render/Products/Vars/* & %type(RenderVar) & {usd_attrib(0, @primpath, "beacon:renderVarType") == "beauty"}` will only iterate over the RenderVars you have defined as **beauty** AOVs.

> [!note]
> You don't need to add a custom API schema, a simple **Edit Properties** on which you add a token parameter will be enough. I created the API schema since it's faster to re-use and less prone to human error.

Here's what the node parameters look like for the **technical** side:

![](../../../../_attachments/houdini_CZaD48TUIz.png)

![](../../../../_attachments/houdini_DAKE2LFwfN.png)

### Base

We can now define our main RenderProduct and RenderSettings:

![](../../../../_attachments/houdini_9wYiAGMLSr.png)

You'll define here the main settings, from which we'll then branch out for our variants: resolution, samples, etc.

> [!note]
> Notice how add all RenderVars to the RenderProduct? You can do it or not, it doesn't really matter as we'll refine the AOV selection for each variant.

### Variants

Here comes the heavy part: setting up the variants for each render preset we want.

![](../../../../_attachments/houdini_EbHPOnuVH5.png)

We'll be working from the **beauty** branch for the following.

Let's start by the **Configure Primitive** nodes: each takes the AOVs **that we don't want** in a branch, and deactivate them. This is a neat way to have a visual feedback when looking at the Scene Graph Tree that some AOVs will be added to the RenderProduct, and others won't:

![](../../../../_attachments/houdini_pz44VAdT28.png)

>[!tip]
> Deactivating primitives also gives you the ability to filter them later on using a primitive pattern: `/Render/Products/Vars/** & {usd_isactive(0, @primpath)}` will only iterate over activated RenderVar primitives.

> [!note]
> You can also enable the **Hide Primitive In Scene Graph** tree to hide them altogether.

We then edit the RenderProduct primitive added in the [base](#Base) section with a **Render Product Edit** node, adding only the AOVs we want for the selected branch:

![](../../../../_attachments/houdini_v2HyT8QQq6.png)

> [!tip]
> You'll notice that we have some sort of a repetition in the primitive pattern: `/Render/Products/Vars/Beauty /Render/Products/Vars/**`. Since RenderVars are added by alphabetical order, you want the **Beauty** to come first, and not something like **Albedo** or **AO**. Otherwise, the Scene Viewer will show the wrong RenderVar first when switching to another Hydra delegate than Houdini GL (or VK).

We can now create our **Render Settings Edit**, and start editing the render engine settings based on the presets you want. In my setup, I have **BTY_high**, **BTY_mid** and **BTY_low**. You can see that each of them are connected to the previous one, that is not mandatory: I'm doing it so I can simply divide the attribute values of the **high** variant into the lower-quality ones.

Once you have your presets saved, simply plug them inside the **Add Variant** second input, set the variant set and variant names, and voila!

## Final

If everything went well, the `/Render` primitive now comes with a bunch of new variants (it should be green in the Scene Graph Tree).

To check that everything worked, drop a new **Set Variant** node and flick through your variants. Here's the **BTY_high** variant:

![](../../../../_attachments/houdini_RPPTvjOhYD.png)

And the **UTL_high** one:

![](../../../../_attachments/houdini_jgQKiFIcNY.png)

Notice how the RenderVars primitives get disabled based on the the variant choice? That's a good sign that your variants are ready. You should also check your RenderSettings and RenderProduct primitive attributes to see if everything went well.

Once you're done, simply export the graph as a USD file, and use import it back as a Sublayer.

> [!tip]
> You might want to add the USD sublayer fairly high in the graph, so you can edit its properties: if you use it in a shot environment, you will maybe need to specify a camera using a **Render Settings Edit** node, which will need to point to an existing RenderSettings primitive.

> [!quote] Resources
> The asset used in the above screenshots is [Golden Knight](https://www.artstation.com/artwork/o2L6ZB) from [Francis Lamoureux](https://www.artstation.com/francislamoureux), downloaded on [Sketchfab](https://sketchfab.com/3d-models/golden-knight-06555f9ebaa640d8a50f356ffb67e8c9).