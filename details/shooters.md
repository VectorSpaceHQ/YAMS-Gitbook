---
icon: crosshairs-simple
---

# Shooters

## Basic Shooter Config

Given the [tutorial](../tutorials/shooter-flywheels.md) we have a basic shooter config below.

```java
SmartMotorControllerConfig smcConfig = new SmartMotorControllerConfig(this)
  .withControlMode(ControlMode.CLOSED_LOOP)
  // Feedback Constants (PID Constants)
  .withClosedLoopController(50, 0, 0, DegreesPerSecond.of(90), DegreesPerSecondPerSecond.of(45))
  .withSimClosedLoopController(50, 0, 0, DegreesPerSecond.of(90), DegreesPerSecondPerSecond.of(45))
  // Feedforward Constants
  .withFeedforward(new SimpleMotorFeedforward(0, 0, 0))
  .withSimFeedforward(new SimpleMotorFeedforward(0, 0, 0))
  // Telemetry name and verbosity level
  .withTelemetry("ShooterMotor", TelemetryVerbosity.HIGH)
  // Gearing from the motor rotor to final shaft.
  // In this example gearbox(3,4) is the same as gearbox("3:1","4:1") which corresponds to the gearbox attached to your motor.
  .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
  // Motor properties to prevent over currenting.
  .withMotorInverted(false)
  .withIdleMode(MotorMode.COAST)
  .withStatorCurrentLimit(Amps.of(40))
  .withClosedLoopRampRate(Seconds.of(0.25))
  .withOpenLoopRampRate(Seconds.of(0.25));

// Vendor motor controller object
SparkMax spark = new SparkMax(4, MotorType.kBrushless);

// Create our SmartMotorController from our Spark and config with the NEO.
SmartMotorController sparkSmartMotorController = new SparkWrapper(spark, DCMotor.getNEO(1), smcConfig);

FlyWheelConfig shooterConfig = new FlyWheelConfig(motor)
  // Diameter of the flywheel.
  .withDiameter(Inches.of(4))
  // Mass of the flywheel.
  .withMass(Pounds.of(1))
  // Maximum speed of the shooter.
  .withUpperSoftLimit(RPM.of(1000))
  // Telemetry name and verbosity for the arm.
  .withTelemetry("Shooter", TelemetryVerbosity.HIGH);

  // Shooter Mechanism
  private FlyWheel shooter = new FlyWheel(shooterConfig);
```

## Diameter and Mass

Diamater and Mass are used to find the moment of inertia for the shooter simulation.

```java
FlyWheelConfig shooterConfig = new FlyWheelConfig(motor)
  // Diameter of the flywheel.
  .withDiameter(Inches.of(4))
  // Mass of the flywheel.
  .withMass(Pounds.of(1))
```

## Soft Limits

There can be upper and lower soft limits with a shooter to prevent going so fast that a game piece could accidentally be shot and if your flywheel has enough inertia it could constantly spin to minimize the shooters ramp up time.

```java
FlyWheelConfig shooterConfig = new FlyWheelConfig(motor)
  // Maximum speed of the shooter.
  .withUpperSoftLimit(RPM.of(1000))
  .withLowerSoftLimit(RPM.of(0))
  // Equivalent to
  .withSoftLimit(RPM.of(0), RPM.of(1000));
```
