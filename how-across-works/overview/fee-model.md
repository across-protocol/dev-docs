# Fee Model

### Liquidity Provider Fees

We view using Across as being similar to lending protocols such as AAVE or Compound. When a user bridges a particular token from one chain to another, the fast bridge isn't "moving tokens from one chain to another" but, rather, it is a relayer or the protocol itself providing tokens on the destination chain in return for tokens on the origin chain. We choose to use a similar pricing model for our liquidity provider fees because of this parallel.

#### Utilization based pricing

We base our pricing model on the one described in [AAVE's documentation](https://docs.aave.com/risk/liquidity-risk/borrow-interest-rate#interest-rate-model). Let,

* $$X$$denote the size of a particular transaction someone is seeking to bridge
* $$0 \leq U_t \leq 1$$denote the utilization of the liquidity providers' capital prior to the transaction, i.e. the amount of the liquidity providers' capital that is in use prior to the current transaction
* $$0 \leq \hat{U}_t \leq 1$$denote the utilization of the liquidity providers' capital after to the transaction, i.e. the amount of the liquidity providers' capital that would be in use if the user chose to execute their transaction
* $$\bar{U}$$denote the "kink utilization" where the slope on the interest rate changes
* $$R_0, R_1, R_2$$denote the parameters governing the interest rate model slopes
  * $$R_0$$is the interest rate that would be charged at 0% utilization
  * $$R_0 + R_1$$is the interest rate that would be charged at $$\bar{U}\%$$utilization
  * $$R_0 + R_1 + R_2$$is the interest rate that would be charged at 100% utilization

The (annualized) interest rate model is then defined by

$$
R(U_t) = R_0 + \frac{\min(\bar{U}, U_t)}{\bar{U}} R_1 + \frac{\max(0, U_t - \bar{U})}{1 - \bar{U}} R_2
$$

We calculate the (annualized) interest rate for a particular loan by aggregating the marginal increases of utilization by integrating over this function

$$
R^a_t = \frac{1}{\hat{U}_t - U_t} \int_{U_t}^{\hat{U}_t} R(u) du
$$

The actual fee that is charged will be based on making a loan at this rate for a 1 week time-span. The rate that is charged can be computed from:

$$
R^w_t = (1 + R^a_t)^{\frac{1}{52}} - 1
$$

and the fee would be



$$
\text{Fee} = R^w_t X
$$

This might seem like a lot that you are required to understand, but when you are interacting with the bridge, these details will mostly be hidden from you and you will just see a value for the "liquidity provider fee".

We chose to charge prices this way to ensure that users are paying a "fair" price for the amount of utilization that their bridge transaction incurs.

### Relayer Fees

Relayer fees play a similar role in the Across ecosystem as gas fees play in the Ethereum ecosystem. Relayer fees are a fee that the user sets to incentivize relayers to relay your bridge transaction.

Relaying a transaction has three costs for relayers:

1. Gas fees: The relayer pays gas to perform the relay and to claim their repayment.
2. Capital opportunity costs: The fact that the relayer is using their capital to perform a relay means that they are not using it for other yield opportunities.
3. Capital at risk: A relayer takes on certain risks by relaying funds. If they make a mistake when relaying that could jeopardize their repayment of the capital they invested.

These fees are automatically set by the front-end but could also be set manually by interacting with the contract directly. If you set them manually, make sure to set them high enough so that a relayer would be willing to relay your transactions.



