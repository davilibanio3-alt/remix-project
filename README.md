<p align="center">[20/3 20:28] Davi Calixto: contrato 0x51de5828d345bdcff865f0d6b793a6fbd1214444
[19/3 00:21] Davi Calixto: // SPDX-License-Identifier: GPL-3.0
  pragma solidity ^0.8.20;

  /**
  * @title Calixto Super Token (CALX)
  * @dev ERC-20 Token com supply inicial de 10,000,000 CALX
  * @notice Deploy via Remix IDE - https://remix.ethereum.org
  * 
  * Mainnet Address: 0x41A1F019d5fa7c8Ac87d40f8dC7A14f73a0861Fd
  * Total Supply: 10,000,000 CALX (10M)
  * Decimals: 18
  */

  // OpenZeppelin Contracts (importar via Remix)
  // @openzeppelin/contracts/token/ERC20/ERC20.sol
  // @openzeppelin/contracts/access/Ownable.sol

  interface IERC20 {
  function totalSupply() external view returns (uint256);
  function balanceOf(address account) external view returns (uint256);
  function transfer(address to, uint256 amount) external returns (bool);
  function allowance(address owner, address spender) external view returns (uint256);
  function approve(address spender, uint256 amount) external returns (bool);
  function transferFrom(address from, address to, uint256 amount) external returns (bool);

  event Transfer(address indexed from, address indexed to, uint256 value);
  event Approval(address indexed owner, address indexed spender, uint256 value);
  }

  interface IERC20Metadata is IERC20 {
  function name() external view returns (string memory);
  function symbol() external view returns (string memory);
  function decimals() external view returns (uint8);
  }

  abstract contract Context {
  function _msgSender() internal view virtual returns (address) {
  return msg.sender;
  }
  }

  abstract contract Ownable is Context {
  address private _owner;

  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  constructor(address initialOwner) {
  _transferOwnership(initialOwner);
  }

  modifier onlyOwner() {
  require(owner() == _msgSender(), "Ownable: caller is not the owner");
  _;
  }

  function owner() public view virtual returns (address) {
  return _owner;
  }

  function renounceOwnership() public virtual onlyOwner {
  _transferOwnership(address(0));
  }

  function transferOwnership(address newOwner) public virtual onlyOwner {
  require(newOwner != address(0), "Ownable: new owner is the zero address");
  _transferOwnership(newOwner);
  }

  function _transferOwnership(address newOwner) internal virtual {
  address oldOwner = _owner;
  _owner = newOwner;
  emit OwnershipTransferred(oldOwner, newOwner);
  }
  }

  contract ERC20 is Context, IERC20, IERC20Metadata {
  mapping(address => uint256) private _balances;
  mapping(address => mapping(address => uint256)) private _allowances;

  uint256 private _totalSupply;
  string private _name;
  string private _symbol;

  constructor(string memory name_, string memory symbol_) {
  _name = name_;
  _symbol = symbol_;
  }

  function name() public view virtual override returns (string memory) {
  return _name;
  }

  function symbol() public view virtual override returns (string memory) {
  return _symbol;
  }

  function decimals() public view virtual override returns (uint8) {
  return 18;
  }

  function totalSupply() public view virtual override returns (uint256) {
  return _totalSupply;
  }

  function balanceOf(address account) public view virtual override returns (uint256) {
  return _balances[account];
  }

  function transfer(address to, uint256 amount) public virtual override returns (bool) {
  address owner = _msgSender();
  _transfer(owner, to, amount);
  return true;
  }

  function allowance(address owner, address spender) public view virtual override returns (uint256) {
  return _allowances[owner][spender];
  }

  function approve(address spender, uint256 amount) public virtual override returns (bool) {
  address owner = _msgSender();
  _approve(owner, spender, amount);
  return true;
  }

  function transferFrom(address from, address to, uint256 amount) public virtual override returns (bool) {
  address spender = _msgSender();
  _spendAllowance(from, spender, amount);
  _transfer(from, to, amount);
  return true;
  }

  function _transfer(address from, address to, uint256 amount) internal virtual {
  require(from != address(0), "ERC20: transfer from the zero address");
  require(to != address(0), "ERC20: transfer to the zero address");

  uint256 fromBalance = _balances[from];
  require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");
  unchecked {
      _balances[from] = fromBalance - amount;
      _balances[to] += amount;
  }

  emit Transfer(from, to, amount);
  }

  function _mint(address account, uint256 amount) internal virtual {
  require(account != address(0), "ERC20: mint to the zero address");

  _totalSupply += amount;
  unchecked {
      _balances[account] += amount;
  }
  emit Transfer(address(0), account, amount);
  }

  function _burn(address account, uint256 amount) internal virtual {
  require(account != address(0), "ERC20: burn from the zero address");

  uint256 accountBalance = _balances[account];
  require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
  unchecked {
      _balances[account] = accountBalance - amount;
      _totalSupply -= amount;
  }

  emit Transfer(account, address(0), amount);
  }

  function _approve(address owner, address spender, uint256 amount) internal virtual {
  require(owner != address(0), "ERC20: approve from the zero address");
  require(spender != address(0), "ERC20: approve to the zero address");

  _allowances[owner][spender] = amount;
  emit Approval(owner, spender, amount);
  }

  function _spendAllowance(address owner, address spender, uint256 amount) internal virtual {
  uint256 currentAllowance = allowance(owner, spender);
  if (currentAllowance != type(uint256).max) {
      require(currentAllowance >= amount, "ERC20: insufficient allowance");
      unchecked {
          _approve(owner, spender, currentAllowance - amount);
      }
  }
  }
  }

  /**
  * @title CalixtoToken
  * @dev Token principal da rede Calixto Super
  */
  contract CalixtoToken is ERC20, Ownable {

  // Eventos customizados
  event TokensMinted(address indexed to, uint256 amount);
  event TokensBurned(address indexed from, uint256 amount);

  /**
  * @dev Constructor - Deploy com supply inicial
  * @param initialRecipient Endereço que recebe o supply inicial
  * @param initialSupply Supply inicial (sem decimais - será multiplicado por 10^18)
  */
  constructor(
  address initialRecipient,
  uint256 initialSupply
  ) payable ERC20("Calixto Super", "CALX") Ownable(msg.sender) {
  require(initialRecipient != address(0), "Invalid recipient address");
  require(initialSupply > 0, "Initial supply must be greater than 0");

  // Mint supply inicial (com 18 decimais)
  _mint(initialRecipient, initialSupply * 10**18);

  emit TokensMinted(initialRecipient, initialSupply * 10**18);
  }

  /**
  * @dev Mint novos tokens (apenas owner)
  * @param to Endereço destinatário
  * @param amount Quantidade (sem decimais)
  */
  function mint(address to, uint256 amount) public onlyOwner {
  require(to != address(0), "Cannot mint to zero address");
  _mint(to, amount * 10**18);
  emit TokensMinted(to, amount * 10**18);
  }

  /**
  * @dev Burn tokens do caller
  * @param amount Quantidade a queimar (sem decimais)
  */
  function burn(uint256 amount) public {
  _burn(msg.sender, amount * 10**18);
  emit TokensBurned(msg.sender, amount * 10**18);
  }

  /**
  * @dev Burn tokens de outro endereço (com allowance)
  * @param from Endereço de origem
  * @param amount Quantidade a queimar (sem decimais)
  */
  function burnFrom(address from, uint256 amount) public {
  _spendAllowance(from, msg.sender, amount * 10**18);
  _burn(from, amount * 10**18);
  emit TokensBurned(from, amount * 10**18);
  }

  /**
  * @dev Retorna informações do token
  */
  function tokenInfo() public pure returns (
  string memory tokenName,
  string memory tokenSymbol,
  uint8 tokenDecimals,
  string memory network
  ) {
  return ("Calixto Super", "CALX", 18, "Calixto Mainnet");
  }
  }+ Corrigir Contrato genesis para endereço 0x51de5828d345bdcff865f0d6b793a6fbd1214444 + Migrar todos TX token transações e contrato bem como saldo para 0x51de5828d345bdcff865f0d6b793a6fbd1214444 tudo real
[19/3 00:28] Davi Calixto: const { ethers } = require("ethers");

// Configurações
const RPC_URL = "SUA_RPC_URL_CALIXTO";
const PRIVATE_KEY = "CHAVE_PRIVADA_DA_CARTEIRA_0x51de5828d345bdcff865f0d6b793a6fbd1214444";
const OLD_CONTRACT = "0x41A1F019d5fa7c8Ac87d40f8dC7A14f73a0861Fd";
const NEW_CONTRACT = "0x51de5828d345bdcff865f0d6b793a6fbd1214444";

const ABI = ["function balanceOf(address) view returns (uint256)", "function transfer(address, uint256) returns (bool)"];

async function migrate() {
    const provider = new ethers.JsonRpcProvider(RPC_URL);
    const wallet = new ethers.Wallet(PRIVATE_KEY, provider);
    
    const oldToken = new ethers.Contract(OLD_CONTRACT, ABI, provider);
    const newToken = new ethers.Contract(NEW_CONTRACT, ABI, wallet);

    // LISTA DE HOLDERS (Você deve extrair do explorer da sua rede)
    const holders = [
        "0xEndereçoExemplo1...",
        "0xEndereçoExemplo2..."
    ];

    for (let holder of holders) {
        const balance = await oldToken.balanceOf(holder);
        if (balance > 0) {
            console.log(`Migrando ${ethers.formatEther(balance)} CALX para ${holder}`);
            const tx = await newToken.transfer(holder, balance);
            await tx.wait();
            console.log("Sucesso!");
        }
    }
}

