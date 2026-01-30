# flip-csci526-2025fall
[中文版本](./README_zh.md)

Game link: https://lagrange151235.github.io/CSCI526/FinalProject/flip_v.0.10.4_with_art_preview/
## Magnetism Effect Implementation
- The implementation of magnetism effect is located at `./Assets/Scripts/Magnetism.cs`
  - Enumeration Types
    - MagnetismAxis: The axis of magnetic force, with optional values ({None, X, Y})
    - MagnetismMode: The mode of magnetic effect, with optional values ({None, Radial, Axial})
      - Radial: The force direction is along the line connecting the centers of the two objects
      - Axial: The force direction is along the axis specified by MagnetismAxis
    - MagneticPole: The magnetic polarity, with optional values ({None, North, South})
  - Parameter List
    - [float] maxMagnetismStrength: Maximum magnetic force strength
    - [float] minMagnetismStrength: Minimum magnetic force strength
    - [bool] changeableMode: Whether the current magnetic effect mode (`currentMode`) is allowed to change automatically or not, used to implement different magnetic effects
    - [MagneticPole] currentPole: Current magnetic polarity
    - [MagnetismMode] currentMode: Current magnetic effect mode
    - [MagnetismAxis] currentAxis: Current magnetic force axis
    - [List<Rigidbody2D>] attractedObjects: A list of objects' Rigidbody2D inside the Collider that limits the range of magnetic force
    - [List<GameObject>] collidingMagnets: A list of GameObjects that are in contact with current object

  - Explanation of Core Functions
    - UpdateColor: Updates the object's color according to its magnetic polarity
    - UpdateMagnetism: Updates the object's magnetic effect
      1. The magnetic effect is only applied to specific objects
          1. The magnetic force does not affect itself
          2. The magnetic force does not affect objects without a "Player" tag
      2. The magnetic effect is applied within a specific Collider
          1. For object with this script attached, a specific Collider component is added and its 'Is Trigger' property is set to True. The bounds of this Collider limit the range within which the magnetic effect is active
          2. Within the Collider's bounds, the magnetic force magnitude is limited to the range [minMagnetismStrength, maxMagnetismStrength]. The magnitude decays with the square of the distance
        ```csharp
        float currentMagnetismStrength = maxMagnetismStrength;
        float distance = direction.magnitude;
        currentMagnetismStrength /= (distance * distance);
        currentMagnetismStrength = Mathf.Max(minMagnetismStrength, Mathf.Min(currentMagnetismStrength, maxMagnetismStrength));
        ``` 
      3. Radial Magnetism/Axial Magnetism Switching Mechanism
          1. The design of radial magnetism is helpful for avoiding weird physical effects caused by changes in the magnetic force direction, but it also leads to an unnatural magnetic effect when the two objects are closely attached. Consider the following scenario --> A magnetic platform moves along the X-axis, and we want the Player to be able to move along the Y-axis and be attracted to the magnetic platform by switching polarity. Once attached, they move together along the X-axis. The following two problems exist:
             - Before the two objects are attached, if radial magnetism is used, the Player will move abnormally along the X-axis
             - When the two objects are attached, if Y-axis axial magnetism is used, the Player cannot move normally along the X-axis
          2. Switching Mechanism Design: Allows an object with axial magnetism to switch to radial magnetism after contact, thus solving the aforementioned problems

- The implementation of Player's magnetic polarity switching is located at `./Assets/Scripts/PlayerMagnetismController.cs`. Pressing the `shift` key to switch the polarity (if currently South, it switches to North, and vice versa). The Player's magnetic polarity is initialized to South

## Friction Effect Implementation
- Purpose of implementing friction: To make the Player's speed gradually decay when moving on a platform, preventing excessive displacement of the Player
- The code for the friction effect implementation is located at `./Assets/Scripts/Friction.cs`
- Add a resistive force in the opposite direction of the Player's movement when the Player is in contact with and moving on the designated platform
```csharp
Vector2 frictionDirection = -otherRb.linearVelocity.normalized;
float frictionMagnitude = frictionStrength;
otherRb.AddForce(frictionDirection * frictionMagnitude, ForceMode2D.Force);
```
