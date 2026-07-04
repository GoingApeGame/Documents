# Animation Workflow

This workflow is going to be a little scuffed no matter what. As far as I can tell, there is no perfect way to handle weapon animation in Roblox, so the goal is to find the easiest workflow that produces the results we need while remaining manageable for animators.

## Weapon Animation

Each weapon needs a base weapon-holding pose that can be used as the foundation for all of its animations.

To create this pose, we use a rig holding the weapon with Roblox `IKControl` instances. The IK setup allows us to position the weapon first and have the character's arms naturally solve toward the weapon's grip points. We can then bake the resulting arm pose into an explicit animation.

### Creating the Base Weapon Pose

* Create the character rig and attach the weapon to the character's upper torso. The weapon joint should have no positional offset by default.
* Open the rig in the Animation Editor and load the monkey's normal idle animation. This animation should not contain any weapon-holding pose.
* Create a new animation for the weapon.
* Position the weapon where it should be held relative to the character's torso.
* Before saving the weapon animation, remove all unrelated keyframes inherited from the character idle animation. The resulting animation should contain only the joints required to position the weapon.

At this point, we should have two animations:

* The character's normal idle animation.
* The weapon offset animation.

### Generating the Arm Pose

The rig is then loaded in Studio with both animations playing simultaneously:

* The normal character idle animation.
* The weapon offset animation.

The rig's `IKControl` instances should be enabled. Because the IK targets are attached to the weapon, the character's arms should automatically solve toward the weapon's grip points.

At this stage, the weapon is explicitly animated while the arms are being procedurally positioned using IK.

We then use an **Animation Extractor** to generate a new `KeyframeSequence` containing the arm transforms produced by the IK setup.

The extracted animation should contain the resulting transforms for:

* Upper arms.
* Lower arms.
* Hands.

This effectively **bakes the IK-generated arm pose into an explicit animation**.

At this point, we have three animations:

* The character idle animation.
* The weapon offset animation.
* The extracted arm animation generated from the IK pose.

### Merging the Base Animation

The three animations are merged into a single `KeyframeSequence`.

The resulting animation contains:

* The character's normal idle pose.
* The weapon's authored position relative to the character.
* The baked arm pose generated from the IK setup.

This merged animation becomes the **base weapon animation** for that specific weapon.

The base animation can then be sent to the animator and used as the foundation for reloads, firing animations, inspections, and other weapon-specific animations.

The important part is that the animator begins with a pose that already closely matches the weapon's runtime IK setup. This should minimize visible snapping when transitioning between explicit animation and runtime IK.

## Animation Graph Masking

The weapon animation layer in the Animation Graph should only affect the joints required for weapon animation.

The mask should include:

* Upper arms.
* Lower arms.
* Hands.
* Weapon root.
* All child joints of the weapon root.

The rest of the character remains controlled by the normal character animation graph.

Runtime IK can then be used during weapon-holding states to maintain hand contact with the weapon as procedural offsets are applied to the weapon root.

For animations such as reloads, runtime IK can be disabled and the baked weapon animation can take full control of the arms and weapon.
