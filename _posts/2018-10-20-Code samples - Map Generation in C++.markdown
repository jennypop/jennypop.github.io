---
layout: single
title:  Code sample - Map Generation in C++ (WIP)
date:   2018-10-20 17:20:43 +0100
excerpt: Random generation of connected grids of 0s / 1s
order: 1
header:
  teaser: map.jpg
---
I'm working on randomly generating maps as 2D grids of 0s and 1s, representing free space and obstacles. The goal is that the maps contain no disconnected areas of space that are blocked off by obstacles.

This is first stage of a larger project to implement different pathfinding algorithms in C++, to help myself practice C++ and AI techniques.

First I use generateUnconnectedGrid() to generate a map by choosing randomly between 0 and 1 for each tile.  
Then labelConnectedComponents() finds the connected components of free space using the flood fill algorithm, and returns a grid with them labelled.  
Finally connectGrid() uses the labelled grid to draw paths connecting the separate components on the original map.

Currently I'm using Catch2 to write unit tests to help with debugging the MapGenerator class. The code excerpt below likely contains bugs, because I have not tested it yet!

The project is on Github [here](https://github.com/jennypop/Pathfinding).

Sample: MapGenerator.cpp


```cpp
#include "pch.h"
#include "Map.h"
#include "MapGenerator.h"

#include <iostream>
#include <algorithm>
#include <stdlib.h>
#include <list>

namespace MapGenerator {

	// a grid with values 0 (space) and 1 (obstacle)
	grid generateUnconnectedGrid(int w, int h) {
		grid g;
		for (int i = 0; i < w; ++i) {
			vector<int> row;
			for (int j = 0; j < h; ++j) {
				int val = min(rand() % 5, 1);	// 1 in 5 times val = 0
				row.push_back(1 - val);
			}
			g.push_back(row);
		}
		return g;
	}

	tile findTileWithValue(grid g, int v) {
		for (size_t i = 0; i < g.size(); ++i) {
			for (size_t j = 0; j < g[0].size(); ++j) {
				if (g[i][j] == v) return pair<int, int>(i, j);
			}
		}
		return pair<int, int>(-1, -1);
	}

	bool inBounds(tile t, grid g) {
		int w = g[0].size(); int h = g.size();
		return (t.first > 0 && t.first < h && t.second > 0 && t.second < w);
	}

	// only applies to tiles of value 0, which represents unlabelled land
	void floodFill(grid &g, tile t, int label) {
		g[t.first][t.second] = label;

		tile n = tile(t); n.first = t.first - 1;
		tile w = tile(t); w.second = t.second - 1;
		tile e = tile(t); e.second = t.second + 1;
		tile s = tile(t); s.first = t.first + 1;

		if (inBounds(n, g) && g[n.first][n.second] == 0) floodFill(g, n, label);
		if (inBounds(w, g) && g[w.first][w.second] == 0) floodFill(g, w, label);
		if (inBounds(e, g) && g[e.first][e.second] == 0) floodFill(g, e, label);
		if (inBounds(s, g) && g[s.first][s.second] == 0) floodFill(g, s, label);
	}

	void replaceValue(grid &g, int a, int b) {
		for (size_t i = 0; i < g.size(); ++i) {
			for (size_t j = 0; i < g[0].size(); ++j) {
				if (g[i][j] == a) g[i][j] = b;
			}
		}
	}

	// returns a new grid showing connected components of non-obstacle space in g:
	// where connected components of space are labelled by numbers (1,2...)
	// obstacles are labelled -1
	grid labelConnectedComponents(grid g) {
		auto ccLabelledGrid(g);
		replaceValue(ccLabelledGrid, 1, -1);
		int ccCount = 0;
		tile t = findTileWithValue(g, 0);
		while (t.first >= 0) {					// while t is a valid tile
			ccCount++;
			floodFill(g, t, ccCount);
			t = findTileWithValue(g, 0);
		}
		return ccLabelledGrid;
	}

	// creates an obstacle-free path (of 0s) between two tiles
	void connectTiles(grid &g, tile t1, tile t2) {
		// go R, then U
		// shuffle (?) walk with random chance between either...
		// fill in on grid
		// stop your walk once you hit any tile of the right value
	}

	// takes: grid g, grid ccLabelledGrid of connected components of g.
	// modifies g so that all the land '0' tiles are connected
	void connectGrid(grid &g, grid ccLabelledGrid) {
		list<tile> componentTileList;
		tile x = pair<int, int>(0, 0);
		for (int i = 1; x.first >= 0; ++i) {
			x = findTileWithValue(g, i);
			componentTileList.push_back(x);
		}

		auto it = componentTileList.begin();
		while (it != componentTileList.end()) {
			connectTiles(g, *it, *(++it));
		}
	}

	Map generate(int w, int h) {
		return Map(generateUnconnectedGrid(w, h));
	}
}
```