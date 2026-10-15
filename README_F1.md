# Silverstone Pit-Stop Strategy Simulation

A Monte Carlo simulation of Formula 1 pit-stop strategies at Silverstone, built from public lap-timing data. It asks: for a dry 52-lap race, how many stops are fastest, and how sensitive is that answer to the assumptions?

## Question

Given how quickly tyres wear and how much time a pit stop costs, which strategy is fastest over a 52-lap Silverstone race, and how confident can we be?

## Data

- Lap times, tyre stints, pit stops and race-control messages from the OpenF1 API (https://openf1.org), session key 9558 (British GP 2024) plus the 2023 British GP.
- Bahrain 2024 and Barcelona 2024 were used only to estimate the fuel-burn effect.
- FastF1 was tried first but its data source returned no data for any race, so the project uses OpenF1.

## Method

1. **Clean the laps.** Removed lap 1, pit in-laps and out-laps, safety-car laps (none occurred in the races used), and laps more than 4% slower than the driver's stint median. 2024 was a mixed wet/dry race: wet laps (intermediates, laps 20-39) were removed, and the transition laps around them excluded, leaving the dry windows only.
2. **Fuel correction.** Each dry race gave a fuel-burn slope of about -0.055 to -0.067 s per lap. The average (-0.0624 s/lap) was added back to every lap time so that tyre age and fuel were not tangled together.
3. **Tyre model.** Linear regression of fuel-corrected lap time on driver, compound and tyre age (plus a post-rain indicator for 2024), with standard errors clustered by driver stint because laps within a stint are not independent.
4. **Simulation.** For 2,730 strategies (1 to 3 stops, stint lengths 8 to 31 laps, at least two compounds), ran 4,000 simulated races. Each run draws the fitted pace offsets and wear rate from their estimated uncertainty and draws each pit stop's time loss from a normal distribution (22 s, standard deviation 2.5 s).
5. **Sensitivity.** Repeated the comparison of the best one-stop and best two-stop strategies across pit-stop losses from 10 to 28 s.

## Results

**Fitted parameters (Silverstone 2023 + dry laps of 2024, 1,304 laps)**

| Parameter | Estimate | Standard error |
|---|---|---|
| Tyre wear (all compounds) | 0.043 s per lap of tyre age | 0.007 |
| Medium pace vs Hard | -0.11 s | 0.07 |
| Soft pace vs Hard | +0.17 s | 0.11 |

The compound pace differences are not statistically distinguishable from zero.

**Strategy comparison (dry 52-lap race, 22 s pit-stop loss)**

| Stops | Best strategy | Mean time vs best one-stop |
|---|---|---|
| 1 | Hard 24 laps then Medium 28 | baseline |
| 2 | Medium 20, Medium 17, Hard 15 | +11.3 s slower |
| 3 | Medium 14, 14, 14, Hard 10 | +27.9 s slower |

- A one-stop strategy was fastest in 99.9% of simulated races.
- A second stop only pays off if the total time lost per stop falls below about 11 s. Real Silverstone stops in the data cost about 21 to 23 s, so two stops are not competitive in a dry race.
- The simulated best first stint is about 23 to 29 laps. In the dry 2023 race, 13 of 18 classified drivers ran one stop, with first stints mostly between 26 and 33 laps, which is the same pattern. Most of the 4 two-stoppers and the 1 three-stopper were probably reacting to safety cars or traffic, which the simulation does not model.

## Limitations

- Compounds cannot be separated with this data. Wear and pace differences between Soft, Medium and Hard were within noise at Silverstone, and the pooled-circuit estimates were driven by which compounds were run where. The simulation therefore uses one common wear rate, and the order of compounds in the "best" strategies is not a finding.
- The pit-stop loss (22 s) is an assumption, supported by only a few clean dry stops in 2024 (median 23 s) and the lower quartile of 2023 stops (21 s). The sensitivity table shows how much the answer depends on it.
- The model has no traffic, track position, undercut effects, safety cars, tyre "cliffs" or weather. Tyre wear is assumed linear.
- Out-of-sample tests were weak: including tyre age reduced held-out lap-time error by only about 2% in the 2024 dry laps.
- The 2024 race was rain-affected, so it was used for calibration only. The simulation answers a dry-race question and does not evaluate what teams did in the rain.
- Fuel-burn slope is an average from other circuits and is treated as fixed.

## Possible extensions

- Add traffic and safety-car scenarios.
- Fit a non-linear tyre curve with a late-stint "cliff".
- Estimate pit-stop loss from more stops and across more circuits.
- Add wet-weather decision modelling for races like Silverstone 2024.

## Tools

Python, pandas, NumPy, statsmodels, matplotlib, OpenF1 API.
