# Enumerating Polycubes

## Introduction
### What Are Polycubes?
Polyomino tiles are 2-D tiles formed from a set of *n* square tiles. In this project I worked with polycubes, their 3-D cousins, which are composed of cubes as opposed to squares. They look something like this...

![8-cube polycube][1]

My goal over the course of this project was to enumerate complete sets of polycubes. That is, given *n* cubes, I wanted to be able to generate a list of every possible arrangement, and a count of how many exist. 

## Methods
### Complete Search
The first step was to produce a 'minimum viable product', something that could technically complete the task, and would be fast to implement. This allowed me to quickly develop and test a way to find and remove duplicates, without worrying about runtime optimizations. Below is the function I wrote to generate polycube sets through a complete search of possibilities in *n* x *n* x *n* space. 

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
       
With this method, I was only able to generate sets up to *n* = 3.

![3-cube set][2]

### Confined Boxes Search
While examining the shapes of the polycubes I was generating, I noticed that every polycube generated for a certain *n* could be fit inside a box of dimensions that summed to no more than *n* + 2. 

In order to use this information to my advantage, I wrote a function that would generate a list of valid boxes, and then ran a complete search within each of them. 

    polycubeBoxSet[n_] := (
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
       ])

    gridSet[grid_, n_] := (
      Block[{pts, pcs}, 
       pts = Table[{x, y, z}, {x, grid[[1]]}, {y, grid[[2]]}, {z, 
           grid[[3]]}] // Flatten[#, 2] &;
       (relocate /@ Permutations[pts, {n}]) // DeleteDuplicates
       ])      
A confined search proved to be much faster than a complete search, and allowed me to generate sets up to *n* = 5.

![5-cube set][3]

### Exploration Search
This method is one I had considered from the start of my project to be a possible solution, but I didn't actually implement until later on due to worries about not finishing. However, it ended up being one of the simplest to implement since I didn't have to check the validity of each structure at the end.
    
    polycubeExploreSet[n_] := (
      Block[{new, pt, res},
       new = res = {{{n, n, n}}};(* Allow building on all faces *)
       
       While[Length@First@res < n,
        new = 
         Sort /@ relocate /@ (explore /@ res // Flatten[#, 1] &) // 
           DeleteDuplicates // unique;
        res = relocate[#, 2] & /@ new;
        ]; Print[Style["COUNT \[DoubleRightArrow] ", Bold], Length@res]; 
       res
       ])
   
    explore[pc_] := (
      Block[{t},
       t = generateTensor[pc, Max@*Flatten@pc + 1];
       Join[pc, {#}] & /@ (openConnections[#, t] & /@ pc // 
           DeleteDuplicates // Flatten[#, 1] &) 
       ])
By cutting out the processing of possibilities that aren't valid polycubes, this algorithm managed to generate mcuh larger sets (up to *n* = 8, a set of 6922 distinct polycubes). Below are some of my favorites...

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