migrate();genesis
[23/3 13:42] Davi Calixto: 0xF4e3d87A85137C6E72672b14AeB0BBDfc391b6E0
[28/3 12:49] Davi Calixto: curl "https://api.etherscan.io/v2/api?chainid=1&module=stats&action=tokensupply&contractaddress=0x57d90b64a1a57749b0f932f1a3395792e12e7055&apikey=YourApiKeyToken"
[30/3 17:20] Davi Calixto: {"language":"Solidity","settings":{"optimizer":{"enabled":true,"runs":200},"outputSelection":{"*":{"*":["*"]}},"metadata":{"bytecodeHash":"none"},"evmVersion":"paris"},"sources":{"(Token).sol":{"content":"/*\nCalixto Main Net\n*/\n\n\n// SPDX-License-Identifier: No License\npragma solidity 0.8.25;\n\nimport {IERC20, ERC20} from \"./ERC20.sol\";\nimport {ERC20Burnable} from \"./ERC20Burnable.sol\";\nimport {Ownable, Ownable2Step} from \"./Ownable2Step.sol\";\nimport {Mintable} from \"./Mintable.sol\";\nimport {SafeERC20Remastered} from \"./SafeERC20Remastered.sol\";\n\nimport {Initializable} from \"./Initializable.sol\";\nimport \"./IUniswapV2Factory.sol\";\nimport \"./IUniswapV2Pair.sol\";\nimport \"./IUniswapV2Router01.sol\";\nimport \"./IUniswapV2Router02.sol\";\n\ncontract Calixtosuper is ERC20, ERC20Burnable, Ownable2Step, Mintable, Initializable {\n    \n    using SafeERC20Remastered for IERC20;\n \n    uint16 public swapThresholdRatio;\n    \n    uint256 private _calixtoPending;\n    uint256 private _liquidityPending;\n\n    address public calixtoAddress;\n    uint16[3] public calixtoFees;\n\n    uint16[3] public liquidityFees;\n\n    mapping (address => bool) public isExcludedFromFees;\n\n    uint16[3] public totalFees;\n    bool private _swapping;\n\n    IUniswapV2Router02 public routerV2;\n    address public pairV2;\n    mapping (address => bool) public AMMs;\n \n    error InvalidAmountToRecover(uint256 amount, uint256 maxAmount);\n\n    error InvalidToken(address tokenAddress);\n\n    error CannotDepositNativeCoins(address account);\n\n    error InvalidSwapThresholdRatio(uint16 swapThresholdRatio);\n\n    error InvalidTaxRecipientAddress(address account);\n\n    error CannotExceedMaxTotalFee(uint16 buyFee, uint16 sellFee, uint16 transferFee);\n\n    error InvalidAMM(address AMM);\n \n    event SwapThresholdUpdated(uint16 swapThresholdRatio);\n\n    event WalletTaxAddressUpdated(uint8 indexed id, address newAddress);\n    event WalletTaxFeesUpdated(uint8 indexed id, uint16 buyFee, uint16 sellFee, uint16 transferFee);\n    event WalletTaxSent(uint8 indexed id, address recipient, uint256 amount);\n\n    event LiquidityFeesUpdated(uint16 buyFee, uint16 sellFee, uint16 transferFee);\n    event LiquidityAdded(uint amountToken, uint amountCoin, uint liquidity);\n    event ForceLiquidityAdded(uint256 leftoverTokens, uint256 unaddedTokens);\n\n    event ExcludeFromFees(address indexed account, bool isExcluded);\n\n    event RouterV2Updated(address indexed routerV2);\n    event AMMUpdated(address indexed AMM, bool isAMM);\n \n    constructor()\n        ERC20(unicode\"Calixtosuper\", unicode\"Calxt\")\n        Ownable(msg.sender)\n        Mintable(10000000000)\n    {\n        assembly { if iszero(extcodesize(caller())) { revert(0, 0) } }\n        address supplyRecipient = 0xF4e3d87A85137C6E72672b14AeB0BBDfc391b6E0;\n        \n        updateSwapThreshold(50);\n\n        calixtoAddressSetup(0xF4e3d87A85137C6E72672b14AeB0BBDfc391b6E0);\n        calixtoFeesSetup(1, 1, 1);\n\n        liquidityFeesSetup(10, 1, 1);\n\n        excludeFromFees(supplyRecipient, true);\n        excludeFromFees(address(this), true); \n\n        _mint(supplyRecipient, 10000000000 * (10 ** decimals()) / 10);\n        _transferOwnership(0xF4e3d87A85137C6E72672b14AeB0BBDfc391b6E0);\n    }\n    \n    /*\n        This token is not upgradeable. Function afterConstructor finishes post-deployment setup.\n    */\n    function afterConstructor(address _router) initializer external {\n        _updateRouterV2(_router);\n    }\n\n    function deploymentKey() external pure returns (bytes32) {\n        return 0x31abed4a0d56ce7e4a11fb77cde05a1ffee58d57624d0b615693b06a7bd61f03;\n    }\n\n    function decimals() public pure override returns (uint8) {\n        return 18;\n    }\n    \n    function recoverToken(uint256 amount) external onlyOwner {\n        uint256 maxRecoverable = balanceOf(address(this)) - getAllPending();\n        if (amount > maxRecoverable) revert InvalidAmountToRecover(amount, maxRecoverable);\n\n        _update(address(this), msg.sender, amount);\n    }\n\n    function recoverForeignERC20(address tokenAddress, uint256 amount) external onlyOwner {\n        if (tokenAddress == address(this)) revert InvalidToken(tokenAddress);\n\n        IERC20(tokenAddress).safeTransfer(msg.sender, amount);\n    }\n\n    // Prevent unintended coin transfers\n    receive() external payable {\n        if (msg.sender != address(routerV2)) revert CannotDepositNativeCoins(msg.sender);\n    }\n\n    function _swapTokensForCoin(uint256 tokenAmount) private {\n        address[] memory path = new address[](2);\n        path[0] = address(this);\n        path[1] = routerV2.WETH();\n        \n        routerV2.swapExactTokensForETHSupportingFeeOnTransferTokens(tokenAmount, 0, path, address(this), block.timestamp);\n    }\n\n    function updateSwapThreshold(uint16 _swapThresholdRatio) public onlyOwner {\n        if (_swapThresholdRatio == 0 || _swapThresholdRatio > 500) revert InvalidSwapThresholdRatio(_swapThresholdRatio);\n\n        swapThresholdRatio = _swapThresholdRatio;\n        \n        emit SwapThresholdUpdated(_swapThresholdRatio);\n    }\n\n    function getSwapThresholdAmount() public view returns (uint256) {\n        return balanceOf(pairV2) * swapThresholdRatio / 10000;\n    }\n\n    function getAllPending() public view returns (uint256) {\n        return 0 + _calixtoPending + _liquidityPending;\n    }\n\n    function calixtoAddressSetup(address _newAddress) public onlyOwner {\n        if (_newAddress == address(0)) revert InvalidTaxRecipientAddress(address(0));\n\n        calixtoAddress = _newAddress;\n        excludeFromFees(_newAddress, true);\n\n        emit WalletTaxAddressUpdated(1, _newAddress);\n    }\n\n    function calixtoFeesSetup(uint16 _buyFee, uint16 _sellFee, uint16 _transferFee) public onlyOwner {\n        totalFees[0] = totalFees[0] - calixtoFees[0] + _buyFee;\n        totalFees[1] = totalFees[1] - calixtoFees[1] + _sellFee;\n        totalFees[2] = totalFees[2] - calixtoFees[2] + _transferFee;\n        if (totalFees[0] > 2500 || totalFees[1] > 2500 || totalFees[2] > 2500) revert CannotExceedMaxTotalFee(totalFees[0], totalFees[1], totalFees[2]);\n\n        calixtoFees = [_buyFee, _sellFee, _transferFee];\n\n        emit WalletTaxFeesUpdated(1, _buyFee, _sellFee, _transferFee);\n    }\n\n    function _swapAndLiquify(uint256 tokenAmount) private returns (uint256 leftover) {\n        // Sub-optimal method for supplying liquidity\n        uint256 halfAmount = tokenAmount / 2;\n        uint256 otherHalf = tokenAmount - halfAmount;\n\n        _swapTokensForCoin(halfAmount);\n\n        uint256 coinBalance = address(this).balance;\n\n        if (coinBalance > 0) {\n            (uint amountToken, uint amountCoin, uint liquidity) = _addLiquidity(otherHalf, coinBalance);\n\n            emit LiquidityAdded(amountToken, amountCoin, liquidity);\n\n            return otherHalf - amountToken;\n        } else {\n            return otherHalf;\n        }\n    }\n\n    function _addLiquidity(uint256 tokenAmount, uint256 coinAmount) private returns (uint, uint, uint) {\n        return routerV2.addLiquidityETH{value: coinAmount}(address(this), tokenAmount, 0, 0, address(0xdead), block.timestamp);\n    }\n\n    function addLiquidityFromLeftoverTokens() external {\n        uint256 leftoverTokens = balanceOf(address(this)) - getAllPending();\n\n        uint256 unaddedTokens = _swapAndLiquify(leftoverTokens);\n\n        emit ForceLiquidityAdded(leftoverTokens, unaddedTokens);\n    }\n\n    function liquidityFeesSetup(uint16 _buyFee, uint16 _sellFee, uint16 _transferFee) public onlyOwner {\n        totalFees[0] = totalFees[0] - liquidityFees[0] + _buyFee;\n        totalFees[1] = totalFees[1] - liquidityFees[1] + _sellFee;\n        totalFees[2] = totalFees[2] - liquidityFees[2] + _transferFee;\n        if (totalFees[0] > 2500 || totalFees[1] > 2500 || totalFees[2] > 2500) revert CannotExceedMaxTotalFee(totalFees[0], totalFees[1], totalFees[2]);\n\n        liquidityFees = [_buyFee, _sellFee, _transferFee];\n\n        emit LiquidityFeesUpdated(_buyFee, _sellFee, _transferFee);\n    }\n\n    function excludeFromFees(address account, bool isExcluded) public onlyOwner {\n        isExcludedFromFees[account] = isExcluded;\n        \n        emit ExcludeFromFees(account, isExcluded);\n    }\n\n    function _updateRouterV2(address router) private {\n        routerV2 = IUniswapV2Router02(router);\n        pairV2 = IUniswapV2Factory(routerV2.factory()).createPair(address(this), routerV2.WETH());\n\n        _approve(address(this), router, type(uint256).max);\n        _setAMM(router, true);\n        _setAMM(pairV2, true);\n\n        emit RouterV2Updated(router);\n    }\n\n    function setAMM(address AMM, bool isAMM) external onlyOwner {\n        if (AMM == pairV2 || AMM == address(routerV2)) revert InvalidAMM(AMM);\n\n        _setAMM(AMM, isAMM);\n    }\n\n    function _setAMM(address AMM, bool isAMM) private {\n        AMMs[AMM] = isAMM;\n\n        if (isAMM) { \n        }\n\n        emit AMMUpdated(AMM, isAMM);\n    }\n\n\n    function _update(address from, address to, uint256 amount)\n        internal\n        override\n    {\n        _beforeTokenUpdate(from, to, amount);\n        \n        if (from != address(0) && to != address(0)) {\n            if (!_swapping && amount > 0 && !isExcludedFromFees[from] && !isExcludedFromFees[to]) {\n                uint256 fees = 0;\n                uint8 txType = 3;\n                \n                if (AMMs[from] && !AMMs[to]) {\n                    if (totalFees[0] > 0) txType = 0;\n                }\n                else if (AMMs[to] && !AMMs[from]) {\n                    if (totalFees[1] > 0) txType = 1;\n                }\n                else if (!AMMs[from] && !AMMs[to]) {\n                    if (totalFees[2] > 0) txType = 2;\n                }\n                \n                if (txType < 3) {\n                    \n                    fees = amount * totalFees[txType] / 10000;\n                    amount -= fees;\n                    \n                    _calixtoPending += fees * calixtoFees[txType] / totalFees[txType];\n\n                    _liquidityPending += fees * liquidityFees[txType] / totalFees[txType];\n\n                    \n                }\n\n                if (fees > 0) {\n                    super._update(from, address(this), fees);\n                }\n            }\n            \n            bool canSwap = getAllPending() >= getSwapThresholdAmount() && balanceOf(pairV2) > 0;\n            \n            if (!_swapping && from != pairV2 && from != address(routerV2) && canSwap) {\n                _swapping = true;\n                \n                if (false || _calixtoPending > 0) {\n                    uint256 token2Swap = 0 + _calixtoPending;\n                    bool success = false;\n\n                    _swapTokensForCoin(token2Swap);\n                    uint256 coinsReceived = address(this).balance;\n                    \n                    uint256 calixtoPortion = coinsReceived * _calixtoPending / token2Swap;\n                    if (calixtoPortion > 0) {\n                        (success,) = payable(calixtoAddress).call{value: calixtoPortion, gas: 20000}(\"\");\n                        if (success) {\n                            emit WalletTaxSent(1, calixtoAddress, calixtoPortion);\n                        }\n                    }\n                    _calixtoPending = 0;\n\n                }\n\n                if (_liquidityPending > 0) {\n                    _swapAndLiquify(_liquidityPending);\n                    _liquidityPending = 0;\n                }\n\n                _swapping = false;\n            }\n\n        }\n\n        super._update(from, to, amount);\n        \n        _afterTokenUpdate(from, to, amount);\n        \n    }\n\n    function _beforeTokenUpdate(address from, address to, uint256 amount)\n        internal\n        view\n    {\n    }\n\n    function _afterTokenUpdate(address from, address to, uint256 amount)\n        internal\n    {\n        if (from == address(0)) {\n        }\n\n    }\n}\n"},"ERC20.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/ERC20.sol)\n\npragma solidity ^0.8.20;\n\nimport {IERC20} from \"./IERC20.sol\";\nimport {IERC20Metadata} from \"./IERC20Metadata.sol\";\nimport {Context} from \"./Context.sol\";\nimport {IERC20Errors} from \"./draft-IERC6093.sol\";\n\n/**\n * @dev Implementation of the {IERC20} interface.\n *\n * This implementation is agnostic to the way tokens are created. This means\n * that a supply mechanism has to be added in a derived contract using {_mint}.\n *\n * TIP: For a detailed writeup see our guide\n * https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226[How\n * to implement supply mechanisms].\n *\n * The default value of {decimals} is 18. To change this, you should override\n * this function so it returns a different value.\n *\n * We have followed general OpenZeppelin Contracts guidelines: functions revert\n * instead returning `false` on failure. This behavior is nonetheless\n * conventional and does not conflict with the expectations of ERC20\n * applications.\n *\n * Additionally, an {Approval} event is emitted on calls to {transferFrom}.\n * This allows applications to reconstruct the allowance for all accounts just\n * by listening to said events. Other implementations of the EIP may not emit\n * these events, as it isn't required by the specification.\n */\nabstract contract ERC20 is Context, IERC20, IERC20Metadata, IERC20Errors {\n    mapping(address account => uint256) private _balances;\n\n    mapping(address account => mapping(address spender => uint256)) private _allowances;\n\n    uint256 private _totalSupply;\n\n    string private _name;\n    string private _symbol;\n\n    /**\n     * @dev Sets the values for {name} and {symbol}.\n     *\n     * All two of these values are immutable: they can only be set once during\n     * construction.\n     */\n    constructor(string memory name_, string memory symbol_) {\n        _name = name_;\n        _symbol = symbol_;\n    }\n\n    /**\n     * @dev Returns the name of the token.\n     */\n    function name() public view virtual returns (string memory) {\n        return _name;\n    }\n\n    /**\n     * @dev Returns the symbol of the token, usually a shorter version of the\n     * name.\n     */\n    function symbol() public view virtual returns (string memory) {\n        return _symbol;\n    }\n\n    /**\n     * @dev Returns the number of decimals used to get its user representation.\n     * For example, if `decimals` equals `2`, a balance of `505` tokens should\n     * be displayed to a user as `5.05` (`505 / 10 ** 2`).\n     *\n     * Tokens usually opt for a value of 18, imitating the relationship between\n     * Ether and Wei. This is the default value returned by this function, unless\n     * it's overridden.\n     *\n     * NOTE: This information is only used for _display_ purposes: it in\n     * no way affects any of the arithmetic of the contract, including\n     * {IERC20-balanceOf} and {IERC20-transfer}.\n     */\n    function decimals() public view virtual returns (uint8) {\n        return 18;\n    }\n\n    /**\n     * @dev See {IERC20-totalSupply}.\n     */\n    function totalSupply() public view virtual returns (uint256) {\n        return _totalSupply;\n    }\n\n    /**\n     * @dev See {IERC20-balanceOf}.\n     */\n    function balanceOf(address account) public view virtual returns (uint256) {\n        return _balances[account];\n    }\n\n    /**\n     * @dev See {IERC20-transfer}.\n     *\n     * Requirements:\n     *\n     * - `to` cannot be the zero address.\n     * - the caller must have a balance of at least `value`.\n     */\n    function transfer(address to, uint256 value) public virtual returns (bool) {\n        address owner = _msgSender();\n        _transfer(owner, to, value);\n        return true;\n    }\n\n    /**\n     * @dev See {IERC20-allowance}.\n     */\n    function allowance(address owner, address spender) public view virtual returns (uint256) {\n        return _allowances[owner][spender];\n    }\n\n    /**\n     * @dev See {IERC20-approve}.\n     *\n     * NOTE: If `value` is the maximum `uint256`, the allowance is not updated on\n     * `transferFrom`. This is semantically equivalent to an infinite approval.\n     *\n     * Requirements:\n     *\n     * - `spender` cannot be the zero address.\n     */\n    function approve(address spender, uint256 value) public virtual returns (bool) {\n        address owner = _msgSender();\n        _approve(owner, spender, value);\n        return true;\n    }\n\n    /**\n     * @dev See {IERC20-transferFrom}.\n     *\n     * Emits an {Approval} event indicating the updated allowance. This is not\n     * required by the EIP. See the note at the beginning of {ERC20}.\n     *\n     * NOTE: Does not update the allowance if the current allowance\n     * is the maximum `uint256`.\n     *\n     * Requirements:\n     *\n     * - `from` and `to` cannot be the zero address.\n     * - `from` must have a balance of at least `value`.\n     * - the caller must have allowance for ``from``'s tokens of at least\n     * `value`.\n     */\n    function transferFrom(address from, address to, uint256 value) public virtual returns (bool) {\n        address spender = _msgSender();\n        _spendAllowance(from, spender, value);\n        _transfer(from, to, value);\n        return true;\n    }\n\n    /**\n     * @dev Moves a `value` amount of tokens from `from` to `to`.\n     *\n     * This internal function is equivalent to {transfer}, and can be used to\n     * e.g. implement automatic token fees, slashing mechanisms, etc.\n     *\n     * Emits a {Transfer} event.\n     *\n     * NOTE: This function is not virtual, {_update} should be overridden instead.\n     */\n    function _transfer(address from, address to, uint256 value) internal {\n        if (from == address(0)) {\n            revert ERC20InvalidSender(address(0));\n        }\n        if (to == address(0)) {\n            revert ERC20InvalidReceiver(address(0));\n        }\n        _update(from, to, value);\n    }\n\n    /**\n     * @dev Transfers a `value` amount of tokens from `from` to `to`, or alternatively mints (or burns) if `from`\n     * (or `to`) is the zero address. All customizations to transfers, mints, and burns should be done by overriding\n     * this function.\n     *\n     * Emits a {Transfer} event.\n     */\n    function _update(address from, address to, uint256 value) internal virtual {\n        if (from == address(0)) {\n            // Overflow check required: The rest of the code assumes that totalSupply never overflows\n            _totalSupply += value;\n        } else {\n            uint256 fromBalance = _balances[from];\n            if (fromBalance < value) {\n                revert ERC20InsufficientBalance(from, fromBalance, value);\n            }\n            unchecked {\n                // Overflow not possible: value <= fromBalance <= totalSupply.\n                _balances[from] = fromBalance - value;\n            }\n        }\n\n        if (to == address(0)) {\n            unchecked {\n                // Overflow not possible: value <= totalSupply or value <= fromBalance <= totalSupply.\n                _totalSupply -= value;\n            }\n        } else {\n            unchecked {\n                // Overflow not possible: balance + value is at most totalSupply, which we know fits into a uint256.\n                _balances[to] += value;\n            }\n        }\n\n        emit Transfer(from, to, value);\n    }\n\n    /**\n     * @dev Creates a `value` amount of tokens and assigns them to `account`, by transferring it from address(0).\n     * Relies on the `_update` mechanism\n     *\n     * Emits a {Transfer} event with `from` set to the zero address.\n     *\n     * NOTE: This function is not virtual, {_update} should be overridden instead.\n     */\n    function _mint(address account, uint256 value) internal {\n        if (account == address(0)) {\n            revert ERC20InvalidReceiver(address(0));\n        }\n        _update(address(0), account, value);\n    }\n\n    /**\n     * @dev Destroys a `value` amount of tokens from `account`, lowering the total supply.\n     * Relies on the `_update` mechanism.\n     *\n     * Emits a {Transfer} event with `to` set to the zero address.\n     *\n     * NOTE: This function is not virtual, {_update} should be overridden instead\n     */\n    function _burn(address account, uint256 value) internal {\n        if (account == address(0)) {\n            revert ERC20InvalidSender(address(0));\n        }\n        _update(account, address(0), value);\n    }\n\n    /**\n     * @dev Sets `value` as the allowance of `spender` over the `owner` s tokens.\n     *\n     * This internal function is equivalent to `approve`, and can be used to\n     * e.g. set automatic allowances for certain subsystems, etc.\n     *\n     * Emits an {Approval} event.\n     *\n     * Requirements:\n     *\n     * - `owner` cannot be the zero address.\n     * - `spender` cannot be the zero address.\n     *\n     * Overrides to this logic should be done to the variant with an additional `bool emitEvent` argument.\n     */\n    function _approve(address owner, address spender, uint256 value) internal {\n        _approve(owner, spender, value, true);\n    }\n\n    /**\n     * @dev Variant of {_approve} with an optional flag to enable or disable the {Approval} event.\n     *\n     * By default (when calling {_approve}) the flag is set to true. On the other hand, approval changes made by\n     * `_spendAllowance` during the `transferFrom` operation set the flag to false. This saves gas by not emitting any\n     * `Approval` event during `transferFrom` operations.\n     *\n     * Anyone who wishes to continue emitting `Approval` events on the`transferFrom` operation can force the flag to\n     * true using the following override:\n     * ```\n     * function _approve(address owner, address spender, uint256 value, bool) internal virtual override {\n     *     super._approve(owner, spender, value, true);\n     * }\n     * ```\n     *\n     * Requirements are the same as {_approve}.\n     */\n    function _approve(address owner, address spender, uint256 value, bool emitEvent) internal virtual {\n        if (owner == address(0)) {\n            revert ERC20InvalidApprover(address(0));\n        }\n        if (spender == address(0)) {\n            revert ERC20InvalidSpender(address(0));\n        }\n        _allowances[owner][spender] = value;\n        if (emitEvent) {\n            emit Approval(owner, spender, value);\n        }\n    }\n\n    /**\n     * @dev Updates `owner` s allowance for `spender` based on spent `value`.\n     *\n     * Does not update the allowance value in case of infinite allowance.\n     * Revert if not enough allowance is available.\n     *\n     * Does not emit an {Approval} event.\n     */\n    function _spendAllowance(address owner, address spender, uint256 value) internal virtual {\n        uint256 currentAllowance = allowance(owner, spender);\n        if (currentAllowance != type(uint256).max) {\n            if (currentAllowance < value) {\n                revert ERC20InsufficientAllowance(spender, currentAllowance, value);\n            }\n            unchecked {\n                _approve(owner, spender, currentAllowance - value, false);\n            }\n        }\n    }\n}\n"},"ERC20Burnable.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/extensions/ERC20Burnable.sol)\n\npragma solidity ^0.8.20;\n\nimport {ERC20} from \"./ERC20.sol\";\nimport {Context} from \"./Context.sol\";\n\n/**\n * @dev Extension of {ERC20} that allows token holders to destroy both their own\n * tokens and those that they have an allowance for, in a way that can be\n * recognized off-chain (via event analysis).\n */\nabstract contract ERC20Burnable is Context, ERC20 {\n    /**\n     * @dev Destroys a `value` amount of tokens from the caller.\n     *\n     * See {ERC20-_burn}.\n     */\n    function burn(uint256 value) public virtual {\n        _burn(_msgSender(), value);\n    }\n\n    /**\n     * @dev Destroys a `value` amount of tokens from `account`, deducting from\n     * the caller's allowance.\n     *\n     * See {ERC20-_burn} and {ERC20-allowance}.\n     *\n     * Requirements:\n     *\n     * - the caller must have allowance for ``accounts``'s tokens of at least\n     * `value`.\n     */\n    function burnFrom(address account, uint256 value) public virtual {\n        _spendAllowance(account, _msgSender(), value);\n        _burn(account, value);\n    }\n}\n"},"Ownable2Step.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (access/Ownable2Step.sol)\n\npragma solidity ^0.8.20;\n\nimport {Ownable} from \"./Ownable.sol\";\n\n/**\n * @dev Contract module which provides access control mechanism, where\n * there is an account (an owner) that can be granted exclusive access to\n * specific functions.\n *\n * The initial owner is specified at deployment time in the constructor for `Ownable`. This\n * can later be changed with {transferOwnership} and {acceptOwnership}.\n *\n * This module is used through inheritance. It will make available all functions\n * from parent (Ownable).\n */\nabstract contract Ownable2Step is Ownable {\n    address private _pendingOwner;\n\n    event OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner);\n\n    /**\n     * @dev Returns the address of the pending owner.\n     */\n    function pendingOwner() public view virtual returns (address) {\n        return _pendingOwner;\n    }\n\n    /**\n     * @dev Starts the ownership transfer of the contract to a new account. Replaces the pending transfer if there is one.\n     * Can only be called by the current owner.\n     */\n    function transferOwnership(address newOwner) public virtual override onlyOwner {\n        _pendingOwner = newOwner;\n        emit OwnershipTransferStarted(owner(), newOwner);\n    }\n\n    /**\n     * @dev Transfers ownership of the contract to a new account (`newOwner`) and deletes any pending owner.\n     * Internal function without access restriction.\n     */\n    function _transferOwnership(address newOwner) internal virtual override {\n        delete _pendingOwner;\n        super._transferOwnership(newOwner);\n    }\n\n    /**\n     * @dev The new owner accepts the ownership transfer.\n     */\n    function acceptOwnership() public virtual {\n        address sender = _msgSender();\n        if (pendingOwner() != sender) {\n            revert OwnableUnauthorizedAccount(sender);\n        }\n        _transferOwnership(sender);\n    }\n}\n"},"Mintable.sol":{"content":"// SPDX-License-Identifier: No License\n\npragma solidity ^0.8.19;\n\nimport {ERC20} from \"./ERC20.sol\";\nimport {Ownable2Step} from \"./Ownable2Step.sol\";\n\nabstract contract Mintable is ERC20, Ownable2Step {\n\n    uint256 public maxSupply;\n\n    error MintCannotExceedMaxSupply();\n\n    constructor(uint256 _maxSupply) {\n        maxSupply = _maxSupply * (10 ** decimals()) / 10;\n    }\n\n    function mint(address to, uint256 amount) public onlyOwner {\n        if (totalSupply() + amount > maxSupply) revert MintCannotExceedMaxSupply();\n\n        _mint(to, amount);\n    }\n}"},"SafeERC20Remastered.sol":{"content":"// SPDX-License-Identifier: MIT\n// Remastered from OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/utils/SafeERC20.sol)\n\npragma solidity ^0.8.20;\n\nimport {IERC20} from \"./IERC20.sol\";\nimport {Address} from \"./Address.sol\";\n\nlibrary SafeERC20Remastered {\n    using Address for address;\n\n    /**\n     * @dev An operation with an ERC20 token failed.\n     */\n    error SafeERC20FailedOperation(address token);\n\n    /**\n     * @dev Transfer `value` amount of `token` from the calling contract to `to`. If `token` returns no value,\n     * non-reverting calls are assumed to be successful.\n     */\n    function safeTransfer(IERC20 token, address to, uint256 value) internal {\n        _callOptionalReturn(token, abi.encodeCall(token.transfer, (to, value)));\n    }\n\n    /**\n     * @dev Transfer `value` amount of `token` from the calling contract to `to`. If `token` returns no value,\n     * non-reverting calls are assumed to be successful.\n     */\n    function safeTransfer_noRevert(IERC20 token, address to, uint256 value) internal returns (bool) {\n        return _callOptionalReturnBool(token, abi.encodeCall(token.transfer, (to, value)));\n    }\n\n    /**\n     * @dev Transfer `value` amount of `token` from `from` to `to`, spending the approval given by `from` to the\n     * calling contract. If `token` returns no value, non-reverting calls are assumed to be successful.\n     */\n    function safeTransferFrom(IERC20 token, address from, address to, uint256 value) internal {\n        _callOptionalReturn(token, abi.encodeCall(token.transferFrom, (from, to, value)));\n    }\n\n    /**\n     * @dev Increase the calling contract's allowance toward `spender` by `value`. If `token` returns no value,\n     * non-reverting calls are assumed to be successful.\n     */\n    function safeIncreaseAllowance(IERC20 token, address spender, uint256 value) internal {\n        uint256 oldAllowance = token.allowance(address(this), spender);\n        forceApprove(token, spender, oldAllowance + value);\n    }\n\n    /**\n     * @dev Set the calling contract's allowance toward `spender` to `value`. If `token` returns no value,\n     * non-reverting calls are assumed to be successful. Meant to be used with tokens that require the approval\n     * to be set to zero before setting it to a non-zero value, such as USDT.\n     */\n    function forceApprove(IERC20 token, address spender, uint256 value) internal {\n        bytes memory approvalCall = abi.encodeCall(token.approve, (spender, value));\n\n        if (!_callOptionalReturnBool(token, approvalCall)) {\n            _callOptionalReturn(token, abi.encodeCall(token.approve, (spender, 0)));\n            _callOptionalReturn(token, approvalCall);\n        }\n    }\n\n    /**\n     * @dev Imitates a Solidity high-level call (i.e. a regular function call to a contract), relaxing the requirement\n     * on the return value: the return value is optional (but if data is returned, it must not be false).\n     * @param token The token targeted by the call.\n     * @param data The call data (encoded using abi.encode or one of its variants).\n     */\n    function _callOptionalReturn(IERC20 token, bytes memory data) private {\n        // We need to perform a low level call here, to bypass Solidity's return data size checking mechanism, since\n        // we're implementing it ourselves. We use {Address-functionCall} to perform this call, which verifies that\n        // the target address contains contract code and also asserts for success in the low-level call.\n\n        bytes memory returndata = address(token).functionCall(data);\n        if (returndata.length != 0 && !abi.decode(returndata, (bool))) {\n            revert SafeERC20FailedOperation(address(token));\n        }\n    }\n\n    /**\n     * @dev Imitates a Solidity high-level call (i.e. a regular function call to a contract), relaxing the requirement\n     * on the return value: the return value is optional (but if data is returned, it must not be false).\n     * @param token The token targeted by the call.\n     * @param data The call data (encoded using abi.encode or one of its variants).\n     *\n     * This is a variant of {_callOptionalReturn} that silents catches all reverts and returns a bool instead.\n     */\n    function _callOptionalReturnBool(IERC20 token, bytes memory data) private returns (bool) {\n        // We need to perform a low level call here, to bypass Solidity's return data size checking mechanism, since\n        // we're implementing it ourselves. We cannot use {Address-functionCall} here since this should return false\n        // and not revert is the subcall reverts.\n\n        (bool success, bytes memory returndata) = address(token).call(data);\n        return success && (returndata.length == 0 || abi.decode(returndata, (bool))) && address(token).code.length > 0;\n    }\n}\n"},"Initializable.sol":{"content":"// SPDX-License-Identifier: MIT\n\npragma solidity ^0.8.19;\n\nabstract contract Initializable {\n\n    /**\n     * @dev Indicates that the contract has been initialized.\n     */\n    bool private _initialized;\n\n    /**\n     * @dev Indicates that the contract is in the process of being initialized.\n     */\n    bool private _initializing;\n\n    /**\n     * @dev Modifier to protect an initializer function from being invoked twice.\n     */\n    modifier initializer() {\n        require(_initializing || !_initialized, \"Initializable: contract is already initialized\");\n\n        bool isTopLevelCall = !_initializing;\n        if (isTopLevelCall) {\n            _initializing = true;\n            _initialized = true;\n        }\n\n        _;\n\n        if (isTopLevelCall) {\n            _initializing = false;\n        }\n    }\n}"},"IUniswapV2Factory.sol":{"content":"pragma solidity >=0.5.0;\n\ninterface IUniswapV2Factory {\n    event PairCreated(address indexed token0, address indexed token1, address pair, uint);\n\n    function feeTo() external view returns (address);\n    function feeToSetter() external view returns (address);\n\n    function getPair(address tokenA, address tokenB) external view returns (address pair);\n    function allPairs(uint) external view returns (address pair);\n    function allPairsLength() external view returns (uint);\n\n    function createPair(address tokenA, address tokenB) external returns (address pair);\n\n    function setFeeTo(address) external;\n    function setFeeToSetter(address) external;\n}\n"},"IUniswapV2Pair.sol":{"content":"pragma solidity >=0.5.0;\n\ninterface IUniswapV2Pair {\n    event Approval(address indexed owner, address indexed spender, uint value);\n    event Transfer(address indexed from, address indexed to, uint value);\n\n    function name() external pure returns (string memory);\n    function symbol() external pure returns (string memory);\n    function decimals() external pure returns (uint8);\n    function totalSupply() external view returns (uint);\n    function balanceOf(address owner) external view returns (uint);\n    function allowance(address owner, address spender) external view returns (uint);\n\n    function approve(address spender, uint value) external returns (bool);\n    function transfer(address to, uint value) external returns (bool);\n    function transferFrom(address from, address to, uint value) external returns (bool);\n\n    function DOMAIN_SEPARATOR() external view returns (bytes32);\n    function PERMIT_TYPEHASH() external pure returns (bytes32);\n    function nonces(address owner) external view returns (uint);\n\n    function permit(address owner, address spender, uint value, uint deadline, uint8 v, bytes32 r, bytes32 s) external;\n\n    event Mint(address indexed sender, uint amount0, uint amount1);\n    event Burn(address indexed sender, uint amount0, uint amount1, address indexed to);\n    event Swap(\n        address indexed sender,\n        uint amount0In,\n        uint amount1In,\n        uint amount0Out,\n        uint amount1Out,\n        address indexed to\n    );\n    event Sync(uint112 reserve0, uint112 reserve1);\n\n    function MINIMUM_LIQUIDITY() external pure returns (uint);\n    function factory() external view returns (address);\n    function token0() external view returns (address);\n    function token1() external view returns (address);\n    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);\n    function price0CumulativeLast() external view returns (uint);\n    function price1CumulativeLast() external view returns (uint);\n    function kLast() external view returns (uint);\n\n    function mint(address to) external returns (uint liquidity);\n    function burn(address to) external returns (uint amount0, uint amount1);\n    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) external;\n    function skim(address to) external;\n    function sync() external;\n\n    function initialize(address, address) external;\n}\n"},"IUniswapV2Router01.sol":{"content":"pragma solidity >=0.6.2;\n\ninterface IUniswapV2Router01 {\n    function factory() external pure returns (address);\n    function WETH() external pure returns (address);\n\n    function addLiquidity(\n        address tokenA,\n        address tokenB,\n        uint amountADesired,\n        uint amountBDesired,\n        uint amountAMin,\n        uint amountBMin,\n        address to,\n        uint deadline\n    ) external returns (uint amountA, uint amountB, uint liquidity);\n    function addLiquidityETH(\n        address token,\n        uint amountTokenDesired,\n        uint amountTokenMin,\n        uint amountETHMin,\n        address to,\n        uint deadline\n    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);\n    function removeLiquidity(\n        address tokenA,\n        address tokenB,\n        uint liquidity,\n        uint amountAMin,\n        uint amountBMin,\n        address to,\n        uint deadline\n    ) external returns (uint amountA, uint amountB);\n    function removeLiquidityETH(\n        address token,\n        uint liquidity,\n        uint amountTokenMin,\n        uint amountETHMin,\n        address to,\n        uint deadline\n    ) external returns (uint amountToken, uint amountETH);\n    function removeLiquidityWithPermit(\n        address tokenA,\n        address tokenB,\n        uint liquidity,\n        uint amountAMin,\n        uint amountBMin,\n        address to,\n        uint deadline,\n        bool approveMax, uint8 v, bytes32 r, bytes32 s\n    ) external returns (uint amountA, uint amountB);\n    function removeLiquidityETHWithPermit(\n        address token,\n        uint liquidity,\n        uint amountTokenMin,\n        uint amountETHMin,\n        address to,\n        uint deadline,\n        bool approveMax, uint8 v, bytes32 r, bytes32 s\n    ) external returns (uint amountToken, uint amountETH);\n    function swapExactTokensForTokens(\n        uint amountIn,\n        uint amountOutMin,\n        address[] calldata path,\n        address to,\n        uint deadline\n    ) external returns (uint[] memory amounts);\n    function swapTokensForExactTokens(\n        uint amountOut,\n        uint amountInMax,\n        address[] calldata path,\n        address to,\n        uint deadline\n    ) external returns (uint[] memory amounts);\n    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline)\n        external\n        payable\n        returns (uint[] memory amounts);\n    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline)\n        external\n        returns (uint[] memory amounts);\n    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline)\n        external\n        returns (uint[] memory amounts);\n    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline)\n        external\n        payable\n        returns (uint[] memory amounts);\n\n    function quote(uint amountA, uint reserveA, uint reserveB) external pure returns (uint amountB);\n    function getAmountOut(uint amountIn, uint reserveIn, uint reserveOut) external pure returns (uint amountOut);\n    function getAmountIn(uint amountOut, uint reserveIn, uint reserveOut) external pure returns (uint amountIn);\n    function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts);\n    function getAmountsIn(uint amountOut, address[] calldata path) external view returns (uint[] memory amounts);\n}\n"},"IUniswapV2Router02.sol":{"content":"pragma solidity >=0.6.2;\n\nimport './IUniswapV2Router01.sol';\n\ninterface IUniswapV2Router02 is IUniswapV2Router01 {\n    function removeLiquidityETHSupportingFeeOnTransferTokens(\n        address token,\n        uint liquidity,\n        uint amountTokenMin,\n        uint amountETHMin,\n        address to,\n        uint deadline\n    ) external returns (uint amountETH);\n    function removeLiquidityETHWithPermitSupportingFeeOnTransferTokens(\n        address token,\n        uint liquidity,\n        uint amountTokenMin,\n        uint amountETHMin,\n        address to,\n        uint deadline,\n        bool approveMax, uint8 v, bytes32 r, bytes32 s\n    ) external returns (uint amountETH);\n\n    function swapExactTokensForTokensSupportingFeeOnTransferTokens(\n        uint amountIn,\n        uint amountOutMin,\n        address[] calldata path,\n        address to,\n        uint deadline\n    ) external;\n    function swapExactETHForTokensSupportingFeeOnTransferTokens(\n        uint amountOutMin,\n        address[] calldata path,\n        address to,\n        uint deadline\n    ) external payable;\n    function swapExactTokensForETHSupportingFeeOnTransferTokens(\n        uint amountIn,\n        uint amountOutMin,\n        address[] calldata path,\n        address to,\n        uint deadline\n    ) external;\n}\n"},"IERC20.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/IERC20.sol)\n\npragma solidity ^0.8.20;\n\n/**\n * @dev Interface of the ERC20 standard as defined in the EIP.\n */\ninterface IERC20 {\n    /**\n     * @dev Emitted when `value` tokens are moved from one account (`from`) to\n     * another (`to`).\n     *\n     * Note that `value` may be zero.\n     */\n    event Transfer(address indexed from, address indexed to, uint256 value);\n\n    /**\n     * @dev Emitted when the allowance of a `spender` for an `owner` is set by\n     * a call to {approve}. `value` is the new allowance.\n     */\n    event Approval(address indexed owner, address indexed spender, uint256 value);\n\n    /**\n     * @dev Returns the value of tokens in existence.\n     */\n    function totalSupply() external view returns (uint256);\n\n    /**\n     * @dev Returns the value of tokens owned by `account`.\n     */\n    function balanceOf(address account) external view returns (uint256);\n\n    /**\n     * @dev Moves a `value` amount of tokens from the caller's account to `to`.\n     *\n     * Returns a boolean value indicating whether the operation succeeded.\n     *\n     * Emits a {Transfer} event.\n     */\n    function transfer(address to, uint256 value) external returns (bool);\n\n    /**\n     * @dev Returns the remaining number of tokens that `spender` will be\n     * allowed to spend on behalf of `owner` through {transferFrom}. This is\n     * zero by default.\n     *\n     * This value changes when {approve} or {transferFrom} are called.\n     */\n    function allowance(address owner, address spender) external view returns (uint256);\n\n    /**\n     * @dev Sets a `value` amount of tokens as the allowance of `spender` over the\n     * caller's tokens.\n     *\n     * Returns a boolean value indicating whether the operation succeeded.\n     *\n     * IMPORTANT: Beware that changing an allowance with this method brings the risk\n     * that someone may use both the old and the new allowance by unfortunate\n     * transaction ordering. One possible solution to mitigate this race\n     * condition is to first reduce the spender's allowance to 0 and set the\n     * desired value afterwards:\n     * https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729\n     *\n     * Emits an {Approval} event.\n     */\n    function approve(address spender, uint256 value) external returns (bool);\n\n    /**\n     * @dev Moves a `value` amount of tokens from `from` to `to` using the\n     * allowance mechanism. `value` is then deducted from the caller's\n     * allowance.\n     *\n     * Returns a boolean value indicating whether the operation succeeded.\n     *\n     * Emits a {Transfer} event.\n     */\n    function transferFrom(address from, address to, uint256 value) external returns (bool);\n}\n"},"Address.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (utils/Address.sol)\n\npragma solidity ^0.8.20;\n\n/**\n * @dev Collection of functions related to the address type\n */\nlibrary Address {\n    /**\n     * @dev The ETH balance of the account is not enough to perform the operation.\n     */\n    error AddressInsufficientBalance(address account);\n\n    /**\n     * @dev There's no code at `target` (it is not a contract).\n     */\n    error AddressEmptyCode(address target);\n\n    /**\n     * @dev A call to an address target failed. The target may have reverted.\n     */\n    error FailedInnerCall();\n\n    /**\n     * @dev Replacement for Solidity's `transfer`: sends `amount` wei to\n     * `recipient`, forwarding all available gas and reverting on errors.\n     *\n     * https://eips.ethereum.org/EIPS/eip-1884[EIP1884] increases the gas cost\n     * of certain opcodes, possibly making contracts go over the 2300 gas limit\n     * imposed by `transfer`, making them unable to receive funds via\n     * `transfer`. {sendValue} removes this limitation.\n     *\n     * https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/[Learn more].\n     *\n     * IMPORTANT: because control is transferred to `recipient`, care must be\n     * taken to not create reentrancy vulnerabilities. Consider using\n     * {ReentrancyGuard} or the\n     * https://solidity.readthedocs.io/en/v0.8.20/security-considerations.html#use-the-checks-effects-interactions-pattern[checks-effects-interactions pattern].\n     */\n    function sendValue(address payable recipient, uint256 amount) internal {\n        if (address(this).balance < amount) {\n            revert AddressInsufficientBalance(address(this));\n        }\n\n        (bool success, ) = recipient.call{value: amount}(\"\");\n        if (!success) {\n            revert FailedInnerCall();\n        }\n    }\n\n    /**\n     * @dev Performs a Solidity function call using a low level `call`. A\n     * plain `call` is an unsafe replacement for a function call: use this\n     * function instead.\n     *\n     * If `target` reverts with a revert reason or custom error, it is bubbled\n     * up by this function (like regular Solidity function calls). However, if\n     * the call reverted with no returned reason, this function reverts with a\n     * {FailedInnerCall} error.\n     *\n     * Returns the raw returned data. To convert to the expected return value,\n     * use https://solidity.readthedocs.io/en/latest/units-and-global-variables.html?highlight=abi.decode#abi-encoding-and-decoding-functions[`abi.decode`].\n     *\n     * Requirements:\n     *\n     * - `target` must be a contract.\n     * - calling `target` with `data` must not revert.\n     */\n    function functionCall(address target, bytes memory data) internal returns (bytes memory) {\n        return functionCallWithValue(target, data, 0);\n    }\n\n    /**\n     * @dev Same as {xref-Address-functionCall-address-bytes-}[`functionCall`],\n     * but also transferring `value` wei to `target`.\n     *\n     * Requirements:\n     *\n     * - the calling contract must have an ETH balance of at least `value`.\n     * - the called Solidity function must be `payable`.\n     */\n    function functionCallWithValue(address target, bytes memory data, uint256 value) internal returns (bytes memory) {\n        if (address(this).balance < value) {\n            revert AddressInsufficientBalance(address(this));\n        }\n        (bool success, bytes memory returndata) = target.call{value: value}(data);\n        return verifyCallResultFromTarget(target, success, returndata);\n    }\n\n    /**\n     * @dev Same as {xref-Address-functionCall-address-bytes-}[`functionCall`],\n     * but performing a static call.\n     */\n    function functionStaticCall(address target, bytes memory data) internal view returns (bytes memory) {\n        (bool success, bytes memory returndata) = target.staticcall(data);\n        return verifyCallResultFromTarget(target, success, returndata);\n    }\n\n    /**\n     * @dev Same as {xref-Address-functionCall-address-bytes-}[`functionCall`],\n     * but performing a delegate call.\n     */\n    function functionDelegateCall(address target, bytes memory data) internal returns (bytes memory) {\n        (bool success, bytes memory returndata) = target.delegatecall(data);\n        return verifyCallResultFromTarget(target, success, returndata);\n    }\n\n    /**\n     * @dev Tool to verify that a low level call to smart-contract was successful, and reverts if the target\n     * was not a contract or bubbling up the revert reason (falling back to {FailedInnerCall}) in case of an\n     * unsuccessful call.\n     */\n    function verifyCallResultFromTarget(\n        address target,\n        bool success,\n        bytes memory returndata\n    ) internal view returns (bytes memory) {\n        if (!success) {\n            _revert(returndata);\n        } else {\n            // only check if target is a contract if the call was successful and the return data is empty\n            // otherwise we already know that it was a contract\n            if (returndata.length == 0 && target.code.length == 0) {\n                revert AddressEmptyCode(target);\n            }\n            return returndata;\n        }\n    }\n\n    /**\n     * @dev Tool to verify that a low level call was successful, and reverts if it wasn't, either by bubbling the\n     * revert reason or with a default {FailedInnerCall} error.\n     */\n    function verifyCallResult(bool success, bytes memory returndata) internal pure returns (bytes memory) {\n        if (!success) {\n            _revert(returndata);\n        } else {\n            return returndata;\n        }\n    }\n\n    /**\n     * @dev Reverts with returndata if present. Otherwise reverts with {FailedInnerCall}.\n     */\n    function _revert(bytes memory returndata) private pure {\n        // Look for revert reason and bubble it up if present\n        if (returndata.length > 0) {\n            // The easiest way to bubble the revert reason is using memory via assembly\n            /// @solidity memory-safe-assembly\n            assembly {\n                let returndata_size := mload(returndata)\n                revert(add(32, returndata), returndata_size)\n            }\n        } else {\n            revert FailedInnerCall();\n        }\n    }\n}\n"},"IERC20Metadata.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (token/ERC20/extensions/IERC20Metadata.sol)\n\npragma solidity ^0.8.20;\n\nimport {IERC20} from \"./IERC20.sol\";\n\n/**\n * @dev Interface for the optional metadata functions from the ERC20 standard.\n */\ninterface IERC20Metadata is IERC20 {\n    /**\n     * @dev Returns the name of the token.\n     */\n    function name() external view returns (string memory);\n\n    /**\n     * @dev Returns the symbol of the token.\n     */\n    function symbol() external view returns (string memory);\n\n    /**\n     * @dev Returns the decimals places of the token.\n     */\n    function decimals() external view returns (uint8);\n}\n"},"Context.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.1) (utils/Context.sol)\n\npragma solidity ^0.8.20;\n\n/**\n * @dev Provides information about the current execution context, including the\n * sender of the transaction and its data. While these are generally available\n * via msg.sender and msg.data, they should not be accessed in such a direct\n * manner, since when dealing with meta-transactions the account sending and\n * paying for execution may not be the actual sender (as far as an application\n * is concerned).\n *\n * This contract is only required for intermediate, library-like contracts.\n */\nabstract contract Context {\n    function _msgSender() internal view virtual returns (address) {\n        return msg.sender;\n    }\n\n    function _msgData() internal view virtual returns (bytes calldata) {\n        return msg.data;\n    }\n\n    function _contextSuffixLength() internal view virtual returns (uint256) {\n        return 0;\n    }\n}\n"},"draft-IERC6093.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (interfaces/draft-IERC6093.sol)\npragma solidity ^0.8.20;\n\n/**\n * @dev Standard ERC20 Errors\n * Interface of the https://eips.ethereum.org/EIPS/eip-6093[ERC-6093] custom errors for ERC20 tokens.\n */\ninterface IERC20Errors {\n    /**\n     * @dev Indicates an error related to the current `balance` of a `sender`. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     * @param balance Current balance for the interacting account.\n     * @param needed Minimum amount required to perform a transfer.\n     */\n    error ERC20InsufficientBalance(address sender, uint256 balance, uint256 needed);\n\n    /**\n     * @dev Indicates a failure with the token `sender`. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     */\n    error ERC20InvalidSender(address sender);\n\n    /**\n     * @dev Indicates a failure with the token `receiver`. Used in transfers.\n     * @param receiver Address to which tokens are being transferred.\n     */\n    error ERC20InvalidReceiver(address receiver);\n\n    /**\n     * @dev Indicates a failure with the `spender`’s `allowance`. Used in transfers.\n     * @param spender Address that may be allowed to operate on tokens without being their owner.\n     * @param allowance Amount of tokens a `spender` is allowed to operate with.\n     * @param needed Minimum amount required to perform a transfer.\n     */\n    error ERC20InsufficientAllowance(address spender, uint256 allowance, uint256 needed);\n\n    /**\n     * @dev Indicates a failure with the `approver` of a token to be approved. Used in approvals.\n     * @param approver Address initiating an approval operation.\n     */\n    error ERC20InvalidApprover(address approver);\n\n    /**\n     * @dev Indicates a failure with the `spender` to be approved. Used in approvals.\n     * @param spender Address that may be allowed to operate on tokens without being their owner.\n     */\n    error ERC20InvalidSpender(address spender);\n}\n\n/**\n * @dev Standard ERC721 Errors\n * Interface of the https://eips.ethereum.org/EIPS/eip-6093[ERC-6093] custom errors for ERC721 tokens.\n */\ninterface IERC721Errors {\n    /**\n     * @dev Indicates that an address can't be an owner. For example, `address(0)` is a forbidden owner in EIP-20.\n     * Used in balance queries.\n     * @param owner Address of the current owner of a token.\n     */\n    error ERC721InvalidOwner(address owner);\n\n    /**\n     * @dev Indicates a `tokenId` whose `owner` is the zero address.\n     * @param tokenId Identifier number of a token.\n     */\n    error ERC721NonexistentToken(uint256 tokenId);\n\n    /**\n     * @dev Indicates an error related to the ownership over a particular token. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     * @param tokenId Identifier number of a token.\n     * @param owner Address of the current owner of a token.\n     */\n    error ERC721IncorrectOwner(address sender, uint256 tokenId, address owner);\n\n    /**\n     * @dev Indicates a failure with the token `sender`. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     */\n    error ERC721InvalidSender(address sender);\n\n    /**\n     * @dev Indicates a failure with the token `receiver`. Used in transfers.\n     * @param receiver Address to which tokens are being transferred.\n     */\n    error ERC721InvalidReceiver(address receiver);\n\n    /**\n     * @dev Indicates a failure with the `operator`’s approval. Used in transfers.\n     * @param operator Address that may be allowed to operate on tokens without being their owner.\n     * @param tokenId Identifier number of a token.\n     */\n    error ERC721InsufficientApproval(address operator, uint256 tokenId);\n\n    /**\n     * @dev Indicates a failure with the `approver` of a token to be approved. Used in approvals.\n     * @param approver Address initiating an approval operation.\n     */\n    error ERC721InvalidApprover(address approver);\n\n    /**\n     * @dev Indicates a failure with the `operator` to be approved. Used in approvals.\n     * @param operator Address that may be allowed to operate on tokens without being their owner.\n     */\n    error ERC721InvalidOperator(address operator);\n}\n\n/**\n * @dev Standard ERC1155 Errors\n * Interface of the https://eips.ethereum.org/EIPS/eip-6093[ERC-6093] custom errors for ERC1155 tokens.\n */\ninterface IERC1155Errors {\n    /**\n     * @dev Indicates an error related to the current `balance` of a `sender`. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     * @param balance Current balance for the interacting account.\n     * @param needed Minimum amount required to perform a transfer.\n     * @param tokenId Identifier number of a token.\n     */\n    error ERC1155InsufficientBalance(address sender, uint256 balance, uint256 needed, uint256 tokenId);\n\n    /**\n     * @dev Indicates a failure with the token `sender`. Used in transfers.\n     * @param sender Address whose tokens are being transferred.\n     */\n    error ERC1155InvalidSender(address sender);\n\n    /**\n     * @dev Indicates a failure with the token `receiver`. Used in transfers.\n     * @param receiver Address to which tokens are being transferred.\n     */\n    error ERC1155InvalidReceiver(address receiver);\n\n    /**\n     * @dev Indicates a failure with the `operator`’s approval. Used in transfers.\n     * @param operator Address that may be allowed to operate on tokens without being their owner.\n     * @param owner Address of the current owner of a token.\n     */\n    error ERC1155MissingApprovalForAll(address operator, address owner);\n\n    /**\n     * @dev Indicates a failure with the `approver` of a token to be approved. Used in approvals.\n     * @param approver Address initiating an approval operation.\n     */\n    error ERC1155InvalidApprover(address approver);\n\n    /**\n     * @dev Indicates a failure with the `operator` to be approved. Used in approvals.\n     * @param operator Address that may be allowed to operate on tokens without being their owner.\n     */\n    error ERC1155InvalidOperator(address operator);\n\n    /**\n     * @dev Indicates an array length mismatch between ids and values in a safeBatchTransferFrom operation.\n     * Used in batch transfers.\n     * @param idsLength Length of the array of token identifiers\n     * @param valuesLength Length of the array of token amounts\n     */\n    error ERC1155InvalidArrayLength(uint256 idsLength, uint256 valuesLength);\n}\n"},"Ownable.sol":{"content":"// SPDX-License-Identifier: MIT\n// OpenZeppelin Contracts (last updated v5.0.0) (access/Ownable.sol)\n\npragma solidity ^0.8.20;\n\nimport {Context} from \"./Context.sol\";\n\n/**\n * @dev Contract module which provides a basic access control mechanism, where\n * there is an account (an owner) that can be granted exclusive access to\n * specific functions.\n *\n * The initial owner is set to the address provided by the deployer. This can\n * later be changed with {transferOwnership}.\n *\n * This module is used through inheritance. It will make available the modifier\n * `onlyOwner`, which can be applied to your functions to restrict their use to\n * the owner.\n */\nabstract contract Ownable is Context {\n    address private _owner;\n\n    /**\n     * @dev The caller account is not authorized to perform an operation.\n     */\n    error OwnableUnauthorizedAccount(address account);\n\n    /**\n     * @dev The owner is not a valid owner account. (eg. `address(0)`)\n     */\n    error OwnableInvalidOwner(address owner);\n\n    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);\n\n    /**\n     * @dev Initializes the contract setting the address provided by the deployer as the initial owner.\n     */\n    constructor(address initialOwner) {\n        if (initialOwner == address(0)) {\n            revert OwnableInvalidOwner(address(0));\n        }\n        _transferOwnership(initialOwner);\n    }\n\n    /**\n     * @dev Throws if called by any account other than the owner.\n     */\n    modifier onlyOwner() {\n        _checkOwner();\n        _;\n    }\n\n    /**\n     * @dev Returns the address of the current owner.\n     */\n    function owner() public view virtual returns (address) {\n        return _owner;\n    }\n\n    /**\n     * @dev Throws if the sender is not the owner.\n     */\n    function _checkOwner() internal view virtual {\n        if (owner() != _msgSender()) {\n            revert OwnableUnauthorizedAccount(_msgSender());\n        }\n    }\n\n    /**\n     * @dev Leaves the contract without owner. It will not be possible to call\n     * `onlyOwner` functions. Can only be called by the current owner.\n     *\n     * NOTE: Renouncing ownership will leave the contract without an owner,\n     * thereby disabling any functionality that is only available to the owner.\n     */\n    function renounceOwnership() public virtual onlyOwner {\n        _transferOwnership(address(0));\n    }\n\n    /**\n     * @dev Transfers ownership of the contract to a new account (`newOwner`).\n     * Can only be called by the current owner.\n     */\n    function transferOwnership(address newOwner) public virtual onlyOwner {\n        if (newOwner == address(0)) {\n            revert OwnableInvalidOwner(address(0));\n        }\n        _transferOwnership(newOwner);\n    }\n\n    /**\n     * @dev Transfers ownership of the contract to a new account (`newOwner`).\n     * Internal function without access restriction.\n     */\n    function _transferOwnership(address newOwner) internal virtual {\n        address oldOwner = _owner;\n        _owner = newOwner;\n        emit OwnershipTransferred(oldOwner, newOwner);\n    }\n}\n"}}}
[3/4 00:16] Davi Calixto: "metadata": {
    "export_date": "2026-04-03T00:43:03.808Z",
    "type": "wallets",
    "user_email": "davilibanio3@gmail.com"
  },
  "wallets": [
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": null,
      "address": "0xe8c060f8052e07423f71d445277c61ac5138a2e5",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 56,
      "balance_usdc": 0,
      "network": "bsc",
      "balance": 1.2,
      "balance_matic": 49203.28547945205,
      "balance_bnb": 0,
      "balance_usdt": 0,
      "wallet_type": "evm",
      "balance_eth": 299187.42524629505,
      "private_key_encrypted": null,
      "id": "6930e3ddb23c5b3603e2889c",
      "created_date": "2025-12-04T01:29:01.108000",
      "updated_date": "2026-04-02T17:54:00.651000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    },
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": null,
      "address": "0xD48915f5ba4D5a9A3013f9953bfab9C3354b4D59",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 137,
      "balance_usdc": 0,
      "network": "polygon",
      "balance": 0,
      "balance_matic": 100,
      "balance_bnb": 0,
      "balance_usdt": 50000,
      "wallet_type": "evm",
      "balance_eth": 0.5,
      "private_key_encrypted": null,
      "id": "692fc226ce9fd7c889a4bbd4",
      "created_date": "2025-12-03T04:52:54.342000",
      "updated_date": "2026-04-01T21:41:04.614000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    },
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": "CALX-SL6MLV0623M",
      "address": "0x091b2478695d95ffbd674f9b9d5395c85197d9c6",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 1,
      "balance_usdc": 98810,
      "network": "ethereum",
      "balance": 1758801.1031506825,
      "balance_matic": 4482.042202739725,
      "balance_bnb": 61.63996666666668,
      "balance_usdt": 6631.5,
      "wallet_type": "evm",
      "balance_eth": 5.47280265029034,
      "private_key_encrypted": "0xffadc04d3dbde6f98130d0dfc31de1a6f992c72ac2801bd9a7ce0b5d48f97985",
      "id": "69040655777e4763947a9d45",
      "created_date": "2025-10-31T00:44:05.089000",
      "updated_date": "2026-03-22T02:00:00.676000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    }
  ],
  "balances": {
    "total_calx": 1758802.3031506825,
    "total_eth": 299193.39804894535,
    "total_btc": 0,
    "total_bnb": 61.63996666666668,
    "total_matic": 53785.32768219178,
    "total_usdt": 56631.5,
    "total_usdc": 98810,
    "total_dai":

  "metadata": {
    "export_date": "2026-04-03T00:43:03.808Z",
    "type": "wallets",
    "user_email": "davilibanio3@gmail.com"
  },
  "wallets": [
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": null,
      "address": "0xe8c060f8052e07423f71d445277c61ac5138a2e5",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 56,
      "balance_usdc": 0,
      "network": "bsc",
      "balance": 1.2,
      "balance_matic": 49203.28547945205,
      "balance_bnb": 0,
      "balance_usdt": 0,
      "wallet_type": "evm",
      "balance_eth": 299187.42524629505,
      "private_key_encrypted": null,
      "id": "6930e3ddb23c5b3603e2889c",
      "created_date": "2025-12-04T01:29:01.108000",
      "updated_date": "2026-04-02T17:54:00.651000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    },
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": null,
      "address": "0xD48915f5ba4D5a9A3013f9953bfab9C3354b4D59",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 137,
      "balance_usdc": 0,
      "network": "polygon",
      "balance": 0,
      "balance_matic": 100,
      "balance_bnb": 0,
      "balance_usdt": 50000,
      "wallet_type": "evm",
      "balance_eth": 0.5,
      "private_key_encrypted": null,
      "id": "692fc226ce9fd7c889a4bbd4",
      "created_date": "2025-12-03T04:52:54.342000",
      "updated_date": "2026-04-01T21:41:04.614000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    },
    {
      "owner_email": "davilibanio3@gmail.com",
      "public_key": "CALX-SL6MLV0623M",
      "address": "0x091b2478695d95ffbd674f9b9d5395c85197d9c6",
      "balance_dai": 0,
      "is_active": true,
      "balance_btc": 0,
      "chain_id": 1,
      "balance_usdc": 98810,
      "network": "ethereum",
      "balance": 1758801.1031506825,
      "balance_matic": 4482.042202739725,
      "balance_bnb": 61.63996666666668,
      "balance_usdt": 6631.5,
      "wallet_type": "evm",
      "balance_eth": 5.47280265029034,
      "private_key_encrypted": "0xffadc04d3dbde6f98130d0dfc31de1a6f992c72ac2801bd9a7ce0b5d48f97985",
      "id": "69040655777e4763947a9d45",
      "created_date": "2025-10-31T00:44:05.089000",
      "updated_date": "2026-03-22T02:00:00.676000",
      "created_by_id": "68f98a47fb826d55ff6696e3",
      "created_by": "davilibanio3@gmail.com",
      "is_sample": false
    }
  ],
  "balances": {
    "total_calx": 1758802.3031506825,
    "total_eth": 299193.39804894535,
    "total_btc": 0,
    "total_bnb": 61.63996666666668,
    "total_matic": 53785.32768219178,
    "total_usdt": 56631.5,
    "total_usdc": 98810,
    "total_dai":
[3/4 00:16] Davi Calixto: 0xF4e3d87A85137C6E72672b14AeB0BBDfc391b6E0
[3/4 22:15] Davi Calixto: Mintable.sol";
import {SafeERC20Remastered} from "./SafeERC20Remastered.sol";

import {Initializable} from "./Initializable.sol";
import "./IUniswapV2Factory.sol";
import "./IUniswapV2Pair.sol";
import "./IUniswapV2Router01.sol";
import "./IUniswapV2Router02.sol";

O contrato Calixtosuper é ERC20, ERC20Burnable, Ownable2Step, Mintable, Initializable {
    
    Utilizando o SafeERC20Remastered para IERC20;
 
    uint16 público swapThresholdRatio;
    
    uint256 privado _calixtoPending;
    uint256 privado _liquidezPendente;

    endereço público calixtoAddress;
    uint16[3] público calixtoFees;

    uint16[3] taxas de liquidez públicas;

    mapeamento (endereço => bool) público isExcludedFromFees;

    uint16[3] público totalFees;
    bool private _swapping;

    IUniswapV2Router02 roteador público V2;
    endereço par públicoV2;
    mapeamento (endereço => bool) AMMs públicos;
 
    erro InvalidAmountToRecover(uint256 amount, uint256 maxAmount);

    erro InvalidToken(endereço tokenAddress);

    erro Não é possível depositar moedas nativas (endereço da conta);

    erro InvalidSwapThresholdRatio(uint16 swapThresholdRatio);

    erro InvalidTaxRecipientAddress(conta de endereço);

    erro CannotExceedMaxTotalFee(uint16 buyFee, uint16 sellFee, uint16 transferFee);

    erro AMM inválido (endereço AMM);
 
    evento SwapThresholdUpdated(uint16 swapThresholdRatio);

    evento _troca = falso;
            }

        }

        super._update(de, para, quantidade);
        
        _afterTokenUpdate(de, para, quantidade);
        
    }

    função _beforeTokenUpdate(endereço de, endereço de destino, uint256 quantidade)
        interno
        visualizar
    {
    }

    função _afterTokenUpdate(endereço de, endereço de destino, uint256 quantidade)
        interno
    {
        se (de == endereço(0)) {
        }

    }
}
"},"ERC20.sol":{"content":"// SPDX-License-Identifier: MIT
// Contratos OpenZeppelin (última atualização v5.0.0) (token/ERC20/ERC20.sol)

