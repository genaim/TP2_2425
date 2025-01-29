In the 2nd phase of the implementation, you should implement the RoadMap class. Don't move on to the next phase until you've got all the tests running correctly.

**RoadMap**

This class essentially maintains a list of all objects in the simulation and provides various ways to access them. It should include the following fields:

* `List<Vehicle> _vehicles;`
* `List<Road> _roads;`
* `List<Junction> _junctions;`
* `Map<String, Vehicle> _vehiclesMap;`
* `Map<String, Road> _roadsMap;`
* `Map<String, Junction> _junctionsMap;`

The maps are used for fast access to the objects.

* `addJunction(j)`:
    * Throws an exception if another junction with the same ID already exists.
    * Adds junction `j` to `_junctions` and a corresponding entry to `_junctionsMap`.

* `addRoad(r)`:
    * Throws an exception if another road with the same ID already exists.
    * Throws an exception if the source or destination junction of `r` is not in the roadmap.
    * Adds road `r` to `_roads` and a corresponding entry to `_roadsMap`.

* `addVehicle(v)`:
    * Throws an exception if a vehicle with the same ID already exists.
    * Throws an exception if the itinerary is not valid. An itinerary is valid if there is a road from the i-th junction to the (i+1)-th junction in the itinerary.

* `getVehicle(id)`, `getRoad(id)`, `getJunction(id)`:
    * Returns the corresponding simulated object.

* `getVehicles()`, `getRoads()`, `getJunctions()`:
    * Returns an unmodifiable view of the corresponding list of simulated objects (e.g., using `Collections.unmodifiableList(_vehicles)`).

* `report()`:
    * Returns a JSON structure as explained in the PDF.

**Testing**

Test the `RoadMap` class using the JUnit test `RoadMapTest`.
