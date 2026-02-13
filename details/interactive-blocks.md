---
icon: hand-pointer
---

# Telemetry

## The basics of telemetry

Every Mechanism has a Tuning table and a display table on the robot. Mechanism tables contain the SmartMotorControllers used in that mechanism.&#x20;

All Mechanism tables are stored under `NT:/Mechanisms` NOT `NT:/SmartDashboard` however `NT:/SmartDashboard` does contain some useful Mechanism2d's that can be shown in your dashboard, and commands like "Live Tuning"

The image below has the Telemetry output of an Elevator with the Telemetry name of `Elevator` and the SmartMotorController TelemetryName of `ElevatorMotor`

<figure><img src="../.gitbook/assets/127.0.0.1 — AdvantageScope 9_2_2025 1_06_09 PM (1).png" alt=""><figcaption></figcaption></figure>

## Simulation vs Reality

The simulation view of each mechanism is done with `Mechanism2d`'s. These windows could be outputted while on the actual robot but that is not necessary as all of the data from the `Mechanism2d` is in the Telemetry fields and accessible to the user.

{% embed url="https://docs.wpilib.org/en/stable/docs/software/dashboards/glass/mech2d-widget.html" %}

## Mechanism Telemetry

Mechanism Telemetry is basically only a holder table for SmartMotorController Telemetry allowing for an easier time finding the values.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## Smart Motor Controller Telemetry

SmartMotorController Telemetry is highly configurable in fact your could enable and disable whatever telemetry you like with the example below.

```java
SmartMotorControllerTelemetryConfig motorTelemetryConfig = new SmartMotorControllerTelemetryConfig()
          .withMechanismPosition()
          .withRotorPosition()
          .withMechanismLowerLimit()
          .withMechanismUpperLimit();
          
SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
      .withClosedLoopController(4, 0, 0, DegreesPerSecond.of(180), DegreesPerSecondPerSecond.of(90))
      .withSoftLimit(Degrees.of(-30), Degrees.of(100))
      .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
      .withIdleMode(MotorMode.BRAKE)
      .withTelemetry("ElevatorMotor", motorTelemetryConfig)
```

<figure><img src="../.gitbook/assets/127.0.0.1 — AdvantageScope 9_2_2025 1_03_52 PM.png" alt=""><figcaption></figcaption></figure>

## Colors

You may notice there are different colors in the Mechanism windows displayed for simulation. These are there to provide you with reference points of your Soft and Hard Limits.&#x20;

* Green is the upper hard limit
* Red is the lower hard limit
* Pink is the upper soft limit
* Yellow is the lower soft limit

These are provided to you so you can identify relative motion easily between the simulation and reality.
