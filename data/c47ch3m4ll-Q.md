# The Issue
There is no incentive for users to add collateral to their loans, as it holds no significance for their position. When users initiate a loan through the `LendingTerm.borrow` function, there are two scenarios in which their loan may be "called" and sent to the auction house (essentially, liquidated):

1. When the lending term is off-boarded.
2. When the partial repay delay has expired, indicating non-repayment of a portion of the loan on time.
The collateral of the loan and the debt-to-collateral ratio do not influence whether the loan can be liquidated. Consequently, users lack the motivation to invoke the ``LendingTerm.addCollateral`` function, as it solely alters the `loan.collateralAmount`, which does not impact the likelihood of the loan being subject to liquidation.

# Recommendations
Introduce an additional criterion for determining whether a loan can be called or not. Consider not only the last time of partial loan repayment but also factor in whether collateral was added during that time. 

For example, if a user increases the collateral for their loan and improves the debt-to-collateral ratio, the likelihood of liquidation decreases. A loan is subject to liquidation only if the debt-to-collateral ratio meets a specified threshold (a parameter that is defined within the specific LendingTerm).