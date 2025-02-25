# Multimodal Pathfinder in Valhalla

<p align="center">
<b>Roses are red,<br>
Violets are blue,<br>
Valhalla, Find me a BSS,<br>
I don't want to be late for my rendezvous</b>
</p>

## Overview 

The algorithm that we introduce in this document can be applied in multimodal pathfinding. More particularly, we're going 
to use Bike Sharing System(BSS) as example. 

In the case of BSS, two scenarios may happen:

1. The origin and the destination are close enough that there is no need to take a bike, in this case, Valhalla should 
be able to find a journey on foot

2. The origin and the destination are far away from each other, Valhalla should be able to find a journey on foot 
leading the user to a BSS station then another journey by bike leading the user to a another BSS station close to the 
destination in order to put the bike back and finally a journey, on foot again, from the second BSS station to the 
destination.

## Algorithm

The whole Prove of Concept is based on the existing algorithms A*, which is used in Valhalla widely.

To understand the multimodal A*, we can image that we are going to explore a graph that have two different layers 
which represent **foot layer** and **bike layer** respectively. The BSS stations are the nodes/edges that connect those 
two layers and through which the traveller can change the travel mode. With that said, we are actually constructing a 
larger graph by "separating" the nodes and the edges of different modes from the original graph and classic path finding 
algorithms can work on this larger graph as usual. In the case of BSS, both origin and destination are located on the 
"foot" layer. Note that this concept works with other multimodal cases too, for example, if a user want to take a bike
form the BSS station, go somewhere without "putting back" the bike (aka a "via"), we only need to project the 
destination on the **bike layer**.
 
The only trick of making the "separation" happen is to add new attribute: `travel_mode` into the key of`real_cost`(see 
below for more details).
 
* **Nomenclature**

  * **pred_map**: predecessor of the current node, used to build the final path
  * **real_cost**: map of real cost of the journey, whose keys are `node, travel_mode`, In this project, we use the sum of normlized length of the journey.  
  * **open_set**: a set of nodes to be visited, as classic implementation, we use a priority queue, who
  * **closed_set**: a set of nodes that has already the optimal solution
  * **normalize_cost**: to be able to accumulate the cost of different modes, we have to normalize the cost. In this 
  project, we use `the length of the edge` * `1` as the cost of foot mode and 
  `the length of the edge` * `foot speed / bike speed`, which actually means on bike, it will cost less/more.  
  * **normalize_heuristic_cost**: it's the same idea of **normalize_cost**. In this project, we use 
  `the crowfly distance` * `1` as the heuristic cost of foot mode and `the crowfly distance` * `foot speed / bike speed` as 
  the heuristic cost of bike mode
  

* **Pseudo code**
    
```python    

pred_map[origin] = None  
real_cost[(origin, foot mode)] = 0
open_set = PriorityQueue(key=sort_cost, value=(origin,foot mode))  
closed_set = {}  

def expand_forward(start_node, end_node, mode):

    tentative_cost = real_cost[start_node] + normalize_cost(start_node, end_node)
    sort_cost = tentative_cost + normalize_heuristic_cost(end_node, destination, mode)

    if (end_node, mode) not in open_set:
        pred_map[end_node] = start_node
        open_set.add(key=sort_cost, value=(neighbour, mode))
        
    if tentative_cost < real_cost[(end_node, mode)]:
        pred_map[end_node] = start_node
        sort_cost = tentative_cost + normalize_heuristic_cost(neighbour, destination, current_mode)
        open_set.update(key=sor_cost, value=(neighbour, mode))
        

def multimodal_A_star(origin, destination):
    
    origin_sort_cost = normalize_heuristic_cost(origin, destination)
    open_set.add(key=origin_sort_csot, value=(neighbour, foot mode))
    
    while open_set is not empty:  
        current_node, current_mode = open_set.pop()  
        
        if current_node == destination node:  
            return make_path(pred_map)  
            
        closed_set.add((current_node, current_mode)) 
           
        for every neighbour of current_node:  
            
            # Case 1: we continue the exploration with the same travel mode
            
            if (neighbour, current_mode) in closed_set:
                continue
            
            expand_forward(current, neighbour, current_mode)
            
            if current_node is not a bss node:
                continue
            
            # Case 2: the current node is a bss node, we should also explore the edge with another travel mode
            another_mode = {foot, bike} \ {current_mode}
            
            if (neighbour, another_mode) in closed_set:
                continue
            
            expand_forward(current, neighbour, another_mode) 
             
``` 

Example of a exploration, final result is on BSS
![Point to Point on BSS](../example_images/double_expansion_one_queue_bss.gif?raw=true)

Example of a exploration, final result is on foot
![Point to Point on foot](../example_images/double_expansion_one_queue_foot.gif?raw=true "Point to Point on foot")  
          
* **Isochrone**

The algorithm of isochrone is quite simple, we only need to set heuristic cost to zero, and it's done. 

![Isochrone on BSS](../example_images/double_expansion_isochrone.gif?raw=true "Isochrone on BSS")  
