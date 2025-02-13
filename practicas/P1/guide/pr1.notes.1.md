In the 1st phase of implementation, you should implement the classes of the simulated objects. Don't move on to the next phase until you've got all the tests running correctly.

**Vehicle**

* The status of a vehicle can be one of the following: `PENDING` (has not yet entered the first road), `TRAVELING` (currently traveling on a road), `WAITING` (waiting in a junction), or `ARRIVED` (arrived at its destination). 

* `advance(time)`:
    * If the vehicle is not traveling, do nothing and simply return. This is very important. 
    * Update the vehicle's location to its current location plus its speed; in other words, move the vehicle. If the new location exceeds the road length, set it to the road length. 
    * Calculate the distance (`d`) the vehicle has traveled. This is the new location minus the old location. 
    * Calculate the current contamination (`c`), which depends on `d` and the vehicle's contamination class. 
    * Add `c` to the vehicle's total contamination and to the contamination of the road the vehicle is currently traveling on. 
    * If the vehicle is at the end of the road, call the `enter` method of the destination junction to add the vehicle to that junction's queue. Change the vehicle's state accordingly, set its speed to 0, etc. Do not remove the vehicle from the current road; it is still there. 
    * It is recommended to keep track of the index of the last junction encountered. This starts at 0 and is incremented by 1 when entering a junction's queue. 

* `moveToNextRoad()`:
    * If the status is not `PENDING` or `WAITING`, throw an exception, as this method should only be called in those states. 
    * If the vehicle is currently on a road (i.e., the current road is not null, or the index of the last seen junction is greater than 0), call the `exit` method of the current road to remove the vehicle from that road. 
    * If the vehicle is waiting in the last junction of its itinerary, change its status to `ARRIVED`, set its road to null, its speed to 0, etc. 
    * Otherwise, request the next road (`r`) in the vehicle's itinerary from the current junction. Then, enter road `r`. The status should be changed to `TRAVELING`, and the location should be set to 0. 


**Road**

* This is an abstract class that will be extended by `CityRoad` and `InterCityRoad`. 
* It is important that the constructor calls `addIncomingRoad` on the destination junction and `addOutgoingRoad` on the source junction. This allows the junctions to determine how to reach each other using the `roadTo` method. 
* It should maintain a list of vehicles (`List<Vehicle>`) currently on the road. This list should always be sorted by the vehicles' locations in descending order. 

* `enter(v)`:
    * This method is called from `moveToNextRoad` to enter a road. 
    * Add the vehicle `v` to the end of the list of vehicles. 
    * Verify that the vehicle's speed is 0 and its location is 0; otherwise, throw an exception. 

* `exit(v)`:
    * Simply remove vehicle `v` from the list of vehicles. 

* `advance(time)`:
    * Call the abstract method `reduceTotalContamination()` to reduce the current contamination. The specific implementation is provided in `CityRoad` or `InterCityRoad`. 
    * Call the abstract method `updateSpeedLimit()` to set the speed limit. The specific implementation is provided in `CityRoad` or `InterCityRoad`. 
    * For each vehicle `v` in the list of vehicles:
        1. Change its speed to the value returned by `calculateVehicleSpeed(v)`. Note that `calculateVehicleSpeed` is an abstract method. 
        2. Call `v.advance(time)` to advance the vehicle. 
    * Sort the list of vehicles by location, as one vehicle might have overtaken another in the previous loop. 


**Note:** The classes `CityRoad` and `InterCityRoad` extend `Road` and implement the abstract methods using the formulas explained in the PDF.

**Junction**

* `List<Road> _inRoads;` 
* `List<List<Vehicle>> _queues;` // the i-th queue corresponds to the i-th road in `_inRoads` 
* `Map<Road, List<Vehicle>> _queueByRoad;` // for efficient lookup of queues 
* `Map<Junction, Road> _outRoadByJunction;` // indicates the road to take to reach a given junction 
* `int _greenLightIndex;` // the index of the road in `_inRoads` that has a green light (-1 if all lights are red) 
* `int _lastSwitchingTime;` // the last time the green light was switched from one road to another 
* `LightSwitchingStrategy _lss;` 
* `DequeuingStrategy _dqs;` 

* `addIncomingRoad(r)`:
    * Check that `r.getDest()` is equal to this junction. 
    * Add the road `r` to `_inRoads`. 
    * Add a new queue to `_queues`. 
    * Add a corresponding entry to `_queueByRoad`. 

* `addOutgoingRoad(r)`:
    * Check that `r.getSrc()` is equal to this junction. 
    * Update `_outRoadByJunction` accordingly. 

* `enter(v)`:
    * Determine the road `r` on which vehicle `v` is traveling. 
    * Get the queue `q` associated with road `r`. 
    * Add `v` to `q`. 

* `roadTo(j)`:
    * Return the road associated with junction `j` in `_outRoadByJunction`. 

* `advance(time)`:
    * If there is a road with a green light (`_greenLightIndex` is not -1): 
        1. Get the queue `q` of the road with the green light. 
        2. Use `_dqs.dequeue(q)` to obtain the list of vehicles (`l`) that should be advanced. 
        3. For each vehicle `v` in `l`:
            * Call `v.moveToNextRoad()` to move the vehicle to the next road. 
            * Remove `v` from `q`. 
    * Call `_lss.chooseNextGreen(...)` to determine the next road that should receive a green light. If it is different from the current one, update `_greenLightIndex` and `_lastSwitchingTime` accordingly. 


**DequeuingStrategy**

There are two strategies:

* `MoveFirstStrategy`: Moves the first vehicle in the queue. The `dequeue` method returns a list containing the first element of the queue (or an empty list if the queue is empty). 
* `MoveAllStrategy`: Moves all vehicles in the queue. The `dequeue` method returns a copy of the input queue. Important: The `dequeue` method should not remove elements from the queue; this is handled in the junction's `advance` method. 


**LightSwitchingStrategy**

There are two strategies:

* `RoundRobinStrategy(timeSlot)`:
    * If `currGreen` is -1, give green to the first road (return 0). 
    * If `currTime - lastSwitchingTime < timeSlot`, maintain the current green light (return `currGreen`). 
    * Otherwise, give green to the next road in the list, in a circular manner. 

* `MostCrowdedStrategy(timeSlot)`:
    * If `currGreen` is not -1 and `currTime - lastSwitchingTime < timeSlot`, maintain the current green light (return `currGreen`). 
    * Otherwise, find the most crowded road, starting the search at index `currGreen + 1` and proceeding in a circular manner. Give the green light to that road. If multiple roads have the same number of vehicles, the first one encountered gets the green light. 


**report() method**

In all simulated objects, the `report()` method should return a JSON representation as specified in the assignment statement. 


**Testing**

When you have finished implementing these classes, test them using the corresponding JUnit tests: `VehicleTest`, `CityRoadTest`, `InterCityRoadTest`, `JunctionTest`, `MostCrowdedStrategyTest`, `RoundRobinStrategyTest`, `MoveFirstStrategyTest`, and `MoveAllStrategyTest`.
