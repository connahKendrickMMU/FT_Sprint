# FT_Sprint
 This focus topic is based on a simple sprint mechanic. Before you begin this project you should play about with this [repo](https://atomechllc.github.io/jump-test-classic/), to see how slight changes in code gives different gameplay styles.

## Overview

This project involves implementing a sprint mechanic, but it currently lacks input for initiating sprints. The goal is to create and assign an input action to enable sprinting:
- Create the input action.
- Update the input mapping context.

From this you will learn about input context mapping, function binding in C++ and some basic C++ logic. You can also see i have added a common style gate way mini game for you to play about with the mechanic you are making and could expand upon it.

## Where to find the relevant scripts

We have two key components you need to place for this, input management components and the C++ scripts for the character. 

### Inputs 

We should always follow a specific pattern for organising our files. As we only have one character, we can just store all the inputs in one folder: **Content->ThirdPerson->Input**
You can see it already has the **Input Mapping Context**, and the **Input Actions** are all in the **Actions** folder. 

![The location of the input folder in the Unreal Engine Content Browser.](Hints/InputMappingLocation.png)

### Scripts

From now on all our script will be stored in the **C++ Classes** folder with the name of the project so for us **C++ Classes->FT_Sprint**. We have the main character scripts here, and a **Public** folder which contains the scripts for the gates. 

![The location of the C++ scripts in the Unreal Engine Content Browser.](Hints/ScriptLocations.png)

## Task
The task is to implement the sprint input and rebind the inputs to allow the player to have a sprint functionality.

## Hints 

### Generate Visual Studio Project
Check the scripts open properly, if in visual studio you open the script and the solution explorer is blank, and the tab in the top left says miscellaneous files, you need to go to the .uproject file **right click->Show more options->Generate Visual Studio Solution**.

### Coding
Remember, the header file declares variables and methods.
The C++ file implements the methods. implementing the Methods.

The bindings are already written for you in **FT_SprintCharacter.cpp lines 97-101**, your main focus is the Unreal side: create the **Input Action**, add it to the **Input Mapping Context** and assign it to the **Sprint Action** in the **BP_ThirdPersonCharacter**.

The input follows the same logic as start and stop jumping, follow that input mapping and context, you can debug the code with:
```
UE_LOG(LogTemp, Warning, TEXT("Sprinting"));
```
For the medium and hard challenges look in the FT_SprintCharacter Constructor (`AFT_SprintCharacter::AFT_SprintCharacter()`), notice the template shows us how to change key values.

## Changes from base project:

### FT_SprintCharacter.h

Lines 47-49 added the input action pointer. This will allow you to assign an **input action in the blueprint editor** for the character.
```
/** Addition 1: Sprint Input Action */
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
UInputAction* SprintAction;
```
you can see this by opening the **BP_ThirdPersonCharacter**, **Content->ThirdPerson->Blueprints**

![The empty input action in the BP_ThirdPersonCharacter.](Hints/AddededInputAction.png)

Lines 51-52 added a variable to remember the normal walking speed, so we always have a value to go back to.
```
/** Addition 2: Hold default value **/
float DefaultSpeed = 0;
```

Lines 66-68 added a start and end sprint function declarations
```
/** Connah Addition 2 methods to ensure we handle sprint enable and disable */
void SprintStart(const FInputActionValue& Value); 
void SprintStop(const FInputActionValue& Value);
```

### FT_SprintCharacter.cpp
Lines 43-44, in the constructor, store the walking speed once it has been set.
```
/** Addition 1: Store base speed **/ 
DefaultSpeed = GetCharacterMovement()->MaxWalkSpeed;
```
Lines 97-101 Added the binding for the input action, this mean when you link the character can begin to sprint. Notice sprint stops on **Completed** and on **Canceled**, Canceled is what you get if the input starts but never fully triggers, for example a quick tap on a **Hold** trigger.
```
/** Addition 2: binding implementation **/ 
EnhancedInputComponent->BindAction(SprintAction, ETriggerEvent::Started, this, &AFT_SprintCharacter::SprintStart);
EnhancedInputComponent->BindAction(SprintAction, ETriggerEvent::Completed, this, &AFT_SprintCharacter::SprintStop);
// Canceled fires instead of Completed if the input never fully triggers, e.g. a quick tap on a Hold trigger
EnhancedInputComponent->BindAction(SprintAction, ETriggerEvent::Canceled, this, &AFT_SprintCharacter::SprintStop);
```
Lines 145-156 Added some code for the start and end sprinting, which will adjust the sprint speed for the player as they start and end sprint. We set the speed from **DefaultSpeed** rather than multiplying the current speed, otherwise a start without a matching stop would keep doubling it.
```
/** Addition: Methods implementation */
void AFT_SprintCharacter::SprintStart(const FInputActionValue& Value)
{
	// we could set speeds, but i like scalers 
	GetCharacterMovement()->MaxWalkSpeed = DefaultSpeed * 2.f;
}

void AFT_SprintCharacter::SprintStop(const FInputActionValue& Value)
{
	// Reset the speeds
	GetCharacterMovement()->MaxWalkSpeed = DefaultSpeed;
}
```

## Challenges
Test your might

## Easy

- Use the sprint you have just made to complete the sprint gate course. Pass through every gate with less than 5 seconds between each one, otherwise the track resets; keep the **Output Log** open to see how you are doing.
- Try the course without sprinting. If you can still finish it, the course is not testing your mechanic, so tune it: add `UPROPERTY(EditAnywhere)` above `gateTimeLimit` in **FT_SprintGateManager.h**, then adjust the time on the gate manager in the level until the course can only be completed with a sprint.
- Connect Xbox controllers for testing; PS controllers can work but are not natively supported.
- Add a stamina limit considering actions like sprinting and jumping, not too advanced can charge/ deplete in the Tick method.

  Hint: follow the task list at the top of **FT_SprintCharacter.cpp**. Add a `float Stamina` and a `bool bIsSprinting` to the header, set the bool in **SprintStart** and **SprintStop**, then declare `virtual void Tick(float DeltaTime) override;` (characters already tick, you just need to override it). In Tick, multiply by **DeltaTime** so the cost is per second, not per frame:

  ```cpp
  if (bIsSprinting)
  {
      Stamina -= SprintCost * DeltaTime;
  }
  ```

  For jumping, bind **JumpAction** to your own function that checks and spends the stamina before calling `Jump()`. A guide can be found [here](Solutions/FT_Sprint_Easy_Stamina.docx).

## Medium
- Introduce acceleration and deceleration, so the character takes time to get up to speed and slow down, when sprint ends.

  Hint: instead of setting **MaxWalkSpeed** straight away in SprintStart and SprintStop, store a `TargetSpeed` and move towards it a little every Tick, with a different rate for speeding up and slowing down:

  ```cpp
  GetCharacterMovement()->MaxWalkSpeed = FMath::FInterpConstantTo(CurrentSpeed, TargetSpeed, DeltaTime, Rate);
  ```

  A guide can be found [here](Solutions/FT_Sprint_Medium_AccelerationAndDeceleration.docx).
- Add delays to stamina recharging, see the gate code.

  Hint: the gate manager counts time in its Tick with `timePassed += DeltaTime;`. Do the same with a `TimeSinceStaminaUsed`, set it back to 0 whenever stamina is spent, and only recharge once it passes your delay. A guide can be found [here](Solutions/FT_Sprint_Medium_StaminaRechargeDelay.docx).
- Try to improve the feedback of the gates as currently they are only debugging.

## Hard 
- Introduce finer elements of gameplay iteration, such as effecting the players turning radius during sprint, see Spyro the Dragon charging.
- Add in an exhaustion mechanic, that adds a longer delay to stamina recharge, or slows the recharge.
- Experiment with different input triggers on your **IA_Sprint**, such as **Hold**, **Tap**, **Pressed** and **Released**, and add `UE_LOG`s to see which events fire: **Started**, **Triggered**, **Completed** and **Canceled**. Why does a quick tap on **Hold** need the Canceled binding? What happens to the sprint with **Pressed**? Then make a toggle sprint, Enhanced Input has no toggle trigger, so this one needs code.

# Reference
Content is made by Connah Kendrick (Connah.Kendrick@mmu.ac.uk) using the Unreal Engine 3rd Person Template for the MMU Game Students.