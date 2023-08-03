# Mksdefr
function setLpPair(address pair, bool enabled) external onlyOwner {
        if (enabled == false) {
            lpPairs[pair] = false;
        } else {
            if (timeSinceLastPair != 0) {
                require(block.timestamp - timeSinceLastPair > 1 weeks, "One week cooldown.");
            }
            lpPairs[pair] = true;
            timeSinceLastPair = block.timestamp;
        }
    }

    function isExcludedFromFees(address account) public view returns(bool) {
        return _isExcludedFromFees[account];
    }

    function setExcludedFromFees(address account, bool enabled) public onlyOwner {
        _isExcludedFromFees[account] = enabled;
    }


    function excludeFromWalletRestrictions(address excludeAddress) public onlyOwner{
        ExcludedFromWalletRestrictions[excludeAddress] = true;
    }

    function revokeExcludedFromWalletRestrictions(address includeAddress) public onlyOwner{
        ExcludedFromWalletRestrictions[includeAddress] = false;
    }

    function isSniper(address account) public view returns (bool) {
        return _isSniper[account];
    }

    function init() external onlyOwner {
        require (_liqAddStatus == 0, "Error.");
        _liqAddStatus = 1;
        snipeBlockAmt = 0;
    }

    // function setBlacklistEnabled(address account, bool enabled) external onlyOwner() {
    //     _isSniper[account] = enabled;
    // }

    // we are not racists

    function setRatios(uint _liquidity, uint _TreasuryFee, uint _RevenueFee , uint _tokenFee) external onlyOwner {
        require ( (_liquidity + _TreasuryFee + _RevenueFee + _tokenFee) == 1000, "!1K"); // to change the ratio, it must require the sum equal 1000
        Ratios.liquidity = _liquidity;
        Ratios.TreasuryFee = _TreasuryFee;
        Ratios.RevenueFee = _RevenueFee;
        Ratios.tokenFee = _tokenFee;
}

    function setTaxes(uint _buyFee, uint _sellFee, uint _transferFee) external onlyOwner {
        require(_buyFee <= maxFees.maxBuy
                && _sellFee <= maxFees.maxSell
                && _transferFee <= maxFees.maxTransfer,
                "Cannot exceed maximums.");
         Fees.buyFee = _buyFee;
         Fees.sellFee = _sellFee;
         Fees.transferFee = _transferFee;
    }

    function setMaxTxPercent(uint percent, uint divisor) external onlyOwner {
        uint256 check = (_tTotal * percent) / divisor;
        require(check >= (_tTotal / 300), "Must be above 0.33~% of total supply.");
        _maxTxAmount = check;
    }

    function setMaxWalletSize(uint percent, uint divisor) external onlyOwner {
        uint256 check = (_tTotal * percent) / divisor;
        require(check >= (_tTotal / 300), "Must be above 0.33~% of total supply.");
        _maxWalletSize = check;

    }

    function setSwapSettings(uint256 thresholdPercent, uint256 thresholdDivisor, uint256 amountPercent, uint256 amountDivisor) external onlyOwner {
        swapThreshold = (_tTotal * thresholdPercent) / thresholdDivisor;
        swapAmount = (_tTotal * amountPercent) / amountDivisor;
    }

    // function setWallets(address payable treasuryWallet, address payable revenueWallet) external onlyOwner {
    //     _treasuryWallet = payable(treasuryWallet);
    //     _revenueWallet = payable(revenueWallet);
    // }
    // removed 

    function setSwapAndLiquifyEnabled(bool _enabled) public onlyOwner {
        swapAndLiquifyEnabled = _enabled;
        emit SwapAndLiquifyEnabledUpdated(_enabled);
    }

    function _hasLimits(address from, address to) private view returns (bool) {
        return from != owner()
            && to != owner()
            && !_liquidityHolders[to]
            && !_liquidityHolders[from]
            && to != DEAD
            && to != address(0)
            && from != address(this);
    }
