# Enumerating Polycubes

## Introduction
### What Are Polycubes?
Polyomino tiles are 2-D tiles formed from a set of *n* square tiles. In this project I worked with polycubes, their 3-D cousins, which are composed of cubes as opposed to squares. They look something like this...

![8-cube polycube][1]

My goal over the course of this project was to enumerate complete sets of polycubes. That is, given *n* cubes, I wanted to be able to generate a list of every possible arrangement, and a count of how many exist. 

## Methods

### Basic Functionality

__Representing Polycubes Internally__: A 'polycube' as represented in my code is simply a list of points in 3-D space where cubes should be placed. For example, a 1 x 1 x 2 polycube (*n* = 2) can be represented as {{1, 1, 1}, {1, 1, 2}}. These points can be converted into 3-D sparse arrays (3rd order tensors), where 1s represent cubes and 0s represent empty space.

__Rendering__: For debugging purposes, it's difficult to look at lists of numbers and determine whether or not your code is working. Looking at nice graphical representations is both easier and a lot more satisfying.

So this: {{1, 1, 1}, {1, 2, 1}, {1, 3, 1}, {1, 2, 2}} 

Becomes this:

![rendered polycube][8]

### Processing

__Checking Connections__: In order to be considered a valid polycube, all cubes in the form must be connected. To ensure all returned polycubes were valid, I implemented a helper function: *valid\[pc\]*. This function selects one cube from the polycube as a start block, and ensures that every other block in the form can in some way be connected back to that initial block. Every time the function finds a block that has a neigbor already included in the 'valid' chunk of blocks, that block and it's neighbors are 'validated'. If all blocks can be 'validated', the form itself is a 'valid' polycube.

 __Removing Duplicates__: This was one of the most challenging parts of my project, as there are so many factors to account for when removing duplicates. I ended up dividing the work into two functions. 
 
 The first is something called *relocate*, which moves a polycube on my coordinate system so it is pressed up against the edges of a box located at {*n*, *n*, *n*}. By doing this, I make certain that any similarly structured and oriented polycubes will look the same if compared. For example, the structures {{1, 1, 1}, {1, 1, 2}} and {{1, 1, 2}, {1, 1, 3}} are the same, but they would not be considered duplicates at a computer since they're at different positions. In such a senario, *relocate* could move the second structure -1 in the z axis, after which they could be considered identical.
 
 The second function is *rotate*, and is used by my program to generate rotations of polycubes (in increments of 90 degrees) by multiplying each point on the polycube by a rotation matrix.
    
    (* Rotation Matrices *)
    xrot[\[Theta]_] := {{1, 0, 0}, {0, 
       Cos[\[Theta]], -Sin[\[Theta]]}, {0, Sin[\[Theta]], Cos[\[Theta]]}}
    yrot[\[Theta]_] := {{Cos[\[Theta]], 0, Sin[\[Theta]]}, {0, 1, 
       0}, {-Sin[\[Theta]], 0, Cos[\[Theta]]}}
    zrot[\[Theta]_] := {{Cos[\[Theta]], -Sin[\[Theta]], 
       0}, {Sin[\[Theta]], Cos[\[Theta]], 0}, {0, 0, 1}}
    
    (* Rotate a polycube about the a axis by \[Theta] degrees *)
    (* NOTE: x = 1 | y = 2 | z = 3 *)
    rotate[pc_, axis_, \[Theta]_] :=  
     Dot[{xrot, yrot, zrot}[[axis]][\[Theta]], #] & /@ pc

### Complete Search
The first step was to produce a 'minimum viable product', something that could technically complete the task, and would be fast to implement. This allowed me to quickly develop and test a way to find and remove duplicates, without worrying about runtime optimizations. Below is the function I wrote to generate polycube sets through a complete search of possibilities in *n* x *n* x *n* space. 

    polycubeSet[n_Integer] := 
      Block[{allPoints, final, res}, 
       allPoints = 
        Table[{x, y, z}, {x, n}, {y, n}, {z, n}] // Flatten[#, 2] &;
       final = 
        Select[(Sort /@ Permutations[allPoints, {n}]) // DeleteDuplicates,
          valid[#] &];
       res = (relocate /@ final) // DeleteDuplicates // unique;
       Print[Style["COUNT \[DoubleRightArrow] ", Bold], Length@res];
       res
       ]
       
The above code generates all permutations of *n* points within the space, and tests each one for uniqueness and validity. With this method, I was only able to generate polycube sets up to *n* = 3 cubes.

![3-cube set][2]

### Confined Boxes Search
While examining the shapes of the polycubes I was generating, I noticed that every polycube generated for a certain *n* could be fit inside a box of dimensions that summed to no more than *n* + 2. For example, the dimensions of a box containing an *n* = 3 polycube could be 1 x 1 x 3 or 1 x 2 x 2.

In order to use this information to my advantage, I wrote a function that would generate a list of valid boxes, and then run a complete search within each of them. 

    polycubeBoxSet[n_] := 
      Block[{grids, res, set},
       grids = 
        Sort /@ Select[
           Permutations[Join[Range[n], Range[n], Range[n]], {3}], 
           Total[#] == (n + 2) &] // DeleteDuplicates;
       set = ((gridSet[#, n] & /@ grids) // Flatten[#, 1] &) // 
         removeDuplicates;
       res = Select[set, valid[#] &];
       Print[Style["COUNT \[DoubleRightArrow] ", Bold], Length@res];
       res
       ]

    gridSet[grid_, n_] := 
      Block[{pts, pcs}, 
       pts = Table[{x, y, z}, {x, grid[[1]]}, {y, grid[[2]]}, {z, 
           grid[[3]]}] // Flatten[#, 2] &;
       (relocate /@ Permutations[pts, {n}]) // DeleteDuplicates
       ]      

A confined search proved to be much faster than a complete search, and allowed me to generate polycube sets up to *n* = 5 cubes.

![5-cube set][3]

### Exploration Search
My final attempt (for now) is a method that works by processing structures in 'generations'. The function generates a complete set for each *n* up to the desired value, and then proceeds to the next set by returning every form that can be obtained by adding one cube to any open location. Since the structures are built upon each other, this approach removes the need to check whether or not all cubes are connected in a given polycube.
    
    polycubeExploreSet[n_] := 
      Block[{new, pt, res},
       new = res = {{{n, n, n}}};(* Allow building on all faces *)
       
       While[Length@First@res < n,
        new = 
         Sort /@ relocate /@ (explore /@ res // Flatten[#, 1] &) // 
           DeleteDuplicates // unique;
        res = relocate[#, 2] & /@ new;
        ]; Print[Style["COUNT \[DoubleRightArrow] ", Bold], Length@res]; 
       res
       ]
   
    explore[pc_] := 
      Block[{t},
       t = generateTensor[pc, Max@*Flatten@pc + 1];
       Join[pc, {#}] & /@ (openConnections[#, t] & /@ pc // 
           DeleteDuplicates // Flatten[#, 1] &) 
       ]

By cutting out the processing of extra possibilities, this approach managed to generate much larger sets (up to *n* = 8, a set of 6922 distinct polycubes). Below are some of my favorites...

![Some 8-cube polycubes][4]
![Some 8-cube polycubes][5]
![Some 8-cube polycubes][6]
![Some 8-cube polycubes][7]

## Results
The number of distinct polycubes possible for *n* cubes.

| n 	| polycubes 	|   	| n  	| polycubes 	|
|:---:	|-----------:	|---	|:----:	|---------:	    |
| 1 	|         1 	|   	| 6  	|       166 	|
| 2 	|         1 	|   	| 7  	|      1023 	|
| 3 	|         2 	|   	| 8  	|      6922 	|
| 4 	|         8 	|   	| 9  	|       - 	    |
| 5 	|        29 	|   	| 10 	|       - 	    |

## Notebook
The notebook containing my results can be found <a href="https://github.com/noelle-crawfish/Enumerating-Polycubes/tree/master/Final%20Project/Final%20Submission">here</a>.


[1]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/8cube.png?raw=true
[2]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/3cube.png?raw=true
[3]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/5cubes.png?raw=true
[4]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/8cubes1.png?raw=true
[5]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/8cubes2.png?raw=true
[6]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/8cubes3.png?raw=true
[7]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/8cubes4.png?raw=true
[8]: https://github.com/noelle-crawfish/Enumerating-Polycubes/blob/master/Wolfram%20Community%20Post/images/4omino.png?raw=true
