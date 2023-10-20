<!-- omit in toc -->

# 12 Lead Trainer EKG Generator DLL

**Version: 1.0**

**Updated: 09/20/2020**

This document outlines the available function calls available through the 12 Lead Trainer EKG Generator DLL.

## Outline

- [Outline](#outline)
- [`public class Generator`](#public-class-generator)
  - [Constructors](#constructors)
  - [`UpdateRhythm`](#updaterhythm)
    - [`void UpdateRhythm(WaveProperty prop, double value)`](#void-updaterhythmwaveproperty-prop-double-value)
    - [`void UpdateRhythm(Dictionary<WaveProperty, double> properties)`](#void-updaterhythmdictionarywaveproperty-double-properties)
    - [`void UpdateRhythm(WaveDefinition definition)`](#void-updaterhythmwavedefinition-definition)
  - [`void UpdateLayout(EKGLead newLayout)`](#void-updatelayoutekglead-newlayout)
  - [`void UpdateNoise(NoiseProfile noise)`](#void-updatenoisenoiseprofile-noise)
  - [`double GetVoltages(double duration_in_ms, double interval_in_ms?)`](#double-getvoltagesdouble-duration_in_ms-double-interval_in_ms)
  - [`Randomize()`](#randomize)

## `public class Generator`

The `Generator` class will act as a factory to produce EKG rhythms. It can be instantiated with a lead layout and/or a rhythm.

Each EKG rhythm displayed simultaneously will need its own `Generator` defined to support it.

### Constructors

`Generator(layout, rhythm)`

This constructor takes in two optional parameters, the rhythm you wish to display, and the layout of the leads you wish to generate. The rhythm parameter can take on one of two forms, a dedicated WaveDefinition object, or a Dictionary of Waveproperties and their values.

<!-- omit in toc -->

#### Parameters

1. `layout[][]` - A jagged array of EKGLeads that will specify what leads should be added to the EKG and where. These leads can be passed in as an array of EKGLead objects or a preset lead layout.

    |Type|Required|Default|
    |--|--|--|
    |EKGLead[][]|No|EKGLayoutPresets.Layout_12Lead|

1. `rhythm` - the rhythm that the generator will initialize to. If none is specified, a Normal Sinus rhythm is generated.

    |Type|Required|Default|
    |--|--|--|
    |WaveDefinition *or* Dictionary<WaveProperty, double>|No|Rhythms.NormalSinus|

<!-- omit in toc -->

#### Examples

<details><summary>Using a preset rhythm</summary>

```c#
new Generator(Rhythms.AtrialTachycardia)
```

</details>

<details><summary>Using a preset layouts</summary>

```c#
Generator gen = new Generator(EKGLayoutPresets.Layout_12Lead_Without_Rhythm_Strip);
```

</details>

<details><summary>Using a preset rhythm with a preset layout</summary>

```c#
Generator gen = new Generator(EKGLayoutPresets.Layout_12Lead_Without_Rhythm_Strip, Rhythms.AtrialTachycardia);
```

</details>

<details><summary>Passing in a custom heart rate</summary>

```c#
//default rhythm is Normal Sinus, so we just have to increase the heart rate to get Sinus Tachycardia
Dictionary<WaveProperty, double> sinusTachycardiaProperties = new Dictionary<WaveProperty, double>();
sinusTachycardiaProperties.Add(WaveProperty.Heart_Rate, 150);

Generator gen = new Generator(sinusTachycardiaProperties);
```

</details>

<details><summary>Using an array of preset leads to specify a layout</summary>

```c#
EKGLead[] row1 = new EKGLead[] { EKGLeadPresets.LeadI, EKGLeadPresets.LeadAVR, EKGLeadPresets.LeadV1, EKGLeadPresets.LeadV4 };
EKGLead[] row2 = new EKGLead[] { EKGLeadPresets.LeadII, EKGLeadPresets.LeadAVL, EKGLeadPresets.LeadV2, EKGLeadPresets.LeadV5 };
EKGLead[] row3 = new EKGLead[] { EKGLeadPresets.LeadIII, EKGLeadPresets.LeadAVF, EKGLeadPresets.LeadV3, EKGLeadPresets.LeadV6 };
[] row4 = new EKGLead[] { EKGLeadPresets.LeadII },

EKGLead[][] layout = new EKGLead[][] {
    row1,
    row2,
    row3,
    row4
};
Generator gen = new Generator(layout);
```

</details>

<details><summary>Creating a layout with custom lead definitions</summary>

```c#
EKGLead firstLead = new EKGLead(60, 0); /* 60 degree axis with no precordial angle */
EKGLead secondLead = new EKGLead(30, 30);/* 30 degree axis with a 30 degree precordial angle */
EKGLead[][] layout = new EKGLead[][] { new EKGLead[] { firstLead, secondLead } };
Generator gen = new Generator(layout);
```

</details>

### `UpdateRhythm`

The UpdateRhythm function allows you to set or modify the property values of the currently displaying rhythm, or load an entirely new rhythm.

#### `void UpdateRhythm(WaveProperty prop, double value)`

This override is designed in order for the user to update individual wave properties, i.e. for a button.

<!-- omit in toc -->
##### Parameters

1. `property` - The wave property we are wishing to modify on the currently displaying rhythm

    |Type|Required|Default|
    |--|--|--|
    |WaveProperty|Yes|N/A|

2. `value` - The double value that we want to set the specified property to

    |Type|Required|Default|
    |--|--|--|
    |double|Yes|N/A|

<!-- omit in toc -->

##### Examples

<details><summary>Turning a normal sinus rhythm into a sinus tachycardia</summary>

```c#
//default rhythm is a 12-lead normal sinus
Generator gen = new Generator();
gen.UpdateRhythm(WaveProperty.Heart_Rate, 150);
```

</details>

#### `void UpdateRhythm(Dictionary<WaveProperty, double> properties)`

This override is provided in for the user to be able to update a set of properties on a rhythm all at once, i.e. to simulate a rhythm suddenly transitioning in response to a stimuli.

<!-- omit in toc -->

##### Parameters

1. `properties` - A dictionary of wave properties and their desired values

    |Type|Required|Default|
    |--|--|--|
    |Dictionary<WaveProperty, double>|Yes|N/A|

<!-- omit in toc -->

##### Examples

<details><summary>Simulate multiple changes in the EKG at once</summary>

```c#
//default rhythm is a 12-lead normal sinus
Generator gen = new Generator();

Dictionary<WaveProperty, double> patientResponse = new Dictionary<WaveProperty, double>();
patientResponse.Add(WaveProperty.Heart_Rate, 130);
patientResponse.Add(WaveProperty.PR_Segment_Width, 102);
patientResponse.Add(WaveProperty.ST_Segment_Width, 123);
patientResponse.Add(WaveProperty.R_Wave_Height, 43);

gen.UpdateRhythm(patientResponse);
```

</details>

#### `void UpdateRhythm(WaveDefinition definition)`

This override is provided for the user to completely swap which rhythm is being displayed by the generator, i.e. to allow the user a list of waves to select to load into the Generator.

<!-- omit in toc -->

##### Parameters

1. `definition` - The desired wave for the generator to display

    |Type|Required|Default|
    |--|--|--|
    |WaveDefinition|Yes|N/A|

<!-- omit in toc -->

##### Examples

<details><summary>Changing the displayed wave based on user input</summary>

```c#
string desiredWave = getUserInput();

Generator gen = new Generator();
switch(desiredWave) {
    case "Hyperkalimia":
        gen.UpdateRhythm(Rhythms.Hyperkalemia);
        break;

    case "Sinus Block":
        gen.UpdateRhythm(Rhythms.SinusBlock);
        break;

    default:
        //the generator defaults to normal sinus, so no action is needed
        print("Wave not recognized, defaulting to Normal Sinus");
        break;
}

```
</details>

<details><summary>Retrieve the current wave definition being used by the generator</summary>

```c#
Generator gen = new Generator();
gen.UpdateRhythm(Rhythms.Hyperkalemia);

WaveDefinition currentDefinition = gen.DisplayedWaveDefinition;
```

</details>

### `void UpdateLayout(EKGLead[][] newLayout)`

The UpdateLayout function allows the user to change the current layout of the EKG, i.e. from a standard 12 Lead to a stacked 12 Lead, a 15 Lead, etc...

This function is solely used to change which Lead's values are returned and at what index from `GetVoltages`

<!-- omit in toc -->

#### Parameters

1. `newLayout` - A jagged array of EKGLeads that define the desired layout of the EKG.

    |Type|Required|Default|
    |--|--|--|
    |Dictionary<WaveProperty, double>|Yes|N/A|

<!-- omit in toc -->

#### Examples

<details><summary>Setting the layout to 12 Lead with no Rhythm Strip using the presets</summary>

```c#
Generator gen = new Generator();
gen.UpdateLayout(EKGLayoutPresets.Layout_12Lead_Without_Rhythm_Strip);
```

</details>

<details><summary>Setting the layout to 12 Lead with no Rhythm Strip using the preset lead definitions</summary>

```c#
Generator gen = new Generator();

EKGLayout[][] layout = new EKGLead[][] {
    new EKGLead[] { EKGLeadPresets.LeadI, EKGLeadPresets.LeadAVR, EKGLeadPresets.LeadV1, EKGLeadPresets.LeadV4 },
    new EKGLead[] { EKGLeadPresets.LeadII, EKGLeadPresets.LeadAVL, EKGLeadPresets.LeadV2, EKGLeadPresets.LeadV5 },
    new EKGLead[] { EKGLeadPresets.LeadIII, EKGLeadPresets.LeadAVF, EKGLeadPresets.LeadV3, EKGLeadPresets.LeadV6 },
}

gen.UpdateLayout(layout);
```

</details>

### `void UpdateNoise(NoiseProfile noise)`

The UpdateNoise function allows a user to change how much noise is currently being displayed on a rhythm without modifying the rhythm itself, i.e. to simulate an EKG being taken on the back of a moving truck vs. a patient on the ground.

<!-- omit in toc -->

#### Parameters

1. `noise` - The NoiseProfile preset that specifies the desired noise to be applied to the wave.

    |Type|Required|Default|
    |--|--|--|
    |NoiseProfile|Yes|N/A|

<!-- omit in toc -->

#### Examples

<details><summary>Removing all noise from an EKG and viewing the underlying rhythm</summary>

```c#
Generator gen = new Generator();
gen.UpdateNoise(NoiseSelector.NoNoise);
```

</details>

### `double[][] GetVoltages(double duration_in_ms, double interval_in_ms?)`

The GetVoltages function allows the user to generate the EKG voltages given the current setting of the generator. This function is designed to be used to generate a set amount of an EKG rhythm, i.e. a standard 12 lead would call this function with 10000ms (10 Seconds).

<!-- omit in toc -->

#### Parameters

1. `duration_in_ms` - The amount of generated EKG desired.

    |Type|Required|Default|
    |--|--|--|
    |double|Yes|N/A|

1. `interval_in_ms` - The frequency of the data points returned, i.e. if a 1000ms duration is selected with an interval of 10ms, 100 data points will be returned that should be displayed with 10ms between them. A lower interval will be more precise but take longer to generate.

    |Type|Required|Default|
    |--|--|--|
    |double|No|3|

<!-- omit in toc -->

#### Output

1. `double[][]` - A jagged array of doubles of `duration_in_ms`/`interval_in_ms` length. A row of output is generated for each row in the EKG Layout specified by `UpdateLayout`.

    If there are 4 leads in a layout, each lead will take up 1/4 of the output, i.e. 10 seconds of generation with a layout of:

    ```c#
    EKGLayout[][] layout = new EKGLead[][] {
        new EKGLead[] { EKGLeadPresets.LeadI, EKGLeadPresets.LeadAVR, EKGLeadPresets.LeadV1, EKGLeadPresets.LeadV4 },
        new EKGLead[] { EKGLeadPresets.LeadII, EKGLeadPresets.LeadAVL, EKGLeadPresets.LeadV2, EKGLeadPresets.LeadV5 },
        new EKGLead[] { EKGLeadPresets.LeadIII, EKGLeadPresets.LeadAVF, EKGLeadPresets.LeadV3, EKGLeadPresets.LeadV6 },
        new EKGLead[] { EKGLeadPresets.LeadII },
    }
    ```

    Will result in an output of:

    ```c#
    [
        [LeadI, LeadAVR, LeadV1, LeadV4], // 2.5 seconds of each
        [LeadII, LeadAVL, LeadV2, LeadV5], // 2.5 seconds of each
        [LeadIII, LeadAVF, LeadV3, LeadV6], // 2.5 seconds of each
        [LeadII], // 10 seconds of Lead II
    ]

<!-- omit in toc -->

#### Examples

<details><summary>Generating 10 seconds of normal sinus with a voltage data point every 10 milliseconds</summary>

```c#
const _10Seconds = 10 * 1000;
Generator gen = new Generator();
double[][] voltages = gen.GetVoltages(_10Seconds, 10);
```

</details>

<details><summary>Generating a single data point for a live update</summary>

```c#
const refreshInterval = 1000 / 60; //60 fps
Generator gen = new Generator();

//set the layout to be just a single rhythm strip
gen.setLayout(new EKGLead[][] { new EKGLead[] { EKGLeadPresets.LeadII } });

//get a single data point

DateTime start = DateTime.Now;
while(true) {
    DateTime now = DateTime.Now;
    TimeSpan diff = start - DateTime.Now;
    if(diff >= refreshInterval) {
        double[][] voltages = gen.GetVoltages(diff.TotalMilliseconds, diff.TotalMilliseconds);
        drawRhythmFunction(voltages);
        start = now;
    }
}

```

</details>

### `Randomize()`

The Randomize function will randomize a new wave of the last chosen rhythm preset. This can be used to sample through multiple waves until the user finds one they like.

<!-- omit in toc -->

#### Examples

<details><summary>Randomizing a rhythm</summary>

```c#
Generator gen = new Generator(Rhythms.PacedAtrial);

//generate a new Paced Atrial rhythm as long as the user tells us to
while(getUserInput().equals(Controls.RandomizeKey)) {
    gen.Randomize();
}
```

</details>
