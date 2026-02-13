# How to disable linear closed loop control?

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption><p>Measurement, Mechanism, and Rotor units</p></figcaption></figure>

## What is linear closed loop control?

Linear closed loop control is when the closed loop controller operate in `meters` and `meters per second` instead of `rotations` and `rotations per second`. This directly affects your PID gains.&#x20;

## When would I want to disable it?

Sometimes you would like to get `Measurement` (like `SmartMotorController.getMeasurementPosition` and `SmartMotorController.getMeasurementVelocity`) data without enabling linear closed loop control.&#x20;

* Measuring shooter speed in `meters per second` for easier shoot on the move calculation
* Supplying linear velocity or data which is translated to angle's or angular velocities inside of the `SmartMotorController`.

## How do I disable it?

Inside of your `SmartMotorControllerConfig` you would use `withLinearClosedLoopController(false)`

<pre class="language-java"><code class="lang-java">SmartMotorControllerConfig motorConfig        = new SmartMotorControllerConfig(this)
      .withControlMode(ControlMode.CLOSED_LOOP)
      .withWheelDiameter(Inches.of(4))
      .withIdleMode(MotorMode.COAST)
      .withClosedLoopController(50,0,0,MetersPerSecond.of(3), MetersPerSecondPerSecond.of(5))
<strong>      .withLinearClosedLoopController(false);
</strong></code></pre>

