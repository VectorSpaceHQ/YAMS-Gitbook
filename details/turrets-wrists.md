---
icon: castle
---

# Turrets/Wrists

## Why are Turrets and Wrists the same thing? Aren't they both Arms?

Turrets and Wrists are known as Pivots in YAMS. They are different from Arms as they are not impacted by gravity to such a significant extent.

## Basic Turret Config

From the YAMS examples we have the following as a basic config.

```java
TalonFXS                   turretMotor = new TalonFXS(1);
SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
      .withControlMode(ControlMode.CLOSED_LOOP)
      .withClosedLoopController(4, 0, 0, DegreesPerSecond.of(180), DegreesPerSecondPerSecond.of(90))
      // Configure Motor and Mechanism properties
      .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
      .withIdleMode(MotorMode.BRAKE)
      .withMotorInverted(false)
      // Setup Telemetry
      .withTelemetry("TurretMotor", TelemetryVerbosity.HIGH)
      // Power Optimization
      .withStatorCurrentLimit(Amps.of(40))
      .withClosedLoopRampRate(Seconds.of(0.25))
      .withOpenLoopRampRate(Seconds.of(0.25));
SmartMotorController       motor       = new TalonFXSWrapper(turretMotor,
                                                             DCMotor.getNEO(1),
                                                             motorConfig);

PivotConfig                m_config         = new PivotConfig(motor)
      .withStartingPosition(Degrees.of(0)) // Starting position of the Pivot
      .withWrapping(Degrees.of(0), Degrees.of(360)) // Wrapping enabled bc the pivot can spin infinitely
      .withHardLimit(Degrees.of(0), Degrees.of(720)) // Hard limit bc wiring prevents infinite spinning
      .withTelemetry("PivotExample", TelemetryVerbosity.HIGH) // Telemetry
      .withMOI(Meters.of(0.25), Pounds.of(4)); // MOI Calculation
```

## Starting Position

Starting position is different from an Arm starting position in a Pivot because 0 is arbitrary, in fact there should never be a Feedforward with a kG on a Pivot because that feedforward would take into account the position and assume 0 is parallel to the ground.&#x20;

So the starting position given is something that would only really be useful for simulation and in practice you should have an [absolute encoder attached to your turret or pivot of some kind and have a ZeroOffset applied on the Absolute Encoder](editor/#external-encoders-are-difficult). You can also use a different vendors absolute encoder and then seed the SmartMotorController with `SmartMotorController.setEncoderPosition`.

```java
PivotConfig m_config = new PivotConfig(motor)
      .withStartingPosition(Degrees.of(0)) // Starting position of the Pivot
```

## Wrapping

Wrapping is only useful for when your Pivot has an infinite turning mobility. Most pivots do not have this kind of freedom due to wiring or collisions with the frame. However limited mechanisms do, like the azimuth of a swerve module.

The start and end of the wrapping option allows you to define what happens when the motor exceeds each boundary. In the example if you are at 0deg it will set the current position to 360 and start going down. If you are at 360deg it will set the current position to 0 and start going up.

```java
PivotConfig m_config = new PivotConfig(motor)
      .withWrapping(Degrees.of(0), Degrees.of(360)) // Wrapping enabled bc the pivot can spin infinitely
```

## Hard Limits

Hard limits are useful for simulation to emulate hard limits that may or may not be imposed on your robot. It is best to give this a wide range if your mechanism is undefined that way you can see exactly what might happen and notice "that could break the robot" beforehand.

```java
PivotConfig m_config = new PivotConfig(motor)
      .withHardLimit(Degrees.of(0), Degrees.of(720)) // Hard limit bc wiring prevents infinite spinning
```

## MOI

The moment of inertia is only used in simulation to estimate the behavior of the Pivot. If you have your MOI similar to the real mechanism than the PID and FF might be similar, however this is unlikely and you will probably have to [set a sim only PID and FF](editor/simulation-only-pid-+-feedforward.md).

```java
PivotConfig m_config = new PivotConfig(motor)
      .withMOI(Meters.of(0.25), Pounds.of(4)); // MOI Calculation
```
