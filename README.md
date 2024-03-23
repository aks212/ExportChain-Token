 ExportChain-Token
a project of non-oil business tokenization using blockchain tech using ethereum network


//SPDX-License-Identifier:MIT
pragma solidity ^0.8.0;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@chainlink/contracts/src/v0.8/ChainlinkAutomation.sol";

contract ProductionChainToken is ERC20, Ownable, ChainlinkAutomation {
    // Mapping to track asset ownership
    mapping(address => uint256) private _assetBalances;
    // Mapping to track staked tokens and rewards
    mapping(address => uint256) private _stakedTokens;
    mapping(address => uint256) private _rewards;

    // Constructor
    constructor() ERC20("ProductionChain Token", "PCT") {
        _mint(msg.sender, 1000000 * (10 ** uint256(decimals())));
    }

    // Function to tokenize an asset
    function tokenizeAsset(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
        _assetBalances[to] += amount;
    }

    // Function to transfer tokens representing assets
    function transferAsset(address from, address to, uint256 amount) public onlyOwner {
        require(_assetBalances[from] >= amount, "Not enough assets to transfer");
        _transfer(from, to, amount);
        _assetBalances[from] -= amount;
        _assetBalances[to] += amount;
    }

    // Function to stake tokens and earn rewards
    function stake(uint256 amount) public {
        _transfer(msg.sender, address(this), amount);
        _stakedTokens[msg.sender] += amount;
        _rewards[msg.sender] = amount * 10 ** uint256(decimals()); // Distribute rewards in proportion to staked tokens
    }

    // Function to withdraw staked tokens and rewards
    function withdraw() public {
        uint256 amount = balanceOf(msg.sender);
        _transfer(address(this), msg.sender, amount);
        _stakedTokens[msg.sender] -= amount;
        _rewards[msg.sender] -= amount * 10 ** uint256(decimals());
    }

    // Function to distribute rewards based on staked tokens
    function distributeRewards() internal {
        uint256 totalStakedTokens = totalSupply(); // Total supply of staked tokens
        uint256 totalRewards = balanceOf(address(this)); // Total rewards available for distribution
        
        for (uint256 i = 0; i < accounts.length; i++) {
            address account = accounts[i];
            uint256 reward = totalRewards * _stakedTokens[account] / totalStakedTokens;
            _transfer(address(this), account, reward);
        }
    }

    function automateRewardDistribution() public onlyOwner {
        ProductionChainToken pct = ProductionChainToken(address(this));
        pct.distributeRewards(); // Call the function to distribute rewards
        address[] accounts = _stakedTokens.getKeys();
        uint256[] stakedValues = _stakedTokens.getValues();
        uint256[] rewardsValues = _rewards.getValues();
        distributeRewards(); // Call the function to distribute rewards
    }
}
