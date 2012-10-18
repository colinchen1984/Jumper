Jumper
======

This is a modified version of the pathfinding library [Jumper](https://github.com/Yonaba/Jumper/blob/master). <br/>
It features the same features as the official release. The specific change is an experimental feature included to support tiles with specific rules of walkability, such as one-way tiles.


<center><img src="http://ompldr.org/vZnhhaA" alt="" width="500" height="391" border="0" /></center>

##Installation
The current repository can be retrieved locally on your computer using the following bash script:

```bash
git clone -b experimental git://github.com/Yonaba/Jumper.git --recursive
````

##Usage##
###Adding Jumper to your project###
Copy this repository contents in a folder named __Jumper__ and put it inside your projet. Use *require* function to call the library.

```lua
local Jumper = require('Jumper.init')
```

**Note** :  You can add __.init.lua__ in your <tt>package path</tt>. As a result, requiring __Jumper__ will become easier.

```lua
package.path = package.path .. ';.\\?\\init.lua'
local Jumper = require ('Jumper')
```
	
**Note** : Some frameworks, like [Löve][] already includes  __.\init.lua__ in their <tt>package.path</tt>. In this case, you can just use :

```lua
local Jumper = require('Jumper')
```

###Initializing Jumper###

To initialize the pathfinder, you will have to pass __four values__. 

```lua
local allowDiagonal = true
local pather = Jumper(map,allowDiagonal,heuristicName,autoFill)
```

Only the first argument is __required__, the __others__ left are __optional__:

* __map__ refers to the matrix representing the 2D world.
* __allowDiagonal__ is a boolean saying whether or not *diagonal moves* are allowed. Will be considered as __true__ if not given.
* __heuristicName__ is a predefined string constant representing the *distance heuristic function* to be used for path computation.
* __autoFill__ is a boolean to __enable or not__ for [automatic path filling](https://github.com/Yonaba/Jumper/#automatic-path-filling).

###Setting your collision map
####The collision map table
The collision map is a regular 2-dimensional Lua table. All rows and columns __must be indexed__ with __consecutive integers__.<br/>
In this table, __each value__ is a __byte__ (*i.e an integer number between 0 and 255*). This value would represent two things:

* __whether or not__ a moving entity __can walk on__ the corresponding tile,
* when the tile is walkable, __from which direction__ this tile __can be entered__.
 
####Byte values : the basic rules
On a regular 2D map, an entity can move in __8 directions__:

* __orthogonally__: *north* (up), *south* (down), *west* (left), *east* (right),
* __Diagonally__: *north-east*, *south-east*, *north-west*, *south-west*.

We can represent this as follows:

<center><img src="http://ompldr.org/vZnhhZA" alt="" width="163" height="62" border="0" /></center>

And thus, map each of these directions with the following values:

<center><img src="http://ompldr.org/vZnhhZQ" alt="" width="163" height="62" border="0" /></center>

So, when we affect to a cell a byte-value of __2__, it means this tile __can only be entered from the North__. A byte-value of __128__ would mean this tile __can only be entered from South-East__.

####Composition rules
Say that we want a tile to be entered from __both North and South__ directions, or __both East and West__...How would we proceed ?<br/>

First, we will start calling __Jumper__, then load a third-party named *bitops.lua*.

```lua
local Jumper = require ('Jumper.init')
local Dir = require 'bitops '
```
First, consider these assumptions :

* Byte 255 corresponds to a fully walkable tile
* Byte 0 corresponds to a fully unwalkable tile (tile that cannot be crossed)

*bitops.lua* returns a table containing three functions:

* AND (...)
* OR (...)
* NOT (...)

These functions will help you baking __the desired byte value__ to be set to a cell on a collision map. Yet, they are very easy to use.

* *Tile A* can be entered from *North or South or West or East*:

```lua
local byteA = Dir.OR(2,64,8,16)
```

* Now we want *Tile A* to remain as it is, but disable entry from *north-west* :

```lua
local byteB = Dir.AND(byteA,Dir.NOT(1))
```

* *Tile B* can be entered __from anywhere but__ *west, east or south-east*:

```lua
local byteB = Dir.NOT(8, 16,128)
```

* We want *Tile B* to remain as it is, but allow entry from *west*:

```lua
local byteB = Dir.OR(byteB,8)
```

And that's it!



###Distance heuristics###

*Jumper* features three (3) distance heuristics.

* MANHATTAN Distance : *|dx| + |dy|*
* EUCLIDIAN Distance : *sqrt(dx*dx + dy*dy)*
* DIAGONAL Distance : *max(|dx|, |dy|)*

By default, when you init  *Jumper*, __MANHATTAN__ will be used by default if 
no heuristic was specified.<br/>
If you want to use __another distance heuristic__, you will have to pass one of the following predefined strings:

```lua
"MANHATTAN" -- for MANHATTAN Distance
"EUCLIDIAN" -- for EUCLIDIAN Distance
"DIAGONAL" -- for DIAGONAL Distance
```

As an example :

```lua
local walkable = 0
local allowDiagonal = true
local Heuristics. = require 'Jumper.core.heuristics'
local Jumper = require('Jumper.init')
local pather = Jumper(map,walkable,allowDiagonal,'EUCLIDIAN')
```

You can __alternatively__ use <tt>pather:setHeuristic(Name)</tt>:

```lua
local walkable = 0
local allowDiagonal = true
local Jumper = require('Jumper.init')
local pather = Jumper(map,walkable,allowDiagonal)
pather:setheuristic('EUCLIDIAN')
```

##Public interface##

###Pathfinder class interface

Once Jumper was loaded and initialized properly, you can used one of the following methods listed below.<br/>
__Assuming <tt>pather</tt> represents an instance of <tt>Jumper</tt> class.__
	
#####pather:setHeuristic(NAME)
Will change the *distance heuristic* used.<br/>
__NAME__ must be passed as a string. Possible values are *MANHATTAN, EUCLIDIAN, DIAGONAL* (case-sensitive!).

* Argument __NAME__: *string*
* Returns: *nil*

#####pather:getHeuristic() 
Will return a reference to the *distance heuristic function* internally used.<br/>

* Argument: *nil*
* Returns: *function*

#####pather:setDiagonalMoves(Bool)
Argument must be a *boolean*. *True* will allow diagonal moves, *false* will allow *only straight-moves*.<br/>

* Argument __Bool__: *boolean*
* Returns: *nil*

#####pather:getDiagonalMoves()
Returns a *boolean* saying whether or not diagonal moves are allowed.

* Argument: *nil*
* Returns: *boolean*

#####pather:getGrid()
Returns a reference to the *internal grid* used by the pathfinder.
This grid is __not__ the map matrix given on initialization, but a __virtual representation__ used internally.

* Argument: *nil*
* Returns: *grid* (regular Lua table)

#####pather:getPath(startX,startY,endX,endY)
Main function, returns a path from location *[startX,startY]* to location*[endX,endY]* as an __ordered array__ of tables (*{x = ...,y = ...}*).<br/>
Otherwise returns *nil* if there is __no valid path__.<br/>
Also returns a __second value__ representing the __total cost of the move__ when a path was found.

* Argument __startX__: *integer*
* Argument __startY__: *integer*
* Argument __endX__: *integer*
* Argument __endY__: *integer*
* Returns: *path* (regular Lua table) or *nil*
* Returns: *cost* or *nil*

#####pather:fill(Path)
Polishes a path

* Argument __Path__: *path* (regular Lua table)
* Returns: *path* (regular Lua table)

#####pather:setAutoFill(bool)
Turns *on/off* the __autoFilling__ feature. When *on*, paths returned with <tt>pather:getPath()</tt> will always be automatically polished with <tt>pather:fill()</tt>

* Argument __bool__: *boolean*
* Returns: *nil*

#####pather:getAutoFill()
Returns the status of the __autoFilling__ feature

* Argument __bool__: *nil*
* Returns: *boolean*

###Grid class interface

<tt>pather:getGrid()</tt> returns a reference to the *internal grid* used by the pathfinder.
On this reference, you can use one of the following methods.<br/>

__Assuming *grid* holds the returned value from <tt>pather:getGrid()</tt>__

#####grid:getNodeAt(x,y)
Returns a reference to *node (X,Y)* on the grid.

* Argument __x__: *integer*
* Argument __y__: *integer*
* Returns: *node* (regular Lua table)

#####grid:isWalkableAt(x,y,ux,uy)
Returns a boolean saying whether or not *node (X,Y)* __exists__ on the grid __and is enterable__ heading along *a unit vector (ux,uy)*.

* Argument __x__: *integer*
* Argument __y__: *integer*
* Argument __ux__: *integer* (either 0 or 1)
* Argument __uy__: *integer* (either 0 or 1)
* Returns: *boolean*

#####grid:isWalkable(x,y)
Returns a boolean saying whether or not *node (X,Y)* __exists__ on the grid and __is walkable__.

* Argument __x__: *integer*
* Argument __y__: *integer*
* Returns: *boolean*

#####grid:setWalkableAt(x,y,walkable)
Sets *node (X,Y)* walkable byte-value

* Argument __x__: *integer*
* Argument __y__: *integer*
* Argument __walkable__: *boolean*
* Returns: *nil*

#####grid:getNeighbours(node,allowDiagonal)
Returns an array list of *nodes neighbouring node (X,Y)*. 
This list will include or not adjacent nodes regards to the boolean *allowDiagonal*.

* Argument __node__: *node* (regular Lua table)
* Argument __allowDiagonal__: *boolean*
* Returns: *neighbours* (regular Lua table)


##Handling paths##

###Using native <tt>pather:getPath()</tt>###

Using <tt>pather:getPath()</tt> will return a table representing a path from one node to another.<br/>
The path is stored in a table using the form given below:

```lua
path = {
          {x = 1,y = 1},
          {x = 2,y = 2},
          {x = 3,y = 3},
          ...
          {x = n,y = n},
        }
```

You will have to make your own use of this to __route your entities__ on the 2D map along this path.<br/>
Note that the path could contains some *holes* because of the algorithm used.<br/>
However, this __will not cause any issue__ as the move from one step to another along this path is __always straight__.<br/>
You can accomodate of this by yourself, or use the __path filling__ feature.

###Path filling###

__Jumper__ provides a __path filling__ feature that can be used to polish a path early computed, filling the holes it may contain.

```lua  
local walkable = 0
local allowDiagonal = true
local Jumper = require('Jumper.init')
-- Assuming map is defined
local pather = Jumper(map,walkable,allowDiagonal)
local path, length = pather:getPath(1,1,3,3)
-- Just pass the path to pather:fill().
pather:fill(path)
```

###Automatic path filling###
This feature will trigger the <tt>pather:fill()</tt> everytime <tt>pather:getPath()</tt> will be called.<br/>
Yet, it is very simple to use:

```lua  
local walkable = 0
local allowDiagonal = true
local Jumper = require('Jumper.init')
-- Assuming map is defined
local pather = Jumper(map,walkable,allowDiagonal)
pather:setAutoFill(true)
local path, length = pather:getPath(1,1,3,3)
-- No need to use path:fill() now !
```

##Chaining##
All setters can be chained.<br/>
This is convenient if you need to __reconfigure__ the pather instance in a __quick and elegant manner__.

```lua 
local Jumper = require ('Jumper.init')
-- Assuming map is defined
local pather = Jumper(map,0)
-- some code
-- calling the pather, reconfiguring it and yielding a new path
local path,length = pather:setAutoFilltrue)
				   :setHeuristic('EUCLIDIAN')
				   :setDiagonalMoves(true)
				   :getPath(1,1,4,3)
