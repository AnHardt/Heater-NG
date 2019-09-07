# Heater-NG
Next Generation Heat Controller for Marlin.

Let me plot the idea for the alternate heater algorithm.
Let's begin with an experiment.
Ideally take a self limiting bed heater.
Begin with the heaters temperature at the environments temperature.
Set `#define PID_OPENLOOP 1`, to have direct access to the heater power.
Set MAXTEMP very high and switch off all protections. (So be careful and stay close to the machine.)
Let's protocol the temperatures and heater power over time. (Repetier Host Temperature Plot).

1. Switch the heater full on.
2. When the temperature does not rise anymore (or you are scared to ruin the heater) switch off the heater.
3. When the temperature does not fall anymore set heater power to 10%.
4. When the temperature does not rise anymore set heater power to 20%.
5. When the temperature does not rise anymore set heater power to 30%.

and so on to 100%.

12. Now again at 100% and max temperature, switch on a fan blowing over the surface of the heater. Observe the temperature falling to a new stable value below the former max.
13. Switch off the heater and restore safe settings.

At first let's have a look on the plot of phase 2. After a few seconds we get a smooth curve. At the beginning falling fast, then more and more slowly falling arriving room temperature. What we see is the temperature loss over time. That is about proportional the energy loss over time. For every temperature above room temperature and below the maximum temperature we can find a value in that graph - how many °C is the temperature dropping in the next second. The curve can be calculated with [Newton's law of cooling](https://en.wikipedia.org/wiki/Newton%27s_law_of_cooling).
![image](https://user-images.githubusercontent.com/211931/64333564-16dcd400-cfd7-11e9-99a1-a69dd8d57230.png)
The temperatur `T` at the time `t` T(t) is equal to the room temperature `Tenv` plus the difference of the start (max) temperature `T(0)` and the room temperature `Tenv` times e (a mathematical constant)
to the power of the product of minus an other konstant `r` and the time `t` since the cooling started.
In `r` we have the properties of the heater system, like surface of the heater and the surrounding media and how fast that moves.

In phase 1 we see power the heater with a constant wattage. If we would not have energy losses, we'd see a straight line from the start to infinity. For every second the heater is on we'd see a constant rise in temperature.
There are losses. The losses are the same as in part 2 of the graph. So after a second of heating we get a constant rise of the temperature but also a temperature dependent decrease what we can get from phase 2. 
At some temperature (the self-limit-temperature) the losses are exactly that high as the heater power.

In the phases 3. to 11. we see about the same as in phase 1. The temperature rises slowly to that temperature where the losses are equal to the energy input. We also observe the distance between the temperatures gets smaller, in the equal 10% power steps.
What is not that obvious is that the power we put in the system is represented exactly the I therm in a PID. When the PID has swung in, is and target temperature is the same P is 0 and if that is for some time D is 0. Only I is remaining.

In phase 13. we see what happens if `r` begins to differ. A PID would after some time adjust its I therm. to get back to the former temperature.

What can we do with that insights.
From phase 2. we can determine `r`. From phase 1. and `r` we can determine the wattage. With 'r` and the wattage we can correlate wattage and target temperature.

For now let's assume we don't have disturbances.
When we want to heat from T1 to T2 we can look up the duration for heating from phase 1 and give full power for that time. After that time we can switch to a heater power looked up from phase 3 to 11. When cooling from T2 to T3 we can look up the duration for switching the power off in phase 2. There after we can again heat with a power looked up in 3 - 11.
In theory we can give or take exactly that amount of energy to reach the new temperature, without overshot.
 
If we have disturbances we could either put a PD controller on top - at least when holding temperatures, or a PID and derive `r` from the I-term, or we could recalculate the actual `r` from the temperatures and heater powers of the last few seconds. But because of the delay between applying the new power and the reaction of temperature this is not trivial. 

For fan off and fan on we will get different but similar values for `r`. When we get a problem with the heater, like separated, shorted, broken sensors or heater cartridges, `r` will differ a lot, so errors can be detected. 