pragma solidity ^0.8.20;

import {IERC20} from "./IERC20.sol";
import {IERC20Metadata} from "./IERC20Metadata.sol";
import {Context} from "./Context.sol";
import {IERC20Errors} from "./draft-IERC6093.sol";

/**
 * @dev Implementação da interface {IERC20}.
 *
 * Esta implementação é agnóstica à forma como os tokens são criados. Isso significa
 * que um mecanismo de fornecimento precisa ser adicionado em um contrato derivado usando {_mint}.
 *
 * DICA: Para obter informações detalhadas, consulte nosso guia.
 * https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226[Como
 * para implementar mecanismos de fornecimento].
 *
 * O valor padrão de {decimals} é 18. Para alterá-lo, você deve sobrescrever.
 * esta função para que retorne um valor diferente.
 *
 * Seguimos as diretrizes gerais de contratos do OpenZeppelin: funções revertem
 * em vez de retornar `false` em caso de falha. Esse comportamento é, no entanto,
 * convencional e não entra em conflito com as expectativas do ERC20
 * aplicações.
 *
 * Além disso, um evento {Approval} é emitido nas chamadas para {transferFrom}.
 * Isso permite que os aplicativos reconstruam a provisão para todas as contas.
 * ao ouvir esses eventos. Outras implementações do EIP podem não emitir
 * esses eventos, pois não são exigidos pela especificação.
 */