-- That's it!				   
```

##Object-orientation
__Jumper__ uses [30log][] as a lightweight *object-orientation* framework.<br/>
*When loading* Jumper, the path to this third-party library is automatically added, so that you can *require* this third-party very easily if you need it.

```lua
local Jumper = require 'Jumper.init'
local Class = require '30log'
```

##Participating Libraries##

* [30log][]
* [Binary heaps][]

##Credits and Thanks##

* [Daniel Harabor][], [Alban Grastien][] : for [technical papers][].<br/>
* [XueXiao Xu][], [Nathan Witmer][]: for their amazing [port][] in Javascript<br/>

##License##

This work is under [MIT-LICENSE][]<br/>
Copyright (c) 2012 Roland Yonaba

    Permission is hereby granted, free of charge, to any person obtaining a
    copy of this software and associated documentation files (the
    "Software"), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be included
    in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[Jump Point Search]: http://harablog.wordpress.com/2011/09/07/jump-point-search/
[30log]: http://yonaba.github.com/30log
[Lua]: http://lua.org
[Binary heaps]: http://yonaba.github.com/Binary-Heaps/
[Löve]: https://love2d.org
[Löve 0.8.0 Framework]: https://love2d.org
[Dragon Age : Origins]: http://dragonage.bioware.com
[Moving AI]: http://movingai.com
[Nathan Witmer]: https://github.com/aniero
[XueXiao Xu]: https://github.com/qiao
[port]: https://github.com/qiao/PathFinding.js
[Alban Grastien]: http://www.grastien.net/ban/
[Daniel Harabor]: http://users.cecs.anu.edu.au/~dharabor/home.html
[technical papers]: http://users.cecs.anu.edu.au/~dharabor/data/papers/harabor-grastien-aaai11.pdf
[MIT-LICENSE]: http://www.opensource.org/licenses/mit-license.php
