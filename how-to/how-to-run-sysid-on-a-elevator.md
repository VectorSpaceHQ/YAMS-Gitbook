# How to run SysId on a Elevator?

{% hint style="danger" %}
This guide will only go over how to use SysId on a Spark or Nova Motor Controller.&#x20;

The methodology behind TalonFX and TalonFXS is slightly different.
{% endhint %}

## Premise

* Already configured the soft and hard limits
* Gearing is correct

SysId is documented in WPILib and we should follow their documentation as close as possible.

{% embed url="https://docs.wpilib.org/en/stable/docs/software/advanced-controls/system-identification/introduction.html" %}

## Creating the SysId Command

<pre class="language-java" data-title="ExampleSubsystem.java" data-line-numbers><code class="lang-java">// Copyright (c) FIRST and other WPILib contributors.
// Open Source Software; you can modify and/or share it under the terms of
// the WPILib BSD license file in the root directory of this project.

package frc.robot.subsystems;

import static edu.wpi.first.units.Units.Amps;
import static edu.wpi.first.units.Units.Meters;
import static edu.wpi.first.units.Units.MetersPerSecond;
import static edu.wpi.first.units.Units.MetersPerSecondPerSecond;
import static edu.wpi.first.units.Units.Feet;
import static edu.wpi.first.units.Units.Pounds;
<strong>import static edu.wpi.first.units.Units.Second;
</strong><strong>import static edu.wpi.first.units.Units.Seconds;
</strong><strong>import static edu.wpi.first.units.Units.Volts;
</strong>
import com.revrobotics.spark.SparkLowLevel.MotorType;
import com.revrobotics.spark.SparkMax;

import edu.wpi.first.math.controller.ElevatorFeedforward;
import edu.wpi.first.math.system.plant.DCMotor;
<strong>import edu.wpi.first.units.measure.Distance;
</strong>import edu.wpi.first.wpilibj2.command.Command;
import edu.wpi.first.wpilibj2.command.SubsystemBase;
import yams.mechanisms.SmartMechanism;
import yams.mechanisms.config.ElevatorConfig;
import yams.mechanisms.positional.Elevator;
import yams.motorcontrollers.SmartMotorController;
import yams.motorcontrollers.SmartMotorControllerConfig;
import yams.motorcontrollers.SmartMotorControllerConfig.ControlMode;
import yams.motorcontrollers.SmartMotorControllerConfig.MotorMode;
import yams.motorcontrollers.SmartMotorControllerConfig.TelemetryVerbosity;
import yams.motorcontrollers.local.SparkWrapper;
import yams.gearing.GearBox;
import yams.gearing.MechanismGearing;

public class ExampleSubsystem extends SubsystemBase {

  private SmartMotorControllerConfig smcConfig = new SmartMotorControllerConfig(this)
  .withControlMode(ControlMode.CLOSED_LOOP)
  // Mechanism Circumference is the distance traveled by each mechanism rotation converting rotations to meters.
  .withMechanismCircumference(Meters.of(Inches.of(0.25).in(Meters) * 22))
  // Feedback Constants (PID Constants)
  .withClosedLoopController(4, 0, 0, MetersPerSecond.of(0.5), MetersPerSecondPerSecond.of(0.5))
  .withSimClosedLoopController(4, 0, 0, MetersPerSecond.of(0.5), MetersPerSecondPerSecond.of(0.5))
  // Feedforward Constants
  .withFeedforward(new ElevatorFeedforward(0, 0, 0))
  .withSimFeedforward(new ElevatorFeedforward(0, 0, 0))
  // Telemetry name and verbosity level
  .withTelemetry("ElevatorMotor", TelemetryVerbosity.HIGH)
  // Gearing from the motor rotor to final shaft.
  // In this example gearbox(3,4) is the same as gearbox("3:1","4:1") which corresponds to the gearbox attached to your motor.
  .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
  // Motor properties to prevent over currenting.
  .withMotorInverted(false)
  .withIdleMode(MotorMode.BRAKE)
  .withStatorCurrentLimit(Amps.of(40))
  .withClosedLoopRampRate(Seconds.of(0.25))
  .withOpenLoopRampRate(Seconds.of(0.25));

  // Vendor motor controller object
  private SparkMax spark = new SparkMax(4, MotorType.kBrushless);

  // Create our SmartMotorController from our Spark and config with the NEO.
  private SmartMotorController sparkSmartMotorController = new SparkWrapper(spark, DCMotor.getNEO(1), smcConfig);

  private ElevatorConfig elevconfig = new ElevatorConfig(sparkSmartMotorController)
      .withStartingHeight(Meters.of(0.5))
      .withHardLimits(Meters.of(0), Meters.of(3))
      .withTelemetry("Elevator", TelemetryVerbosity.HIGH)
      .withMass(Pounds.of(16));