O contrato abstrato ERC20 é Contexto, IERC20, Metadados IERC20, Erros IERC20 {
    mapeamento(conta de endereço => uint256) saldos privados;

    mapeamento(endereço conta => mapeamento(endereço gastador => uint256)) permissões privadas;

    uint256 privado _totalSupply;

    string nome_privado;
    string private _symbol;

    /**
     * @dev Define os valores para {name} e {symbol}.
     *
     * Esses dois valores são imutáveis: eles só podem ser definidos uma vez durante
     * construção.
     */
    construtor(string nome_da_memória, string símbolo_da_memória) {
        _nome = nome_;
        _símbolo = símbolo_;
    }

    /**
     * @dev Retorna o nome do token.
     */
    function name() public view virtual returns (string memory) {
        retornar _nome;
    }

    /**
     * @dev Retorna o símbolo do token, geralmente uma versão abreviada do
     * nome.
     */
    function symbol() public view virtual returns (string memory) {
        retornar _símbolo;
    }

    /**
     * @dev Retorna o número de casas decimais usadas para obter sua representação para o usuário.
     * Por exemplo, se `decimals` for igual a `2`, um saldo de `505` tokens deverá
     * ser exibido ao usuário como `5,05` (`505 / 10 ** 2`).
     *
     * Os tokens geralmente optam por um valor de 18, imitando a relação entre
     * Éter e Wei. Este é o valor padrão retornado por esta função, a menos que
     * Foi anulado.
     *
     * NOTA: Esta informação é utilizada apenas para fins de _exibição_: ela em
     * não afeta de forma alguma os cálculos do contrato, incluindo
     * {IERC20-balanceOf} e {IERC20-transfer}.
     */
    função decimals() pública view virtual retorna (uint8) {
        retornar 18;
    }

    /**
     * @dev Veja {IERC20-totalSupply}.
     */
    função totalSupply() pública view virtual retorna (uint256) {
        retornar _totalSupply;
    }

    /**
     * @dev Veja {IERC20-balanceOf}.
     */
    função balanceOf(address account) public view virtual returns (uint256) {
        retornar _balances[conta];
    }

    /**
     * @dev Veja {IERC20-transfer}.
     *
     * Requisitos:
     *
     * - `to` não pode ser o endereço zero.
     * - o chamador deve ter um saldo de pelo menos `valor`.
     */
    função transfer(endereço de destino, valor uint256) pública virtual retorna (bool) {
        proprietário do endereço = _msgSender();
        _transferir(proprietário, para, valor);
        retornar verdadeiro;
    }

    /**
     * @dev Veja {IERC20-allowance}.
     */
    função allowance(proprietário do endereço, gastador do endereço) visualização pública retornos virtuais (uint256) {
        retornar _allowances[proprietário][gastador];
    }

    /**
     * @dev Veja {IERC20-approve}.
     *
     * NOTA: Se `value` for o valor máximo de `uint256`, a permissão não será atualizada em
     * `transferFrom`. Isso é semanticamente equivalente a uma aprovação infinita.
     *
     * Requisitos:
     *
     * - `spender` não pode ser o endereço zero.
     */
    função approve(endereço spender, uint256 valor) pública virtual retorna (bool) {
        proprietário do endereço = _msgSender();
        _aprovar(proprietário, gastador, valor);
        retornar verdadeiro;
    }

    /**
     * @dev Veja {IERC20-transferFrom}.
     *
     * Emite um evento {Approval} indicando a permissão atualizada. Isso não é
     * exigido pelo EIP. Consulte a nota no início do {ERC20
  <img src="./apps/remix-ide/src/assets/img/icon.png" alt="Remix Logo" width="200"/>
</p>
<h3 align="center">Remix Project</h3>
    
<div align="center">


[![CircleCI](https://img.shields.io/circleci/build/github/remix-project-org/remix-project?logo=circleci)](https://circleci.com/gh/remix-project-org/remix-project)
[![Documentation Status](https://readthedocs.org/projects/remix-ide/badge/?version=latest)](https://remix-ide.readthedocs.io/en/latest/index.html)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat&logo=github)](https://github.com/remix-project-org/remix-project/blob/master/CONTRIBUTING.md)
[![GitHub contributors](https://img.shields.io/github/contributors/remix-project-org/remix-project?style=flat&logo=github)](https://github.com/remix-project-org/remix-project/graphs/contributors)
[![Awesome Remix](https://img.shields.io/badge/Awesome--Remix-resources-green?logo=awesomelists)](https://github.com/remix-project-org/awesome-remix)
[![GitHub](https://img.shields.io/github/license/remix-project-org/remix-project)](https://github.com/remix-project-org/remix-project/blob/master/LICENSE)
[![Discord](https://img.shields.io/badge/join-discord-brightgreen.svg?style=flat&logo=discord)](https://discord.gg/MzhfCGstNA)
[![X Follow](https://img.shields.io/twitter/follow/ethereumremix?style=flat&logo=x&color=green)](https://x.com/ethereumremix)

</div>

## Remix Project

**Remix Project** is a rich toolset including Remix IDE, a comprehensive smart contract development tool. The Remix Project also includes Remix Plugin Engine and Remix Libraries which are low-level tools for wider use.  

## Remix IDE
**Remix IDE** is used for the entire journey of contract development by users of any knowledge level. It fosters a fast development cycle and has a rich set of plugins with intuitive GUIs. The IDE comes in 2 flavors and a VSCode extension:

**Remix Online IDE**, see: [https://remix.ethereum.org](https://remix.ethereum.org)

:point_right: Supported browsers: Firefox v100.0.1 & Chrome v101.0.4951.64. No support for Remix's use on tablets or smartphones or telephones.

**Remix Desktop IDE**, see releases: [https://github.com/remix-project-org/remix-desktop/releases](https://github.com/remix-project-org/remix-desktop/releases)

![Remix screenshot](https://github.com/remix-project-org/remix-project/raw/master/apps/remix-ide/remix-screenshot-400h.png)


## Remix libraries 
Remix libraries are essential for Remix IDE's native plugins. Read more about libraries [here](libs/README.md)

## Offline Usage

The `gh-pages` branch of [remix-live](https://github.com/remix-project-org/remix-live) always has the latest stable build of Remix. It contains a ZIP file with the entire build. Download it to use offline.

Note: It contains the latest supported version of Solidity available at the time of the packaging. Other compiler versions can be used online only.


## Setup

* Install **Yarn** and **Node.js**. See [Guide for NodeJs](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) and [Yarn install](https://classic.yarnpkg.com/lang/en/docs/install)<br/>
*Supported versions:*
```bash
"engines": {
    "node": "^20.0.0",
    "npm": "^6.14.15"
  }
```
* Install [Nx CLI](https://nx.dev/using-nx/nx-cli) globally to enable running **nx executable commands**.
```bash
yarn global add nx
```
* Clone the GitHub repository (`wget` need to be installed first):

```bash
git clone https://github.com/remix-project-org/remix-project.git
```
* Build and Run `remix-project`:

1. Move to project directory: `cd remix-project`
2. Install dependencies: `yarn install` or simply run `yarn`
3. Build Remix libraries: `yarn run build:libs`
4. Build Remix project: `yarn build`
5. Build and run project server: `yarn serve`. Optionally, run `yarn serve:hot` to enable hot module to reload for frontend updates.

Open `http://127.0.0.1:8080` in your browser to load Remix IDE locally.

Go to your `text editor` and start developing. The browser will automatically refresh when files are saved.

## Production Build
To generate react production builds for remix-project.
```bash
yarn run build:production
```
Build can be found in `remix-project/dist/apps/remix-ide` directory.

```bash
yarn run serve:production
```
Production build will be served by default to `http://localhost:8080/` or `http://127.0.0.1:8080/`

## Nx Cloud caching

This repo uses Nx Cloud to speed up builds and keep CI deterministic via remote caching.

- Configuration: `nx.json` uses the Nx Cloud runner and reads the token from the `NX_CLOUD_ACCESS_TOKEN` environment variable.
- CI: CircleCI jobs automatically use `--cloud` when the token is present; for forked PRs (no secrets), they fall back to local-only caching. Build logs are stored under `logs/nx-build.log`.
- Verifying locally: run the same target twice; the second run should print “Nx read the output from the cache”. Example: `nx run remix-ide:build` and run it again.
- Insights: View cache analytics and run details at https://nx.app (links appear in Nx output when the token is configured).

## Docker:

Prerequisites: 
* Docker (https://docs.docker.com/desktop/)
* Docker Compose (https://docs.docker.com/compose/install/)

### Run with docker

If you want to run the latest changes that are merged into the master branch then run:

```
docker pull remixproject/remix-ide:latest
docker run -p 8080:80 remixproject/remix-ide:latest
```

If you want to run the latest remix-live release run.
```
docker pull remixproject/remix-ide:remix_live
docker run -p 8080:80 remixproject/remix-ide:remix_live
```

### Run with docker-compose:

To run locally without building you only need docker-compose.yaml file and you can run:

```
docker-compose pull
docker-compose up -d
```

Then go to http://localhost:8080 and you can use your Remix instance.

To fetch the docker-compose file without cloning this repo run:
```
curl https://raw.githubusercontent.com/remix-project-org/remix-project/master/docker-compose.yaml > docker-compose.yaml
```

### Troubleshooting

If you have trouble building the project, make sure that you have the correct version of `node`, `npm` and `nvm`. Also, ensure [Nx CLI](https://nx.dev/using-nx/nx-cli) is installed globally.

Run:

```bash
node --version
npm --version
nvm --version
```

In Debian-based OS such as Ubuntu 14.04LTS, you may need to run `apt-get install build-essential`. After installing `build-essential`, run `npm rebuild`.

## Unit Testing

Run the unit tests using library name like: `nx test <project-name>`

For example, to run unit tests of `remix-analyzer`, use `nx test remix-analyzer`

## Browser Testing

To run the tests via Nightwatch:

 - Install webdrivers for the first time: `yarn install_webdriver`
 - Build & Serve Remix: `yarn serve`

        
**NOTE:**

- **The `ballot` tests suite** requires running `ganache` locally.

- **The `remixd` tests suite** requires running `remixd` locally.

- **The `gist` tests suite** requires specifying a GitHub access token in **.env file**. 
```
    gist_token = <token> // token should have permission to create a gist
```

There is a script to allow selecting the browser and a specific test to run:

```
yarn run select_test
```

You need to have 

- selenium running 

- the IDE running

- optionally have remixd or ganache running

### Splitting tests with groups

Groups can be used to group tests in a test file together. The advantage is you can avoid running long test files when you want to focus on a specific set of tests within a test file.

These groups only apply to the test file, not across all test files. So for example group1 in the ballot is not related to a group1 in another test file.

Running a group only runs the tests marked as belonging to the group + all the tests in the test file that do not have a group tag. This way you can have tests that run for all groups, for example, to perform common actions.

There is no need to number the groups in a certain order. The number of the group is arbitrary.

A test can have multiple group tags, this means that this test will run in different groups.

You should write your tests so they can be executed in groups and not depend on other groups.

To do this you need to:

- Add a group to tag to a test, they are formatted as #group followed by a number: so it becomes #group1, #group220, #group4. Any number will do. You don't have to do it in a specific order. 

```
  'Should generate test file #group1': function (browser: NightwatchBrowser) {
    browser.waitForElementPresent('*[data-id="verticalIconsKindfilePanel"]')
```

- add '@disabled': true to the test file you want to split:

```
module.exports = {
  '@disabled': true,
  before: function (browser: NightwatchBrowser, done: VoidFunction) {
    init(browser, done) // , 'http://localhost:8080', false)
  },
```
- change package JSON to locally run all group tests (point to appropriate config file depending on environment):

```
    "nightwatch_local_debugger": "yarn run build:e2e && nightwatch --config dist/apps/remix-ide-e2e/nightwatch-chrome.js dist/apps/remix-ide-e2e/src/tests/debugger_*.spec.js --env=chrome",
```

- run the build script to build the test files if you want to run the locally

```
yarn run build:e2e
```

### Locally testing group tests

You can tag any test with a group name, for example, #group10 and easily run the test locally.

- make sure you have nx installed globally
- group tests are run like any other test, just specify the correct group number

#### method 1

This script will give you an options menu, just select the test you want
```
yarn run select_test
```

### Run the same (flaky) test across all instances in CircleCI

In CircleCI all tests are divided across instances to run in parallel. 
You can also run 1 or more tests simultaneously across all instances.
This way the pipeline can easily be restarted to check if a test is flaky.

For example:

```
  'Static Analysis run with remixd #group3 #flaky': function (browser) {
```

Now, the group3 of this test will be executed in firefox and chrome 80 times.
If you mark more groups in other tests they will also be executed. 

**CONFIGURATION**

It's important to set a parameter in the .circleci/config.yml, set it to false then the normal tests will run.
Set it to true to run only tests marked with flaky.
```
parameters:
  run_flaky_tests:
    type: boolean
    default: true
```

## Important Links

- Official website: https://remix.live
- Official documentation: https://remix-ide.readthedocs.io/en/latest/
- Curated list of Remix resources: https://github.com/remix-project-org/awesome-remix
- Substack: https://ethereumremix.substack.com
- Linkedin: https://www.linkedin.com/company/ethereum-remix
- X: https://x.com/ethereumremix
- Join our Discord: https://discord.gg/MzhfCGstNA
