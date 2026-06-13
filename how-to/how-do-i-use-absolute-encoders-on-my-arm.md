# How do I use absolute encoders on my Arm?

## What are absolute encoders?

Absolute Encoders are encoders which read one rotation and persists between power cycles and reboots.&#x20;

{% embed url="https://www.revrobotics.com/rev-11-1271/" %}

{% embed url="https://store.ctr-electronics.com/products/cancoder" %}

{% embed url="https://store.ctr-electronics.com/products/wcp-throughbore-encoder" %}

## External Encoders&#x20;

`SmartMotorController` s   are only able to use the same encoders as the brand of the `SmartMotorController`. If you are using the wrong brand YAMS will through an exception.

## Limitations of Absolute Encoders

Absolute encoders can only read from 0 rotations to 1 rotation. If you traverse the discontinuity point it will wrap back around. It does not travel past 1!

{% stepper %}
{% step %}
### Check your inversions!

Using the hardware client of the vendor, apply positive power and graph the position of your absolute encoder. Your absolute encoder should increase with positive power

If it needs to be inverted you can use `SmartMotorControllerConfig.withExternalEncoderInverted(true)`
{% endstep %}

{% step %}
### Check your gear ratio and mount!

Using the vendor hardware client ensure that the absolute encoder **NEVER** goes beyond 1 rotation. You will see that it goes beyond 1 rotation by the position wraps around from 1rotation to 0rotations. If this happens more than once the gear ratio is invalid and your `SmartMotorController` cannot use the absolute encoder as the primary feedback device.

You can use `SmartMotorControllerConfig.withExternalEncoderGearing(1.0)` to set the reduction gear ratio on the absolute encoder.

{% hint style="warning" %}
This could also be caused when the absolute encoders discontinuity point is within the range of movement, to solve this take the absolute encoder off and rotate the bearing to ensure it never reaches that discontinuity point during the range of motion.
{% endhint %}
{% endstep %}

{% step %}
### Find your zero

Using the hardware client of the vendor find the zero value of the absolute encoder corresponding the the arm being completely horizontal from the ground.

You can set the horizontal zero using `SmartMotorControllerConfig.withExternalEncoderZeroOffset(Degrees.of(offset_degrees))`&#x20;
{% endstep %}

{% step %}
### Bring it all together!

<pre class="language-java"><code class="lang-java">private final SparkMax                   armMotor         = new SparkMax(1, MotorType.kBrushless);

private final SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
      .withClosedLoopController(4, 0, 0)
      .withSoftLimit(Degrees.of(-30), Degrees.of(100))
      .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
      .withIdleMode(MotorMode.BRAKE)
      .withTelemetry("ArmMotor", TelemetryVerbosity.HIGH)
      .withStatorCurrentLimit(Amps.of(40))
      .withMotorInverted(false)
      .withClosedLoopRampRate(Seconds.of(0.25))
      .withOpenLoopRampRate(Seconds.of(0.25))
      .withFeedforward(new ArmFeedforward(0, 0, 0, 0))
      .withControlMode(ControlMode.CLOSED_LOOP)
<strong>      .withExternalEncoder(armMotor.getAbsoluteEncoder())
</strong><strong>      .withExternalEncoderInverted(false)
</strong><strong>      .withExternalEncoderGearing(1)
</strong><strong>      .withExternalEncoderZeroOffset(Degrees.of(33.25))
</strong><strong>      .withUseExternalFeedbackEncoder(true);
</strong>      
private final SmartMotorController       motor            = new SparkWrapper(armMotor,
                                                                               DCMotor.getNEO(1),
                                                                               motorConfig);
</code></pre>
{% endstep %}
{% endstepper %}
