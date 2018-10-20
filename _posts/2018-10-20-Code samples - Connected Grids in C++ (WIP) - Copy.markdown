---
layout: single
title:  Code samples - Connected Grids in C++ (WIP)
date:   2018-10-20 17:20:43 +0100
---

I'm working on randomly generating maps as 2D grids of 0s and 1s, representing free space and obstacles. I'm making sure that there are no disconnected areas of space that are blocked off from one another by obstacles.

First I generate a map by making each tile a random choice between 0 or 1. 
Then I label the connected components of free space using the flood fill algorithm. 
Finally I connect all of the separate connected components by drawing paths of 0s on the map to join them. 

Currently I'm using Catch2 to write unit tests to help with debugging the MapGenerator class.

The project is at: https://github.com/jennypop/Pathfinding