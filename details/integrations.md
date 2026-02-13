---
icon: plug-circle-plus
---

# Live Tuning

## What is Live Tuning?

Live tuning allows you to tune your closed loop controller and feedforward easily through NetworkTables using AdvantageScope or Elastic.

Live Tuning is enabled when any tunable setpoint is enabled with `TelemetryVerbosity.HIGH`

<pre class="language-java"><code class="lang-java">SmartMotorControllerConfig lowerFlyWheelConfig = new SmartMotorControllerConfig(this)
      .withControlMode(ControlMode.CLOSED_LOOP)
      .withIdleMode(MotorMode.COAST)
      .withGearing(new MechanismGearing(GearBox.fromReductionStages(3, 4)))
      .withMomentOfInertia(Inches.of(4), Pounds.of(2))
      .withClosedLoopController(1,
                                0,
                                0) // You generally do not want a profile because its not a position controlled loop.
      .withFeedforward(new SimpleMotorFeedforward(0, 0, 0)) // Helps track changing RPM goals
      .withMotorInverted(false)
<strong>      .withTelemetry("LowerFlyWheel", SmartMotorControllerConfig.TelemetryVerbosity.HIGH)
</strong></code></pre>

## How do I use Live Tuning?

{% hint style="danger" %}
Live Tuning can be **DANGEROUS** please test in sim before the real robot to understand the implications and dangers you will experience on the real robot!
{% endhint %}

To enable Live Tuning drag the `Live Tuning` command in from `NT:/SmartDashboard/MECHANISM_NAME/Commands/SUBSYSTEM_NAME/Live Tuning` with Elastic.

After ensuring the robot is enabled in you can press the button to enable Live Tuning!

{% hint style="warning" %}
The button will run the `Live Tuning` command which can be interrupted by the controller or any other triggers for that subsystem in your program!
{% endhint %}

<figure><img src="../.gitbook/assets/new_liveutinging.gif" alt=""><figcaption></figcaption></figure>

Now you can tune your mechanism while your robot is enabled!

<figure><img src="../.gitbook/assets/AScopeLiveTuningExample.gif" alt=""><figcaption></figcaption></figure>

## When should I NOT use Live Tuning?

You should not use Live Tuning if you have not tested it in sim yet. It is ALWAYS better to know what you're getting yourself into!
