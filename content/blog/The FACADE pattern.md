---
title: "The Facade Design Pattern"
date: 2020-10-05T12:10:45+24:00
description: "The Facade Design Pattern"
author: "Sajad Hayatlou"
type: "post"
---


### What is the FACADE

This pattern which fits in the category of `structural` design patterns, is responsible for encapsulating a complex subsystem behind a simple interface (`Facade interface`).

The purpose of FACADE pattern is to provide a unified interface to a set of interfaces in a subsystem. It actually defines a higher-level interface that makes the subsystem easier to use.


Structuring a system into subsystems helps `reduce complexity`. A common design goal is to minimize the communication and dependencies between subsystems. One way to achieve this goal is to introduce a facade object that provides a single, simplified interface to the more general facilities of a subsystem.


{{< figure src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/13/Facade_Design_Pattern_Class_Diagram_UML.svg/300px-Facade_Design_Pattern_Class_Diagram_UML.svg.png" 
  width=300 height=300 style="float:left;" >}}



### EXAMPLE

Let’s say that we want to start a car. The following diagram represents the legacy system, which allows us to do so:



{{< figure src="https://www.baeldung.com/wp-content/uploads/2018/04/facade-class-diagram.png" 
  style="float:left;" >}}

So the processes to start a car would be as this order:


* `airFlowController.takeAir()`
* `fuelInjector.on()`
* `fuelInjector.inject()`
* `starter.start()`
* `coolingController.setTemperatureUpperLimit(DEFAULT_COOLING_TEMP)`
* `coolingController.run()`
* `catalyticConverter.on()`


Similarly, stopping the engine also requires quite a few steps:


- `fuelInjector.off()`
- `catalyticConverter.off()`
- `coolingController.cool(MAX_ALLOWED_TEMP)`
- `coolingController.stop()`
- `airFlowController.off()`

Now, the facade is what we need here, the life saver. 
Let see an implementation of this savior:


{{< highlight java >}}
public class CarEngineFacade {
    private static int DEFAULT_COOLING_TEMP = 90;
    private static int MAX_ALLOWED_TEMP = 50;
    private FuelInjector fuelInjector = new FuelInjector();
    private AirFlowController airFlowController = new AirFlowController();
    private Starter starter = new Starter();
    private CoolingController coolingController = new CoolingController();
    private CatalyticConverter catalyticConverter = new CatalyticConverter();
 
    public void startEngine() {
        fuelInjector.on();
        airFlowController.takeAir();
        fuelInjector.on();
        fuelInjector.inject();
        starter.start();
        coolingController.setTemperatureUpperLimit(DEFAULT_COOLING_TEMP);
        coolingController.run();
        catalyticConverter.on();
    }
 
    public void stopEngine() {
        fuelInjector.off();
        catalyticConverter.off();
        coolingController.cool(MAX_ALLOWED_TEMP);
        coolingController.stop();
        airFlowController.off();
    }

{{< /highlight >}}

Now, to start and stop a car, we need only 2 lines of code, instead of 13:


{{< highlight java >}}

facade.startEngine();
// ...
facade.stopEngine();

{{< /highlight >}}

### In Summary

Use this pattern when you want to reduce number of direct interactions between your clients and subsystem’s implementations. You can decouple these two by introducing an entry point (The `FACADE` object, a unified interface) to the top most layer of your architecture.




