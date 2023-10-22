> [!quote]- Origin
>  [Making of an Alias in Houdini](https://www.artstation.com/blogs/sidefxlabs/VMvb/making-of-an-alias-in-houdini)

_Compilable Segments_ was recently introduced to Labs as a quick way to segment out geometry and to get started with a compiled process very quickly. It's not a tool per se, but something we call an alias which is another way to help speed up a workflow by configuring a node or a set of nodes into a preset of sorts. In this tutorial, we will break down how the  alias was made, but first...

# **What is an Alias?**

First thing's first, what exactly is an alias in terms of Houdini? An alias is an alternate name given to a node with specific parameter presets set by default, or a set of nodes chained together in a specific way. For example, _Tangent_ is an alias for the _PolyFrame_ SOP with the parameters configured to calculate tangents in MikkT space.

![](https://cdnb.artstation.com/p/media_assets/images/images/001/149/497/large/tangent.jpg?1693591996)

Another example of an alias are the different For Each Blocks, which is configured with a For Each Begin wired into a For Each End with the settings configured differently depending on which type of block you chose. You can access aliases through the **Tab** menu as with all other Houdini nodes.

# **A Look at the Compilable Segments Alias**

With the recently released _Connectivity and Segmentation_ tool in Labs, we have also released a helpful alias for the tool called _Compilable Segments_, which is a quick setup for multithreading geometry processes. _Connectivity and Segmentation_ was created specifically to make multithreading geometry easier, so we wanted to provide a quick way to get started with multithreading using _Compile Blocks_. 

The alias drops down a _Connectivity and Segmentation_ node with a _For-Loop_ inside of a _Compile Block_ with all the settings configured to multithread processes inside of the _For-Loop,_ as shown below.

![](https://cdna.artstation.com/p/media_assets/images/images/001/149/516/medium/connectivityandseg.jpg?1693593701)

# **Role of Shelf Tools**

An alias is essentially a hidden shelf tool. Remember, you can make a shelf tool automatically by dragging and dropping a sequence of nodes onto a shelf, and it will create a shelf tool that you can use again by either clicking the shelf tool or by using the **Tab** menu to drop down the shelf tool name.

You can access the Python code of the created tool by right clicking the shelf tool, left clicking on **Edit Tool**, then navigating to the **Script** tab to see the Python code. The code generated for an automatic tool may work well for certain cases, it is not very readable or easy to edit. Below is the script of an automatic shelf tool of a _Sphere_ node wired into a _Copy to Points_ node, and you can see how unreadable the script is:

![](https://cdnb.artstation.com/p/media_assets/images/images/001/149/517/large/unreadable.jpg?1693593724)

When the same exact tool can be made by writing a few lines of Python:

![](https://cdna.artstation.com/p/media_assets/images/images/001/149/518/large/readable.jpg?1693593736)

For Labs, we make aliases by manually scripting a shelf tool this way, or by directly modifying a `.shelf` tool that contains the tool. We save our aliases to the `alias.shelf` file if you are curious to see how we accomplish that.

# **Let's Make an Alias**

Before we begin writing the code for an alias, we'll get the node layout assembled first, complete with custom-set parameters. The network layout for _Compilable Segments_ is shown in the second image above. This tutorial is a detailed, step-by-step explanation for a moderately complicated alias with multiple nodes and multiple parameter adjustments. Simple aliases such as renaming nodes can be done by editing the `.shelf` file directly.

1. To get started on a new alias, right click on a shelf in Houdini and select **New Tool.** !
2. From this window, you can set the **Name**, **Label**, and **Icon** of the tool. You can also choose which `.shelf` file to **Save To**. This window can be accessed again later by right clicking the shelf tool and left clicking on **Edit Tool.**
3. From there, navigate to the **Context** tab to set which context this alias will be used in. It will most likely be the **SOP** context.
4. Now we are ready to start scripting our alias. Navigate to the **Script** tab to begin. Here is the complete code for the _Compilable Segments_, and we will break down the different pieces of code further.
	 - **Node Initialization:** In lines 1 - 14, we create all the nodes that we need, and we create a list to easily loop over them later on. Python functions used in Houdini are documented in SideFX’s documentation, but we will touch on a few specific functions. 
	 - The `soptoolutils.genericTool()` function allows you to place the tool from the **Tab** menu as expected, by moving your cursor in the viewport and clicking to set the tool down. This is done for the _Connectivity and Segmentation_ node only, and the rest of the nodes needed for the network are created automatically in the parent node (current geometry node) afterwards with `createNode()`.
	 - An easy way to find the node label needed to call in these functions is by middle mouse clicking or opening the node info window to find the label. For instance, the _Compile End_ label is  `compile_end`.
	 - **Setting Inputs**: The nodes are now created, but they have no relationship to each other yet. The list of nodes is populated in the specific order they should be connected in, so a for loop runs over the list starting from the second element (index 1) and sets the current node’s input to the previous node’s output with the `setInput()` function.
	 - **Renaming Nodes**: We decided to rename a few of the nodes for several reasons. The alias uses our _Connectivity and Segmentation_ node, but we wanted the name of the placed node to reflect the name of the alias, so the name was changed to “compilable_segments”. Also, when constructing a _For-Loop_ from scratch, it starts with a _Block Begin_ and ends with a _Block End_ node, but we wanted the names to reflect the _For-Loop_ structure that users are accustomed to seeing. These nodes had their names changed with the `setName()` function. 
	 - **Setting Custom Parameters**: Nodes are created with their default parameters, so they must be configured to work seamlessly together. With this example, _Block Begin_ and _Block End_ in the _For-Loop_ must reference each others’ names with the **blockpath** parameters, and the _Block Begin Compile_ node must reference the name of the _Block End Compile_ node. We do this by getting the names of the necessary nodes with the **name()** function and setting the **blockpath** parameter values in lines 26, 27, 29, and 30 with the `setParms()` function. 
	 - Other parameters set include:   
		 - Line 27: setting the **Method (method)** parameter on the _Block Begin_ node of the _For-Loop_ to **Fetch Input** (value of 1)   
		 - Line 28: setting the **Iteration Method (itermethod)** to **By Pieces or Points** (value of 1), the **Gather Method (method)** to **Merge Each Iteration** (value of 1), the **Piece Elements (class)** to **Primitives** (value of 0), and the **Piece Attribute (attrib)** to **segment** to reference the **segment** primitive attribute output of _Connectivity and Segmentation_
		 - Line 31: enabling the **Multithread when Compiled** toggle on the _Block End_ of the _For-Loop_
	 - **Finishing Touches**: To keep the visual elements of the workflow familiar, we coloured the _Compile Blocks_ their distinct yellow colour using the `setColor()` function. Secondly, the nodes that are created at the origin in the Network view unless otherwise specified, so the `layoutChildren()` function was used on our node list to evenly align them from our initially placed node. We moved the two _End Blocks_ down a bit further to create a space between the _For-Loop_ blocks to make it easier for users to start inserting their own nodes into the workflow.
	 - `Optional`. One last step is transferring aliases manually between `.shelf` files. If you navigate to the `.shelf` file the toolbar is saved to, you will see that the the tool’s code exists within. This will usually be the `default.shelf` file, but you can create multiple `.shelf` files to use. Below is a part of the code as seen in the `alias.shelf` file. You can copy the code starting from  and ending with and paste this within another `.shelf` file.  ![](https://cdna.artstation.com/p/media_assets/images/images/001/149/580/large/5.jpg?1693596573)

Tadam!

---
### `alias.shelf` Code

```python
import hou
import soptoolutils


# The main node to place when dropping down this alias
node = soptoolutils.genericTool(
    kwargs, "labs::connectivity_and_segmentation::1.0", exact_node_type=False
)

# Gets the parent of the placed node to places sequential nodes in the series
parent_node = node.parent()

# Sequential nodes initialization
compile_begin = parent_node.createNode("compile_begin")
for_each_begin = parent_node.createNode("block_begin")
for_each_end = parent_node.createNode("block_end")
compile_end = parent_node.createNode("compile_end")
node_list = [node, compile_begin, for_each_begin, for_each_end, compile_end]

# Sets node inputs based on the node list order above
for i in range(1, 5):
    node_list[i].setInput(0, node_list[i - 1], 0)

# Names first node to alias name and set for each nodes to familiar names
node.setName("compilable_segments", True)
for_each_begin.setName("foreach_beginl", True)
for_each_end.setName("foreach_end1", True)

# Sets default parameters
compile_begin.setParms({"blockpath": "../" + compile_end.name()})
for_each_begin.setParms({"blockpath": "../" + for_each_end.name(), "method": 1})
for_each_end.setParms(
    {"itermethod": 1, "method": 1, "class": 0, "attrib": "segment"}
)
for_each_end.setParms({"blockpath": "../" + for_each_begin.name()})
for_each_end.setParms({"templatepath": "../" + for_each_begin.name()})
for_each_end.setParms({"multithread": 1})

# Sets compile blocks to familiar yellow color
compile_begin.setColor(hou.Color(0.75, 0.75, 0))
compile_end.setColor(hou.Color(0.75, 0.75, 0))

# Layouts nodes properly, giving the for each block more space for work area
parent_node.layoutChildren(node_list)
for_each_end.setPosition(for_each_begin.position() + hou.Vector2(0, -2))
compile_end.setPosition(for_each_end.position() + hou.Vector2(0, -1))
```

> [!note] Note 
>  If you want to take a tool off of a toolbar, only do so with the **Remove from Shelf** option. If you select the delete option, it will delete the code within the `.shelf` file. Keeping your aliases within a separate `.shelf` file will prevent accidentally deleting an alias. An alias or tool that is removed can still be accessed with the **Tab** menu or added again to a shelf, but an alias that is deleted is gone.

> [!info] Info 
>  You can test the alias while working on it by dropping it down from the **Tab** menu into your *Network* view!
