---
author: wuzhou
comments: false
date: 2014-09-12 15:45:39+00:00
layout: post
slug: a-star-algorithm-note
title: A* 算法笔记
wordpress_id: 211
categories:
- 编程
tags:
- 编程方法
---

A* 是一种路径寻找算法。这种算法被广泛地运用于游戏开发中。它的基本思路就是把地图用图（graph）来表示，然后通过搜索图来获取路径。


# 广度优先搜索（Breath First Search）


广度优先搜索是 A* 算法的基础，也可以说是其他路径寻找算法的基础，使用它可以遍历到图形的每个点。广度优先的主要步骤有：

构建一个队列（Queue）用来存储图中的点，也就是是地图中的位置，然后重复下面的步骤直到这个队列为空为止。




  1. 把没有被访问（visited）过的点放入队列中。


  2. 从队列都头部取出一个点，把它标记为已访问过的点，同时与这个点相邻的（neighbors）、还没有被访问过的点放入队列中。


用 Python 代码可以这样表示：


    frontier = Queue()
    frontier.put(start)
    visited = {}
    visited[start] = True
    while not frontier.empty():
       current = frontier.get()
       for next in graph.neighbors(current):
          if next not in visited:
             frontier.put(next)
             visited[next] = True


但是目前所描述的的算法只是遍历了整个图，而没有记录下遍历这个图的路径。为了把路径的信息保留下，就不能只是简单地把一个被访问的点标记为已访问，而是要把是从哪一个点访问过来的信息保存起来，所以就有了下面的代码：


    frontier = Queue()
    frontier.put(start)
    came_from = {}
    came_from[start] = None
    while not frontier.empty():
       current = frontier.get()
       for next in graph.neighbors(current):
          if next not in came_from:
             frontier.put(next)
             came_from[next] = current


然后使用逆推就可以重建路径：


    current = goal
    path = [current]
    while current != start:
       current = came_from[current]
       path.append(current)




# Dijkstra 算法


在实际的应用中，寻找路径并不简单地、毫无约束地去寻找，往往还需要考虑从一个点到另一个点的成本（cost）。通常我们会需要寻找一个移动成本最小的路径。这时就需要用到 Dijkstra 算法。 这个算法的主要步骤有：

构建一个带有优先级的队列（Priority Queue）,用来存放需要访问的点，然后重复下面的步骤直达队列为空：




  1. 取出队列中的优先级最高的点，也就是移动成本最低的点，找出与他相邻的点。


  2. 计算移动到相邻点的成本。


  3. 如果与他相邻的点没有本访问过，把这个相邻点和移动到这个相邻点所需的成本（作为优先级）的放入队列中，并把这个当前点作为移动到该相邻点的前一个点。


  4. 如果移动到这个点成本小于之前从其他点移动这个点的成本，那么更新这个相邻点在队列中的优先级，并把这个当前点作为移动到该相邻点的前一个点。




该算法可以用代码表示为:




    frontier = PriorityQueue()
    frontier.put(start, 0)
    came_from = {}
    cost_so_far = {}
    came_from[start] = None
    cost_so_far[start] = 0
    while not frontier.empty():
       current = frontier.get()
       for next in graph.neighbors(current):
          new_cost = cost_so_far[current] + graph.cost(current, next)
          if next not in cost_so_far or new_cost < cost_so_far[next]:
             cost_so_far[next] = new_cost
             priority = new_costfrontier.put(next, priority)
             came_from[next] = current




# 启发式搜索


与 Dijkstra 算法类似，启发式搜索也要用到带有优先级的队列，不同的是启发式搜索考虑的不是移动到下一点的成本而是下一点到目标点之间的距离。该算法用代码可以表示为：


    frontier = PriorityQueue()
    frontier.put(start, 0)
    came_from = {}
    came_from[start] = None
    while not frontier.empty():
       current = frontier.get()
       if current == goal:
          break
          for next in graph.neighbors(current):
          if next not in came_from:
             priority = heuristic(goal, next)
             frontier.put(next, priority)
             came_from[next] = current


其中 heuristic 函数就是用来计算两点之间距离的方法。启发式算法得出的路径并不一定是移动成本最小的路径，但是它可以用较少的搜索次数找到目标点。


# A* 算法


A* 算法综合了 Dijkstra 算法和启发式搜索这两种算法。它同样在寻找路径时要用到带有优先级的队列，只不过这个优先级要同时考虑移动成本和到目标点的距离。A* 算法可以用下面的代码来表示：


    frontier = PriorityQueue()
    frontier.put(start, 0)
    came_from = {}
    cost_so_far = {}
    came_from[start] = None
    cost_so_far[start] = 0
    while not frontier.empty():
       current = frontier.get()
       for next in graph.neighbors(current):
          new_cost = cost_so_far[current] + graph.cost(current, next)
          if next not in cost_so_far or new_cost < cost_so_far[next]:
             cost_so_far[next] = new_cost
             priority = new_cost + heuristic(goal, next)
             frontier.put(next, priority)
             came_from[next] = current


A* 算法的优势就是在于尽量保持较小的移动成本的同时以较少的搜索次数来找到目标点。

参考：[Introduction to A*](http://www.redblobgames.com/pathfinding/a-star/introduction.html)
