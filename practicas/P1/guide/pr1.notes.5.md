In the 5th phase of the implementation, you should implement builders and factories to create events and other objects from JSON specifications provided by the user. Do not proceed to the next phase until all tests for this phase pass.

**Motivation**

We now have all the components of our traffic simulation model, and we need a way to create events based on user-provided specifications. We will use the JSON format for this purpose. For example, the user might provide the following JSON to create a new vehicle event:

```json
{
  "type": "new_vehicle",
  "data": {
    "time": 1,
    "id": "v1",
    "maxspeed": 100,
    "class": 3,
    "itinerary": ["j1", "j3", "j4"]
  }
}
```

We will assume that the input is given as a JSON object for now (later we will see how to read it from a file), and we want to convert it automatically to a `NewVehicleEvent`.

**Factories**

We will use the concept of factories to achieve this. A factory is an object that creates other objects. We will define the following interface for factories:

```java
public interface Factory<T> {
    public T createInstance(JSONObject info);
    public List<JSONObject> getInfo();
}
```

This interface encapsulates the creation of objects of type `T`. For example, if we have a factory `f` of type `Factory<Event>`, we can create a new event using the following code:

```java
Event e = f.createInstance(x);
```

where `x` is the `JSONObject` corresponding to the JSON input.

The `getInfo()` method returns a list of JSON objects describing the objects that can be created by the factory. This method is mainly used in the second assignment and is not important for now.

**Builders**

We will use a particular type of factory that is very easy to extend. It is based on builders, which are objects that can create only one kind of object from a given `JSONObject`. The builder receives a `JSONObject` corresponding to the "data" section of the JSON input (i.e., the specifications of the object) and creates the corresponding object.

Builders for different objects share many commonalities, so we define the following abstract class to include all common functionality:

```java
public abstract class Builder<T> {
    private String _typeTag;
    private String _desc;

    public Builder(String typeTag, String desc) {
        if (typeTag == null || desc == null || typeTag.isBlank() || desc.isBlank())
            throw new IllegalArgumentException("Invalid type/desc");
        _typeTag = typeTag;
        _desc = desc;
    }

    public String getTypeTag() {
        return _typeTag;
    }

    public JSONObject getInfo() {
        JSONObject info = new JSONObject();
        info.put("type", _typeTag);
        info.put("desc", _desc);
        JSONObject data = new JSONObject();
        fillInData(data);
        info.put("data", data);
        return info;
    }

    protected void fillInData(JSONObject o) {
    }

    @Override
    public String toString() {
        return _desc;
    }

    protected abstract T createInstance(JSONObject data);
}
```

Specific builders will extend this class, fill in the `_typeTag` field to indicate what type of object they create, and implement the `createInstance` method to create the actual object.

**Example: NewVehicleEventBuilder**

```java
public class NewVehicleEventBuilder extends Builder<Event> {

    public NewVehicleEventBuilder() {
        super("new_vehicle", "A new vehicle");
    }

    @Override
    protected void fillInData(JSONObject o) {
        o.put("time", "The time at which the event is executed");
        o.put("maxspeed", "The vehicle's max speed");
        //... add other fields
    }

    @Override
    protected Event createInstance(JSONObject data) {
        int time = data.getInt("time");
        String id =...
        int maxSpeed =...
        int contClass =...
        List<String> itinerary =...

        return new NewVehicleEvent(time, id, maxSpeed, contClass, itinerary);
    }
}
```

Now, if we have a builder `b` of type `NewVehicleEventBuilder`, we can use the following code to create a `NewVehicleEvent`:

```java
Event e = b.createInstance(x);
```

where `x` is the `JSONObject` corresponding to the "data" section of the JSON input.

**BuilderBasedFactory**

Now, suppose we have builders for all possible events. We can define a factory that uses these builders as follows:

```java
public class BuilderBasedFactory<T> implements Factory<T> {
    private Map<String, Builder<T>> _builders;
    private List<JSONObject> _buildersInfo;

    public BuilderBasedFactory() {
        _builders = new HashMap<>();
        _buildersInfo = new LinkedList<>();
    }

    public BuilderBasedFactory(List<Builder<T>> builders) {
        this();
        for (Builder<T> b: builders) {
            addBuilder(b);
        }
    }

    public void addBuilder(Builder<T> b) {
        _builders.put(b.getTypeTag(), b);
        _buildersInfo.add(b.getInfo());
    }

    @Override
    public T createInstance(JSONObject info) {
        if (info == null) {
            throw new IllegalArgumentException("'info' cannot be null");
        }

        String type = info.getString("type");
        Builder<T> builder = _builders.get(type);
        if (builder!= null) {
            JSONObject data = info.has("data")? info.getJSONObject("data"): new JSONObject();
            T instance = builder.createInstance(data);
            if (instance!= null) {
                return instance;
            }
        }

        throw new IllegalArgumentException("Unrecognized 'info': " + info.toString());
    }

    @Override
    public List<JSONObject> getInfo() {
        return Collections.unmodifiableList(_buildersInfo);
    }
}
```

In the constructor, we pass a list of builders and store them in a map, where the keys are the type tags and the values are the builders. When we want to create an object, we simply look up the corresponding builder in the map and use it to create the object.

**Using Factories**

You will have three factories: one for light switching strategies, one for queuing strategies, and one for events. For example, the following code initializes the factory for light switching strategies:

```java
List<Builder<LightSwitchingStrategy>> lsbs = new ArrayList<>();
lsbs.add(new RoundRobinStrategyBuilder());
lsbs.add(new MostCrowdedStrategyBuilder());
Factory<LightSwitchingStrategy> lssFactory = new BuilderBasedFactory<>(lsbs);
```

And the following code initializes the factory for queuing strategies:

```java
List<Builder<DequeuingStrategy>> dqbs = new ArrayList<>();
dqbs.add(new MoveFirstStrategyBuilder());
dqbs.add(new MoveAllStrategyBuilder());
Factory<DequeuingStrategy> dqsFactory = new BuilderBasedFactory<>(dqbs);
```

And the following code initializes the factory for events:

```java
List<Builder<Event>> ebs = new ArrayList<>();
ebs.add(new NewJunctionEventBuilder(lssFactory, dqsFactory));
ebs.add(new NewCityRoadEventBuilder());
ebs.add(new NewInterCityRoadEventBuilder());
//... add other event builders
Factory<Event> eventsFactory = new BuilderBasedFactory<>(ebs);
```

Notice that we pass the light switching and queuing factories to the `NewJunctionEventBuilder` because it needs to be able to create the corresponding strategies.

**Testing**

Test your implementation with all the tests in the `factories` package. You can test one builder at a time, and then test the factories.
