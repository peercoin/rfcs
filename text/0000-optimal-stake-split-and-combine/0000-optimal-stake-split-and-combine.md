# Optimal Stake Split and Combine

- Status: proposed
- Type: enhancement
- Start Date: 20-08-2023
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Supersedes: [RFC-0016](../0016-split-and-combine/0016-split-and-combine.md)
- Author: MatthewLM

## Summary

To improve chain security and provide fair rewards for minters, coinstake
transactions shall split and combine outputs to target an output size that
provides an average reward close to the maximum possible reward for the current
supply and difficulty.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- "BlockIntervalSeconds" is the number of seconds targeted between PoS blocks,
    currently 600.
- "Diff" is the PoS difficulty.
- "Static" is the static reward in PPC.
- "MaxDayWeight" is the maximum day weight of an output for adjusting the PoS
    kernel hash target. Currently this is 60 days.
- "Supply" is the current supply of PPC as determined by the
    client.
- "OptimumSize" is the optimum output size to maximise reward in PPC.
- "SecurityLevel" is the fraction of supply that would need to be minted
    continuously at the lowest output value to meet the current PoS difficulty.

## Motivation

Currently stake amounts are split whenever a mint occurs before the maximum
probability is reached (90 days). The justification is that these outputs are
large enough to mint quickly and therefore can be split without leading to a
high chance of taking too long to mint.

However, this behaviour relies on a stochastic process because mint timing is
random over a probability distribution. This may lead to smaller outputs
splitting and larger outputs not splitting by chance. It also does not determine
what the true optimal output size is for maximising minting rewards.

Coinstakes also combine outputs of the same address that have reached maximum
probability where the stake amount is less than one-third of the PoW reward. The
PoW reward is an irrelevant measure. Taken together, the split and combine logic
does not target output sizes that are truly optimal.

The solution in this RFC targets output sizes near to the economic optimum. The
optimum is the output size that produces the highest expected return when
continuously minting. The changes proposed in this RFC splits large stakes into
more than two outputs which avoids requiring multiple splits.

Combination within coinstakes only includes inputs with values that are a small
fraction of the optimum to restrict the loss of chain security.

## Background

### Calculating Rewards

It is possible to determine the expected reward for a given output size,
difficulty and static reward. The minting probabilities can be calculated for an
output size across a number of days into the future. The expected reward can be
calculated by the probability-weighted average of possible rewards over these
days.

