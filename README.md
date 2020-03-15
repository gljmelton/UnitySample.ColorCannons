# Color Cannons+ Code Samples

For a look at what Color Cannons+ is I reccomend you check out [Color Cannons+ on itch.io](https://symptomatic.itch.io/color-cannons) to at least get familiar with what the game looks like. Here I'm going to go over some bits of code from the game.

## Turning

Movement in Color Cannons+ is large physics driven but mechanics like turning are more deterministic. Here's a look at the system for turning the players:

```C#
//Rotates the players cannon. A modifier button determines of we should turn 90 or 45 degrees
    private void Turn(bool turnRight, bool bigTurn) {
        turning = true;
        turnLerpTimer = 0f;
        turnStartAngle = cannon.transform.rotation.eulerAngles.z;

        //If turning right subtract degrees from the turn target, otherwise add.
        if (turnRight) {
            if (bigTurn) {
                turnTargetAngle = -90f;
            }else {
                turnTargetAngle = -45f;
            }

        }else {
            if (bigTurn) {
                turnTargetAngle = 90f;
            } else {
                turnTargetAngle = 45f;
            }
        }

        //Play a sound for turning at a modified pitch.
        audioSource.pitch = Random.Range(0.8f, 1.2f);
        audioSource.PlayOneShot(turnSound);
        
    }
```
Most mechanics in CC+ rely on cooldown timers. For turning, moving, firing, the first thing I do is start the cooldown timer that gets checked and updated in the update loop. There's also some vestigial code in here that's looking for "Big Turn". There used to be a mechanic that allowed players to turn either 90 or 45 degrees in either direction, but that ended up being more complicated than necessary, so eventually the turn cooldown was reduced and players were only allowed to turn 45 degrees at a time.

Then we set the angle we want to turn to and the angle we're at now. Here's how the actual math that's used for turning looks:
```C#
if (turning) {
    turnLerpTimer += Time.deltaTime;

    //Figure out how far through the turn we are. This should return a number <= the time it takes to turn.
    float turnAmount = (turnLerpTimer * turnSpeed);

    //Decide if we need to keep turning after this
    if (turnAmount >= turnLerpTime) {
        turning = false;
    }

    //Lerp the angle we're at now toward where we want to be, evaluated on a curve to smooth out the motion
    float newAngle = Mathf.LerpAngle(turnStartAngle, turnTargetAngle+turnStartAngle, turnCurve.Evaluate(turnAmount / turnLerpTime));

    //Set our rotation
    cannon.transform.rotation = Quaternion.Euler(cannon.transform.rotation.eulerAngles.x, cannon.transform.rotation.eulerAngles.y, newAngle);
}
```
This code lives in the update loop, and eases the players cannon toward the target angle.
