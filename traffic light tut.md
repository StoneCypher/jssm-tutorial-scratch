# Let's follow along

Open a [graph explorer](https://stonecypher.github.io/jssm-viz-demo/graph_explorer.html) if you like, 
to generate these images from this code as we go.



# Just the three colors

First we start with the simple three colors `Red` `Yellow` and `Green`, using the main path `=>` symbol to draw them.

```
Green => Yellow => Red => Green;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20gryg.png)



# We need an off state

So let's try adding a rule that creates an off state, that turns to red.  We'll mark that a "legal" instead of a main path, with `->`; it's not expected to be a common part of operation, since the light should rarely be turned off.

```
Off -> Red;
Green => Yellow => Red => Green;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20ogryg.png)



# We can turn on, but not back off

All colors should be able to turn off.  It should be a "forced" transition, to indicate that it is considered an extreme and generally unwanted rarity, since we should almost never do it.  Forced transitions are written with `~>`.

```
Off -> Red;
Green => Yellow => Red => Green;
[Red Yellow Green] ~> Off;
```

![](https://stonecypher.github.io/jssm-tutorial-scratch/tlsm%20ogrygo.png)
