+++
title = "广度优先搜索迷宫"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2018-05-07T01:23:38
updated_at = 2018-05-07T01:23:38
description = ""
tags = ["go", "算法"]
+++

## 迷宫原型

先创建一个文件，写入迷宫原型的坐标

`maze.in`

写入下面内容

```bash
6 5
0 1 0 0 0
0 0 0 1 0
0 1 0 1 0
1 1 1 0 0
0 1 0 0 1
0 1 0 0 0
```

## 实现

`maze.go`

```golang
package main

import (
	"fmt"
	"os"
)

// 将 maze.in 中的文件内容读取，保存到 maze 二维数组中
// 实现的数据结构： [[0 1 0 0 0] [0 0 0 1 0] [0 1 0 1 0] [1 1 1 0 0] [0 1 0 0 1] [0 1 0 0 0]]
func readMaze(filename string) [][]int {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	var row, col int
	fmt.Fscanf(file, "%d %d", &row, &col)

	maze := make([][]int, row)
	for i := range maze {
		maze[i] = make([]int, col)
		for j := range maze[i] {
			fmt.Fscanf(file, "%d", &maze[i][j])
		}
	}
	return maze
}

type point struct {
	i, j int
}

var dirs = [4]point{{-1, 0}, {0, -1}, {1, 0}, {0, 1}}

func (p point) add(r point) point {
	return point{p.i + r.i, p.j + r.j}
}

func (p point) at(grid [][]int) (int, bool) {
	if p.i < 0 || p.i >= len(grid) {
		return 0, false
	}
	if p.j < 0 || p.j >= len(grid[p.i]) {
		return 0, false
	}
	return grid[p.i][p.j], true
}

func walk(maze [][]int, start, end point) [][]int {
	steps := make([][]int, len(maze))
	for i := range maze {
		steps[i] = make([]int, len(maze[i]))
	}

	Q := []point{start}
	for len(Q) > 0 {
		cur := Q[0]
		Q = Q[1:]

		// 如果发现终点，退出探索
		if cur == end {
			break
		}

		for _, dir := range dirs {
			next := cur.add(dir)

			// 判断是否是不允许走的节点（maze.in 中的 1），如果是跳过去看下一个节点
			val, ok := next.at(maze)
			if !ok || val == 1 {
				continue
			}

			// 判断是否走过，如果是已经走过的节点不允许再走
			val, ok = next.at(steps)
			if !ok || val != 0 {
				continue
			}

			// 如果是起点，不允许再走
			if next == start {
				continue
			}

			curSteps, _ := cur.at(steps)
			steps[next.i][next.j] = curSteps + 1
			Q = append(Q, next)
		}
	}

	return steps
}

func printPath(maze [][]int) {

	for _, row := range maze {
		for _, val := range row {
			fmt.Printf("%3d", val)
		}
		fmt.Println()
	}
}

func printStepNum(s [][]int, end point) {
	fmt.Printf("到达终点（%d, %d)的总步数是： %d", end.i, end.j, s[end.i][end.j])
}

func main() {
	// 获取文件中的地图原型坐标
	maze := readMaze("maze/maze.in")
	fmt.Println("迷宫原型：")
	printPath(maze)
	fmt.Println("--------------------------")

	fmt.Println("走迷宫的结果:")
	end := point{len(maze) - 1, len(maze[0]) - 1}

	// 探索迷宫
	steps := walk(maze, point{0, 0}, end)

	// 打印行走路径
	printPath(steps)
	fmt.Println("--------------------------")

	// 打印到达终点需要的步数
	printStepNum(steps, end)
}
```

## 结果

```bash
迷宫原型：
  0  1  0  0  0
  0  0  0  1  0
  0  1  0  1  0
  1  1  1  0  0
  0  1  0  0  1
  0  1  0  0  0
--------------------------
走迷宫的结果:
  0  0  4  5  6
  1  2  3  0  7
  2  0  4  0  8
  0  0  0 10  9
  0  0 12 11  0
  0  0 13 12 13
--------------------------
到达终点（5, 4)的总步数是： 13
```