  // Elevator Mechanism
  private Elevator elevator = new Elevator(elevconfig);

  /**
   * Set the height of the elevator.
   * @param angle Distance to go to.
   */
  public Command setHeight(Distance height) { return elevator.setHeight(height);}

<strong>  /**
</strong><strong>   * Move the elevator up and down.
</strong><strong>   * @param dutycycle [-1, 1] speed to set the elevator too.
</strong><strong>   */
</strong><strong>  public Command set(double dutycycle) { return elevator.set(dutycycle);}
</strong>
<strong>  /**
</strong><strong>   * Run sysId on the {@link Elevator}
</strong><strong>   */
</strong><strong>  public Command sysId() { return elevator.sysId(Volts.of(7), Volts.of(2).per(Second), Seconds.of(4));}
</strong>
  /** Creates a new ExampleSubsystem. */
  public ExampleSubsystem() {}

  @Override
  public void periodic() {
    // This method will be called once per scheduler run
    elevator.updateTelemetry();
  }

  @Override
  public void simulationPeriodic() {
    // This method will be called once per scheduler run during simulation
    elevator.simIterate();
  }
}

</code></pre>

## Binding Buttons to SysId

<pre class="language-java" data-title="RobotContainer.java" data-line-numbers><code class="lang-java">// Copyright (c) FIRST and other WPILib contributors.
// Open Source Software; you can modify and/or share it under the terms of
// the WPILib BSD license file in the root directory of this project.

package frc.robot;

import frc.robot.Constants.OperatorConstants;
import frc.robot.commands.Autos;
import frc.robot.commands.ExampleCommand;
import frc.robot.subsystems.ExampleSubsystem;

<strong>import static edu.wpi.first.units.Units.Degrees;
</strong>
import edu.wpi.first.wpilibj2.command.Command;
import edu.wpi.first.wpilibj2.command.button.CommandXboxController;
import edu.wpi.first.wpilibj2.command.button.Trigger;

/**
 * This class is where the bulk of the robot should be declared. Since Command-based is a
 * "declarative" paradigm, very little robot logic should actually be handled in the {@link Robot}
 * periodic methods (other than the scheduler calls). Instead, the structure of the robot (including
 * subsystems, commands, and trigger mappings) should be declared here.
 */
public class RobotContainer {
  // The robot's subsystems and commands are defined here...
  private final ExampleSubsystem m_exampleSubsystem = new ExampleSubsystem();

  // Replace with CommandPS4Controller or CommandJoystick if needed
  private final CommandXboxController m_driverController =
      new CommandXboxController(OperatorConstants.kDriverControllerPort);

  /** The container for the robot. Contains subsystems, OI devices, and commands. */
  public RobotContainer() {
    // Configure the trigger bindings
    configureBindings();

<strong>    // Set the default command to force the elevator to release.
</strong><strong>    m_exampleSubsystem.setDefaultCommand(m_exampleSubsystem.set(0));
</strong>  }

  /**
   * Use this method to define your trigger->command mappings. Triggers can be created via the
   * {@link Trigger#Trigger(java.util.function.BooleanSupplier)} constructor with an arbitrary
   * predicate, or via the named factories in {@link
   * edu.wpi.first.wpilibj2.command.button.CommandGenericHID}'s subclasses for {@link
   * CommandXboxController Xbox}/{@link edu.wpi.first.wpilibj2.command.button.CommandPS4Controller
   * PS4} controllers or {@link edu.wpi.first.wpilibj2.command.button.CommandJoystick Flight
   * joysticks}.
   */
  private void configureBindings() {
    
<strong>    // Schedule `sysId` when the Xbox controller's A button is pressed,
</strong><strong>    // cancelling on release.
</strong><strong>    m_driverController.a().whileTrue(m_exampleSubsystem.sysId());
</strong><strong>    m_driverController.b().whileTrue(m_exampleSubsystem.set(0));
</strong><strong>    // Schedule `set` when the Xbox controller's B button is pressed,
</strong><strong>    // cancelling on release.
</strong><strong>    m_driverController.x().whileTrue(m_exampleSubsystem.set(0.3));
</strong><strong>    m_driverController.y().whileTrue(m_exampleSubsystem.set(-0.3));
</strong>
  }

  /**
   * Use this to pass the autonomous command to the main {@link Robot} class.
   *
   * @return the command to run in autonomous
   */
  public Command getAutonomousCommand() {
    // An example command will be run in autonomous
    return Autos.exampleAuto(m_exampleSubsystem);
  }
}

</code></pre>

## Continue along with WPILib!

{% embed url="https://docs.wpilib.org/en/stable/docs/software/advanced-controls/system-identification/loading-data.html" %}
