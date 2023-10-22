> [!quote]- Origin
>  [Define a geometry node (SOP) using Python](https://www.sidefx.com/docs/houdini/hom/pythonsop.html#external_source)

# Overview

- If you want to define a new, reusable geometry node using Python, see “creating a new node type” below.
- If you just want to script a one-off change, you can use the [Python SOP](https://www.sidefx.com/docs/houdini/nodes/sop/python.html "Runs a Python snippet to modify the incoming geometry.") to run a Python snippet on geometry without having to create a new node type.

# Creating a new node type

1. Choose **File ▸ New Asset**.
3. Set the **Operator Definition** to `python`, then set **Network type** to **Geometry**.
4. Use the **Save to library** option to set an OTL library file to save the new node type into.
5. Click **Accept**.
	The [type properties window](https://www.sidefx.com/docs/houdini/ref/windows/optype.html "The type properties window lets you edit the metadata and parameter interface of a digital asset node type.") appears.
1. Use the [options in the type properties window](https://www.sidefx.com/docs/houdini/ref/windows/optype.html "The type properties window lets you edit the metadata and parameter interface of a digital asset node type.") to define the interface for your new node type.
2. Click the **Code** tab to view and edit the Python script that defines the SOP’s behaviour.

> [!tip] Tip
>  If you need to edit the script after closing the type properties window, right-click an instance of the node and choose **Type properties**.

# Writing the code

To get the node’s incoming geometry, use:

```python
geo = hou.pwd().geometry()
```

The `hou.pwd()` function returns the currently cooking [Node](https://www.sidefx.com/docs/houdini/hom/hou/Node.html "The base class for all nodes in Houdini (objects, SOPs, COPs, etc.)  An instance of this class corresponds to exactly one instance of a node in Houdini."), and the `geometry()` method returns a [hou.Geometry](https://www.sidefx.com/docs/houdini/hom/hou/Geometry.html "A Geometry object contains the points and primitives that define a 3D geometric shape.  For example, each SOP node in Houdini generates a single Geometry object.") object. If an input SOP is connected to the first input of the Python SOP, Houdini copies the input SOP’s geometry into the Python SOP’s geometry before running the Python code.

> [!tip] Tip
> To get the geometry from an input other than the first, use the `inputs()` method of [Node](https://www.sidefx.com/docs/houdini/hom/hou/Node.html "The base class for all nodes in Houdini (objects, SOPs, COPs, etc.)  An instance of this class corresponds to exactly one instance of a node in Houdini.") to get the input nodes, and then the `geometry()` method of one of those nodes to get its geometry.

```python
this_node = hou.pwd()
inputs = this_node.inputs()
# Get the geometry from the second input
# (first input=0, second input=1, third=2, etc.)
second_input_geo = inputs[1].geometry()
```

You can then call methods on the geometry object (`geo` in this example) to modify the outgoing geometry. See the documentation for the [hou.Geometry](https://www.sidefx.com/docs/houdini/hom/hou/Geometry.html "A Geometry object contains the points and primitives that define a 3D geometric shape.  For example, each SOP node in Houdini generates a single Geometry object.") object for how to manipulate the geometry.

For example, to add a polygon to the geometry:

```python
poly = geo.createPolygon()
for position in (0,0,0), (1,0,0), (0,1,0):
    point = geo.createPoint()    
    point.setPosition(position)    
    poly.addVertex(point)
```

If you want to create a “source” node rather than a “filter” node, simply set the node’s number of inputs to 0 in the type properties. Calling `hou.pwd().geometry()` will return an empty geometry you can add to.

# Making the SOP interruptible

If the node takes a long time to cook when given large input geometry, you may want to be able to interrupt its cooking by pressing Escape. To make your SOP interruptible, periodically call [hou.updateProgressAndCheckForInterrupt()](https://www.sidefx.com/docs/houdini/hom/hou/updateProgressAndCheckForInterrupt.html "Deprecated: Use InterruptableOperation."). For example, you can press Escape to stop this SOP from cooking further:

```python
geo = hou.pwd().geometry()

# Evaluate the "t" parameter to see how much to translate each point.
translation = hou.Vector3(hou.parmTuple("t").eval())
for point in geo.points():
    # Set the new position for each point.    
    point.setPosition(point.position() + translation)
    # Check if the user pressed Escape.    
    if hou.updateProgressAndCheckForInterrupt():        
	    break
```

# SOP Errors and Warnings

If your Python surface node generates an exception, the node will turn red with an error and you can view the stack trace of the error by middle-clicking on it.

If you would like to generate an error message to the user that doesn’t contain a Python stack trace, raise a [hou.NodeError](https://www.sidefx.com/docs/houdini/hom/hou/NodeError.html "Raise this exception in a Python node to signal that the node is in error.") exception. For example, running:

```python
raise hou.NodeError("Invalid parameter settings")
```

will turn the node red with an error message of `"Invalid parameter settings"`. Similarly, you can add node warnings by raising instances of [hou.NodeWarning](https://www.sidefx.com/docs/houdini/hom/hou/NodeWarning.html "Raise this exception in a Python node to signal that the node has a warning.").

# Creating local variables for new attributes

[hou.Geometry.addAttrib](https://www.sidefx.com/docs/houdini/hom/hou/Geometry.html#addAttrib) contains a parameter to control whether Houdini creates a local variable for newly added attributes. There is no need to modify the varmap attribute directly.

# Profiling the SOP code

You can profile your Python surface node with Python’s `cProfile` module.

> [!tip] Tip
> If you are on Ubuntu, use `aptitude` to install `python-profiler` since the standard library is missing pstats in Ubuntu.

You should write your surface node’s script so the work is done inside a function, instead of a bunch of statements at the top level, for example:

```python
def cook():
    poly = geo.createPolygon()    
    for position in (0,0,0), (1,0,0), (0,1,0):        
	    point = geo.createPoint()        
	    point.setPosition(position)        
	    poly.addVertex(point)
cook()
```

Import `cProfile` and use `cProfile.runctx` to call your function instead of calling it directly:

```python
def cook():
    poly = geo.createPolygon()    
    for position in (0,0,0), (1,0,0), (0,1,0):        
	    point = geo.createPoint()        
	    point.setPosition(position)        
	    poly.addVertex(point)
cProfile.runctx('cook()', globals(), locals())
```

This will print a summary of your script’s run time to the Python shell when the node cooks.

# Writing Part of the SOP in C++

See [Extending HOM with C++](https://www.sidefx.com/docs/houdini/hom/extendingwithcpp.html) to see how to easily write a portion of your SOP in C++.

# Storing your Code Outside a Digital Asset

You may want to store your source code outside of your digital asset in order to manage it under a version control system. It is possible to use the Type Properties dialog to set up your asset’s parameters and then inject the Python source code into the `otl` file.

You can use the following function to update your digital asset to use the Python source code from a file:

```python
def loadPythonSourceIntoAsset(otl_file_path, node_type_name, source_file_path):
    # Load the Python source code.
    source_file = open(source_file_path, "rb")
    source = source_file.read()
    source_file.close()

    # Find the asset definition in the otl file.
    definitions = [
        definition
        for definition in hou.hda.definitionsInFile(otl_file_path)
        if definition.nodeTypeName() == node_type_name
    ]
    assert len(definitions) == 1
    definition = definitions[0]

    # Store the source code into the PythonCook section of the asset.
    definition.addSection("PythonCook", source)
```