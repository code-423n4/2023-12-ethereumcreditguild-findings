1) AuctionHouse::startAuction starts the auction of the collateral of a loan, to be exchanged for CREDIT, in 
  order to pay the debt of the loan.

  https://github.com/volt-protocol/ethereum-credit- 
  guild/blob/4d33abf95fee69391af0652e3cbe5e0cffa25f9f/src/loan/AuctionHouse.sol#L75-L113

  When startAuction is called, the callDebt parameter specifies the amount of credit that should be recovered 
  by auctioning the collateral. However, there is no validation that this callDebt actually matches real debt owed 
  by the borrower in the loan. 

  Validate callDebt on startAuction against actual loan debt