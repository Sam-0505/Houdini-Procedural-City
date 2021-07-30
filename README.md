# Houdini_Procedural_City

This is practice project to generate a city in houdini procedurally using Python. Some of the important aspects ,where python is used, are loading building assets as well as setting attributes.

NOTE: At many places, VEX or node based approach will be a better and faster approach but the intention of the project was to learn to implement things in houdini using python.

# Important Code Snippets

## 1) Buidling Asset Loading
```ruby
import os

#fpath gets the path to the folder containing building assets
buildings_path=hou.node(".").parm("fpath").eval()
buildings_name=os.listdir(buildings_path)

switch_houses=hou.node(".").createNode("switch","switch_houses")
switch_com=hou.node(".").createNode("switch","switch_commercial")
merge=hou.node(".").createNode("merge")
counter=0

#Variables to store the number of house 
#and commercial building assets present
housesCount=0
comCount=0

#The naming convention followed is
# commercial_{x} where x becomes the commercial building id
# house-{x} where x becomes the house id

for bname in buildings_name:
    if bname.endswith(".fbx"):
        #variables
        path=buildings_path+bname
        id=bname.split("_")[1].split(".")[0]
        type=bname.split("_")[0]
        
        #There will be four nodes for each building before merge and switch nodes
        #Nodes: file, bound, transform and attribute create
        #set the file node
        file_node=hou.node(".").createNode("file")
        file_node.parm("file").set(path)
        
        #set the transform node
        transform=hou.node(".").createNode("xform")
        transform.parm("scale").setExpression("ch('../u_scale')")
        transform.setFirstInput(file_node)
        
        #set the attribute_create
        attr=hou.node(".").createNode("attribcreate")
        attr.parm("numattr").set(2)
        
        attr.parm("name1").set("buidling_id")
        attr.parm("type1").set(1)
        attr.parm("value1v1").set(id)
        
        attr.parm("name2").set("buidling_type")
        attr.parm("type2").set(3)
        attr.parm("string2").set(type)
        attr.setFirstInput(transform)
        
        merge.setInput(counter,attr)
        counter+=1
        
        if type=="commercial":
           switch_com.setInput(comCount,attr)
           comCount+=1
           
        elif type=="house":
            switch_houses.setInput(housesCount,attr)
            housesCount+=1
            
        else:
            pass
hou.node("../CONTROL").parm("houseNum").set(housesCount)
hou.node("../CONTROL").parm("commercialNum").set(comCount)
```
## 2) Random asset assignment

```ruby
import random

node = hou.pwd()
geo = node.geometry()

# Add code to modify contents of geo.
# Use drop down menu to select examples.
points=geo.points()
if len(points):
    houseNum=hou.node("../../CONTROL").parm("houseNum").eval() #Get the total number of house assets
    comNum=hou.node("../../CONTROL").parm("commercialNum").eval() #Get the total number of commercial building assets
    geo.addAttrib(hou.attribType.Point,"id",-1)
    
    #Set the end parameter for the randint function based on the type of point in map
    end=0
    type=points[0].attribValue("block_type")
    if type=="commercial" :
        end=comNum
    elif type=="house" :
        end=houseNum
    else :
        pass

for point in points:
    point.setAttribValue("id",random.randint(1,end))
```
