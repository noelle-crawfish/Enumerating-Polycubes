##Introduction
###What Are Polycubes?
Polynomino tiles are 2-D tiles formed from a set of n square tiles. In this project I worked with polycubes, their 3-D cousins, which are composed of cubes as opposed to squares. They look something like this...

![8-cube polycube][1]

###Goals
My goal over the course of this project was to enumerate complete sets of polycubes. That is for a set of *n* cubes, I wanted to be able to generate a list of every possible arrangement.

##Methods
###Complete Search
Since I began this project as a part of the Wolfram Summer Camp, I had a limited amount of time to complete it. The first thing I did was write a function to generate polycube sets through a complete search of possibilities in *n* x *n* x *n* space. 

    polycubeSet[n_Integer] := (
      Block[{allPoints, final, res}, 
       allPoints = 
        Table[{x, y, z}, {x, n}, {y, n}, {z, n}] // Flatten[#, 2] &;
       final = 
        Select[(Sort /@ Permutations[allPoints, {n}]) // DeleteDuplicates,
          valid[#] &];
       res = (relocate /@ final) // DeleteDuplicates // unique;
       Print[Style["COUNT \[DoubleRightArrow] ", Bold], Length@res];
       res
       ])
###Confined Search
While examining the shapes of the polycubes I was generating, I noticed that every polycube generated for a certain *n* could be fit inside a box of dimensions that summed to no more than *n* + 2.

##Results
| n 	| polycubes 	|   	| n  	| polycubes 	|
|---	|-----------	|---	|----	|:---------:	|
| 1 	|         1 	|   	| 6  	|       166 	|
| 2 	|         1 	|   	| 7  	|      1023 	|
| 3 	|         2 	|   	| 8  	|      6922 	|
| 4 	|         8 	|   	| 9  	|       n/a 	|
| 5 	|        29 	|   	| 10 	|       n/a 	|

##Notebook
Attached to this post is a notebook containing my work.


  [1]: https://community.wolfram.com//c/portal/getImageAttachment?filename=98818omino.png&userId=1720553
