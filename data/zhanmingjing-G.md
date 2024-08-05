https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217

amountMinted is calculated but never used.

       function mintWithBudget(
           bytes32[] calldata proof
       )
        public
        payable
        whenNotPaused
        nonReentrant
        onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
      {
        uint256 mintPrice = calculateMintPrice();
        uint256 amountMinted = 0;
        uint256 budgetLeft = msg.value;

        while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
          _mintInternal(msg.sender, mintPrice);
          amountMinted++;
          budgetLeft -= mintPrice;
          mintPrice = calculateMintPrice();
        }
        if (budgetLeft > 0) {
          (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
          require(refundSuccess, 'Refund failed.');
        }
      }