# Let's model an intersection

It's both easier and harder than it sounds.  By the end we get some purty pictures.

Open a [graph explorer](https://stonecypher.github.io/jssm-viz-demo/graph_explorer.html) if you like,
to generate these images from this code as we go.  Alternately, it's fine to just read them in place;
these are fairly small steps.



<br/><br/>

# Just the three colors

First we start with the simple three colors `Red` `Yellow` and `Green`, using the main path `=>` symbol to draw them.

```
Green => Yellow => Red => Green;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20gryg.png)



<br/><br/>

# We need an off state

So let's try adding a rule that creates an off state, that turns to red.  We'll mark that a "legal" instead of a main path, with `->`; it's not expected to be a common part of operation, since the light should rarely be turned off.

```
Off -> Red;
Green => Yellow => Red => Green;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20ogryg.png)



<br/><br/>

# We can turn on, but not back off

All colors should be able to turn off.  It should be a "forced" transition, to indicate that it is considered an extreme and generally unwanted rarity, since we should almost never do it.  Forced transitions are written with `~>`.

```
Off -> Red;
Green => Yellow => Red => Green;
[Red Yellow Green] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20ogrygo.png)



<br/><br/>

# Intersections are four lights, not one

![](https://stonecypher.github.io/jssm-tutorial-scratch/there%20are%20four%20lights.jpg)

We could try to make an individual light then build intersections out of that, but it's
simpler to just model the whole intersection directly.

A basic four-way intersection could be written, for example, by the color of the north
light and the color of the east light.  Say, `GreenN_RedE` for an intersection where the
north/south lane was `Green`, and the east/west lane was `Red`.  Then you'd end up with
a machine like this:

```
Off -> RedN_RedE;

RedN_RedE => RedN_GreenE => RedN_YellowE => RedN_RedE_2 => GreenN_RedE => YellowN_RedE => RedN_RedE;

[RedN_GreenE RedN_YellowE RedN_RedE GreenN_RedE YellowN_RedE RedN_RedE_2] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20simple%204way%20with%20off.png)

## RedN_RedE_2

Notice that we've had to distinguish two different `Red`/`Red` pairs (`RedN_RedE` and `RedN_RedE_2`), so
that the machine would know which lane was going to go `Green` afterwards.

If we didn't do that, we'd end up with this bug instead:

```
Off -> RedN_RedE;

RedN_RedE => RedN_GreenE => RedN_YellowE => RedN_RedE => GreenN_RedE => YellowN_RedE => RedN_RedE;

[RedN_GreenE RedN_YellowE RedN_RedE GreenN_RedE YellowN_RedE] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20simple%204way%20with%20off%20no%202%20bug.png)



<br/><br/>

# Four-way flashing red

Let's add a state for four-way flashing red.  Since we want spaces in its name we'll use the quoted
syntax to write it - `"Four-way Flash Red"`.  All quotes do is let you write a name less conveniently,
but with more label expression (including unicode callouts.)

We can normal transition to our flashing red state from either of the `Red`/`Red`s, such as for late
night simple traffic.

```
[RedN_RedE RedN_RedE_2] -> "Four-way Flash Red";
```

We can transition the machine out of flash into either red/red pair, though the main one is expected,
as indicated by main pathing.

```
"Four-way Flash Red" => RedN_RedE;
"Four-way Flash Red" -> RedN_RedE_2;
```

We can also forced-transition it from any of the other four color pairings, in case of an emergency,
such as a traffic accident, natural disaster, or Gojira.

```
[RedN_GreenE RedN_YellowE GreenN_RedE YellowN_RedE] ~> "Four-way Flash Red";
```

## Result

That leaves us with the following code.  I added some spaces to make it easier to see how the lists
line up, but they don't actually affect the code, and you can take them out if you don't like them.

```
Off -> RedN_RedE;
RedN_RedE => RedN_GreenE => RedN_YellowE => RedN_RedE_2 => GreenN_RedE => YellowN_RedE => RedN_RedE;

"Four-way Flash Red" => RedN_RedE;
"Four-way Flash Red" -> RedN_RedE_2;

[                         RedN_RedE                          RedN_RedE_2] -> "Four-way Flash Red";
[RedN_GreenE RedN_YellowE           GreenN_RedE YellowN_RedE            ] ~> "Four-way Flash Red";
[RedN_GreenE RedN_YellowE RedN_RedE GreenN_RedE YellowN_RedE RedN_RedE_2] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20simple%204way%20flashred.png)



<br/><br/>

# 2-way flash red/yellow

Of course, many four-way lights want to flash yellow in one direction, but red in the other.  Since
this state isn't useful for emergencies like `Four-way Flash Red` is, we won't support forced
transitions for it; major state changes should only be made from `Red`/`Red` states.

So we'll just define two rules, each from a `Red`/`Red` pair to one of the two flashing pairs and
then back to non-`_2`.

```
[RedN_RedE RedN_RedE_2] -> "Flash NS-Red EW-Yellow" -> [RedN_RedE RedN_RedE_2];
[RedN_RedE RedN_RedE_2] -> "Flash NS-Yellow EW-Red" -> [RedN_RedE RedN_RedE_2];
```

## Result

That leaves us with

```
Off -> RedN_RedE;
RedN_RedE => RedN_GreenE => RedN_YellowE => RedN_RedE_2 => GreenN_RedE => YellowN_RedE => RedN_RedE;

"Four-way Flash Red" => RedN_RedE;
"Four-way Flash Red" -> RedN_RedE_2;

[RedN_RedE RedN_RedE_2] -> "Flash NS-Red EW-Yellow" -> [RedN_RedE RedN_RedE_2];
[RedN_RedE RedN_RedE_2] -> "Flash NS-Yellow EW-Red" -> [RedN_RedE RedN_RedE_2];

[                         RedN_RedE                          RedN_RedE_2] -> "Four-way Flash Red";
[RedN_GreenE RedN_YellowE           GreenN_RedE YellowN_RedE            ] ~> "Four-way Flash Red";
[RedN_GreenE RedN_YellowE RedN_RedE GreenN_RedE YellowN_RedE RedN_RedE_2] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20extraflash.png)



<br/><br/>

# Exercise for the reader

Without worrying about states like `Off` and the flashing states, maybe try building two four-way
intersections, both around green arrows:

1. A four-way intersection where `Green`-`Yellow`-gone arrows are added for both directions for left
turns simultaneously, and

2. A four-way intersection where they're only added North-South, with a `Green`-`Yellow`-gone arrow
for north while north is also `Green`, and where south stays `Red` until the arrow is gone.