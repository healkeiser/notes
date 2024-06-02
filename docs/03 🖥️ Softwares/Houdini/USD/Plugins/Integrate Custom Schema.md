---
title: Integrate Custom Schemas
icon: material/toy-brick
slug: integrate-custom-schemas
publish: true
---

_How to integrate a custom USD schema like a native Houdini primitive node._

> [!note]
> You can find the **Fxquinox Context Info** HDA on [GitHub](https://github.com/healkeiser/fxquinox/blob/main/plugins/houdini/otls/lop_fxquinox.context_info.hda).

## Native Node

Let’s take a look at the native **Render Settings** primitive node:

![[../../../../_attachments/DViGtQ2cWB.png]]

It contains a couple of important elements:

- A **Frame Range Sampling** menu (useful to get rid of time dependencies)
- The **Primitive Path** parm
- The **Action** menu, to either **Create** or **Edit** the primitive
- The **Initialize Parameters** menu, which lets you batch control the USD parms or retrieve another primitive attribute values (if in **Edit** mode)
- The **Create Primitives** menu, which allows you to control how and where the primitive will be created (if in **Create** mode)

> [!warning] 
> The **Primitive Path** parm internal name should be `primpattern`, so it can run the callback set on **Initialize Parameters**.

## Custom Node (HDA)

In order to emulate all those, we can create an HDA with the following structure:

> [!tip]- Lock Primitive Path
>  Here, **Primitive Path** is a locked parameter: you might want to add an **OnCreated** callback to lock it (or simply make the parm invisible) in order to force artists to only use pipeline defined primitive paths.

![[../../../../_attachments/zHNp9aWjDm.png]]

![[../../../../_attachments/houdini_X60V05qBjV.png]]

## Internal Nodes

### [Error](https://www.sidefx.com/docs/houdini/nodes/lop/error.html)

The error node contains the following code:

``` python
node = hou.pwd()
inputs = node.inputs()

if inputs and node.parent().parm("createprims").eval() == 1:
    prim = node.evalParm("../primpattern")
    stage = inputs[0].stage()
    return int(stage.GetPrimAtPath(prim).IsValid())

return 0
```

It will return this error to the user:

> An existing `chs("hda_label")` prim has been found at `chs("../primpattern")`. You can only have one in your current scene. For selective edits, use a `chs("hda_label")` Edit instead.

if another primitive **with the same path** is found, and you’re about to override it by **creating** a new one **on the** **same path**.

> [!tip]- HDA Label
> `hda_label` is a spare parameter created on the Error node with the value `Fxquinox Context Info`. This is added so you can quickly change it and use this HDA as a template.

> [!note] 
> You can set the message verbosity (**error**, **warning**, **info**) directly onto the node. In this specific instance, we don’t want to allow the creation of multiple **Fxquinox Context Info** primitives, so it’s set on **error**, which will effectively set the HDA in an error-state and fail its cooking. **Render Settings** just throws a warning.

### [Primitive](https://www.sidefx.com/docs/houdini/nodes/lop/primitive.html)

This is the node that creates the primitive itself. On this node you’ll choose the primitive type, primitive kind, parent primitive **type** (if applicable) and primitive specifier.

To apply your custom schema, select **Primitive Type** to your schema class. In this instance, **FxquinoxContextInfo**.

This node is also tied to the **Action** menu. If the mode is set to **Create**, it will enable the node, effectively creating a new primitive. Is it is set to **Edit** of **Force Edit** it will **not** create the primitive and bypass the node, immediately cooking the **Edit Properties from Node**. Here’s the Python expression driving it, set on this node **Activation** parameter (You can create it through a node Right-click > **LOP Action** > **Create Activation Parameter**):

``` python
create_prims = hou.pwd().parent().parm("createprims").eval() 
return 0 if create_prims in (0, 2) else 1
```

### [Edit Properties from Node](https://www.sidefx.com/docs/houdini/nodes/lop/editpropertiesfromnode.html)

This node is where the magic happens. It’s pretty much the same as the [Edit Properties](https://www.sidefx.com/docs/houdini/nodes/lop/editproperties.html) node, with a nice change:

> [!quote] 
> Instead of adding spare parameters to this node, you must direct it to another node from which it reads parameters that correspond to attributes on prims in the scene graph tree. When you edit these parameters on the other node, this node authors equivalent changes to the equivalent USD attributes.

That allows us to add the properties on the HDA itself: they will always be read from it and added accordingly.

This node is also linked to the **Frame Range Sampling** menu, **Primitive Path** parm, **Action** menu, and the **Initialize Parameters** menu. It’s the one that allows our HDA to behave just like a native node.

> [!note] 
> You can simply drag and drop the **Edit Properties from Node** node onto the **Type Properties** window of your HDA to automatically promote **all** its parameters onto your HDA, then simply delete the ones you don't need.

## Create USD Attributes

In order for the **Edit Properties from Node** node to work, we need to bring the USD schema attributes to the HDA interface. With a custom schema, this gets very easy. Open the HDA **Type Properties**, and inside the **Parameters** tab navigate to the **From USD** tab. You can now filter the schema you want and add all its attributes to your parameters.

![[../../../../_attachments/ZksJ49B28q.png]]

They will be automatically read by the **Edit Properties from Node** node!

## Add an Alias

On most of the native nodes, you’ll get two options: the node, and the node **edit**.

![[../../../../_attachments/TLcLb9mGWC.png]]

Under the hood, this is a simple tool alias, which sets the **Action** menu to **Edit**, and runs `loputils.setAllControlParameters(node, 'none')` to set all USD parameters to None.

Here’s how it’s done for **Render Settings**:

![[../../../../_attachments/houdini_0w1hgRxb5M.png]]

![[../../../../_attachments/houdini_ODFY580WFB.png]]

On our custom HDA, we can click on **Create New** and copy-paste the values of **Render Settings** in the **Options** tab. 

For the **Script** tab, we need some adjustments:

``` python
import loptoolutils, loputils

node = loptoolutils.genericTool(kwargs, '$HDA_NAME', '$HDA_NAME'.split("::")[1] + '_edit1')
node.parm('createprims').set('off')
node.parm('primpattern').lock(False)
loputils.setAllControlParameters(node, 'none')
```

As you can see, we need to split the name: `$HDA_NAME` will return `jf::jfcontextinfo`, and `:` is an unauthorized character for a Houdini node name. We simply isolate the second name component, and add the `edit_1` part to it.

![[../../../../_attachments/houdini_Y3xCYIrgLk.png]]

![[../../../../_attachments/houdini_Lk1cldxtXE.png]]

You can now try to drop the node, and should be welcomed by your HDA and its Edit variant:

![[../../../../_attachments/bhslDwCmk8.png]]

## Custom Icon

You can add an icon in the `$HOUDINI_PATH` following this naming convention:
- `SCENEGRAPH_primtype_<custom schema class>.svg`

To be displayed inside the Houdini **Scene Graph Tree** every time you apply your custom schema to a primitive:

![](../../../../_attachments/3Nd4p8emvV.png)

> [!example]- Icon Name 
> `SCENEGRAPH_primtype_fxquinoxcontextinfo.svg`