A [Jupyter
notebook](https://github.com/MatthewLM/OptimumPeercoinUTXO/blob/master/OptimumUTXO.ipynb)
has been produced that can calculate expected rewards and determine the optimal
output size to maximise reward using a peak-finding algorithm. The optimal size
assumes continuous minting without orphan blocks (rare).

A simulation within the notebook has shown that the predicted rewards are made
with good accuracy with increasing variance for low output values and an
under-estimation for larger values. The under-estimation is likely due to
compounding effects of large outputs that mint often. The error is too small to
be concerned about (under 5bps). The optimum output value is in close alignment.

### Normalising for Supply

A function to estimate rewards can be determined from the current difficulty and
static reward value:

$$ OptimumSize = f_{1}(Diff, Static) $$

It is possible to simplify the function to a single input variable by
normalising the optimum size as a fraction of supply and the difficulty as a
fraction of maximum difficulty. The latter is referred to as the "security
level" or "security parameter". When doing this, the static reward is no longer
required.

The maximum difficulty can be determined by assuming that the entire supply is
minting with the smallest possible output. The calculation may be simplified by
assuming that outputs are minting at maximum probability at all times. Whilst
this is not true, the minimal 0.01 PPC outputs are at maximum probability most
of the time so it provides a close approximation.

The security level can be closely approximated as:

$$ SecurityLevel = \frac{2^{32} \cdot Diff}{MaxDayWeight \cdot Supply \cdot BlockIntervalSeconds} $$

The maximum difficulty is relative to supply according to the $MaxDayWeight$
, $BlockIntervalSeconds$ and a power of 2 which is used to calculate mint
probabilities in the client.

Using the approximated security level, the optimum size can be determined by a
new function $f_{2}$ that determines the optimum fraction of supply:

$$ OptimumSize = f_{2}(SecurityLevel)\cdot Supply $$

The $f_{2}$ function can use the $f_{1}$ function after determining the
difficulty and static reward from the supply, but the result is invariant to the
supply allowing any value to be used. Therefore only a single input variable is
required to this new function.

### Regression Model

An OLS regression model found in the notebook can closely predict the optimum
size as a fraction of supply from the security level.

Root terms in the form of $x^{1/i}$ were added with increasing values of $i$
until the AIC number stopped falling. The relationship between security level
and the optimum fraction tends towards being linear and roots predict this
relationship well. All terms in the model are highly statistically significant.

This regression model is used to calculate the optimum output size in this RFC.

Alternative coefficients are provided for testnet. Mainnet coefficients can be
used, though this will lead to inaccurate estimated optimums which can be up to
60% higher as the security level approaches 1.

### Optimal Splitting

Output values can be split in a coinstake to approach the optimum value. An
output can be split into a number of new outputs that minimises the logarithmic
distance between the resulting output values and the optimum.

The point at which the logarithmic distance is equal between the number of split
outputs $y$ and $y-1$ is given by:

$$ | \log(\frac{x}{y}) | = | \log(\frac{x}{y-1}) | $$

The resulting $y$ is given by:

$$ y = \frac{1}{2}\left( \sqrt{4x^{2}+1}+1 \right) $$

Since $y$ and $y-1$ provide an equal distance from the optimum, the optimal
split can be determined by whatever integer lies between them. Therefore
$floor(y)$ provides the optimal number of split outputs.

## Detailed design

The client code shall have the files `src/wallet/optimum_mint.h` and
`src/wallet/optimum_mint.cpp` containing new functions and constants.

A constant named `RECOMBINE_DIVISOR` shall be set to 10. This shall divide the
optimum value to determine the threshold under which small outputs can be
combined.

A constant named `MIN_COINSTAKE_AMOUNT` shall be set to `10*COIN` to be the
floor value for coinstake output amounts.

A constant named `MAX_COINSTAKE_INPUTS` shall be set to 4 to be the maximum
number of inputs that can be included in a coinstake transaction. This ensures
at least 11 outputs can fit in the coinstake without exceeding the 1KB fee-free
limit.

### SecurityToOptimalFraction

A `double SecurityToOptimalFraction(double security);` function shall take the
security level and return the fraction of supply for the optimal output size.
The calculation must be determined from the linear regression model. The
coefficients for the current network parameters are as follows:

```
          const: -0.01205449390140
       security: 0.00021672965052
 security^(1/2): 0.00843001158271
 security^(1/3): -0.31668853152981
 security^(1/4): 3.15194385589535
 security^(1/5): -12.93131277924088
 security^(1/6): 25.02314857164244
 security^(1/7): -22.70985422100839
 security^(1/8): 7.78628320089075
```

If the network parameters that determine optimal rewards are changed, the
regression must be performed again and new coefficients and/or terms
must be included to be used only when the new rules are activated.

Alternative coefficients may be used for the current testnet parameters:

```
          const: -0.00277474262686
       security: 0.00015824596035
 security^(1/2): 0.00012080445529
 security^(1/3): -0.04679164943874
 security^(1/4): 0.56524766014644
 security^(1/5): -2.50113964308348
 security^(1/6): 5.03930675692864
 security^(1/7): -4.68984257228335
 security^(1/8): 1.63578550998557
```

### CalcOptimalOutputSize

A `CAmount CalcOptimalOutputSize(double difficulty, CAmount supply)` function shall
calculate the optimal output size from the difficulty and supply using the
`SecurityToOptimalFraction` function. The security level shall be calculated
from the difficulty and supply using:

```
double securityLevel = (2 << 31)*difficulty / maxDayWeight / supply / nStakeTargetSpacing
```

The `maxDayWeight` shall be determined by the difference between `nStakeMinAge`
and `nStakeMaxAge` in days.

The result shall be multiplied by the supply and clipped to at least
`MIN_COINSTAKE_AMOUNT`.

### CalcOptimalSplit

A `int CalcOptimalSplit(CAmount target, CAmount current)` function shall take
the target output size and the current total value to determine how many outputs
should be created to evenly split the amount. This calculation shall be done
according to the following expression:
`floor((sqrt(4*((current/target)^2)+1) + 1) / 2)`.

### CreateCoinStake

The `CreateCoinStake` function in `src/wallet/wallet.cpp` shall be modified.

`nStakeSplitAge` shall be removed and replaced with `nOptimalOutputSize` set by
a call to `CalcOptimalOutputSize`. `nCombineThreshold` shall be set to
`nOptimalOutputSize / RECOMBINE_DIVISOR`.

When adding inputs, they shall no longer be skipped if they have not reached
maximum probability. Inputs shall continue to be added when the total input
value goes above `nCombineThreshold`. `nCombineThreshold` shall only apply to
individual inputs. The number of inputs added will be limited to
`MAX_COINSTAKE_INPUTS`.

`CalcOptimalSplit` shall be used to determine how many outputs should be
included in the coinstake. The number of outputs shall be limited to how many
can be added to the coinstake without exceeding 1KB. This is a simple
calculation given that P2PKH and P2WPKH outputs have a known size.

If the `splitcoins` option is false, coins shall not be split.

## Drawbacks

The optimum size assumes continuous minting and no stale blocks. Stale blocks
are currently rare and do not pose a major concern. Occasional minters may be
better served with higher output sizes that are more likely to mint before the
coinage reward is capped at 365 days. However, continuous minting is important
for security and therefore users should be encouraged to maximise rewards
through leaving their client to mint continuously.

The inclusion of a formula from the regression model requires knowledge of this
model to understand. It is not apparent from the code alone how this will
produce correct answers.

The regression model must be updated when the network parameters change.

## Alternatives

`RECOMBINE_DIVISOR` causes small outputs that aren't under the smaller threshold
but are under the optimum to not be combined. This can lead to a reduction in
output rewards. The optimal recombine threshold could be set higher. This is not
done to retain the chain security offered by smaller outputs. This may be
reconsidered after chain parameters have been altered to favour small outputs
and/or reduce the amount of security lost at the economic optimum.

The optimal output size could be determined by taking the probability-weighted
average of possible returns for a given output size and then using a
peak-finding algorithm to find the optimal. This is already provided in the
Jupyter notebook. This would provide a more analytical calculation that can find
the optimal output sizes for different network parameters. However it does add
time and code complexity with loops. A closed-form analytical solution hasn't
been found that can take into account all variables. The regression model works
well for given network parameters and provides a straight-forward expression.

The `MAX_COINSTAKE_INPUTS` could be set to a higher amount to optimise small
outputs more quickly but the current value allows for at least 11 outputs
whilst avoiding possible controversy of coinstake fees. There could be an
option to determine the behaviour, allowing for coinstake fees. Currently up-to
100 inputs can be combined and this RFC changes that to only 4.

