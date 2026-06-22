<p align="center">// SPDX-License-Identifier: MIT
  pragma solidity ^0.8.25;

  /*
   Calixto Super (CALX) — BNB Smart Chain (chainId 56)
   Supply fixo: 10,000,000 CALX (18 decimais)
   Deploy por EOA habilitado (sem restrições)
  */

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
    function owner() public view virtual returns (address) { return _owner; }
    function renounceOwnership() public virtual onlyOwner { _transferOwnership(address(0)); }
    function transferOwnership(address newOwner) public virtual onlyOwner {
      require(newOwner != address(0), "Ownable: new owner is the zero address");
      _transferOwnership(newOwner);
    }
    function _transferOwnership(address newOwner) internal virtual {
      address oldOwner = _owner; _owner = newOwner; emit OwnershipTransferred(oldOwner, newOwner);
    }
  }

  contract ERC20 is Context, IERC20, IERC20Metadata {
    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;
    uint256 private _totalSupply;
    string private _name; string private _symbol;
    constructor(string memory name_, string memory symbol_) { _name = name_; _symbol = symbol_; }
    function name() public view virtual override returns (string memory) { return _name; }
    function symbol() public view virtual override returns (string memory) { return _symbol; }
    function decimals() public view virtual override returns (uint8) { return 18; }
    function totalSupply() public view virtual override returns (uint256) { return _totalSupply; }
    function balanceOf(address account) public view virtual override returns (uint256) { return _balances[account]; }
    function transfer(address to, uint256 amount) public virtual override returns (bool) {
      address owner = _msgSender(); _transfer(owner, to, amount); return true;
    }
    function allowance(address owner, address spender) public view virtual override returns (uint256) { return _allowances[owner][spender]; }
    function approve(address spender, uint256 amount) public virtual override returns (bool) { _approve(_msgSender(), spender, amount); return true; }
    function transferFrom(address from, address to, uint256 amount) public virtual override returns (bool) {
      _spendAllowance(from, _msgSender(), amount); _transfer(from, to, amount); return true;
    }
    function _transfer(address from, address to, uint256 amount) internal virtual {
      require(from != address(0), "ERC20: transfer from the zero address"); require(to != address(0), "ERC20: transfer to the zero address");
      uint256 fromBalance = _balances[from]; require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");
      unchecked { _balances[from] = fromBalance - amount; _balances[to] += amount; } emit Transfer(from, to, amount);
    }
    function _mint(address account, uint256 amount) internal virtual {
      require(account != address(0), "ERC20: mint to the zero address"); _totalSupply += amount; unchecked { _balances[account] += amount; } emit Transfer(address(0), account, amount);
    }
    function _burn(address account, uint256 amount) internal virtual {
      require(account != address(0), "ERC20: burn from the zero address"); uint256 bal = _balances[account]; require(bal >= amount, "ERC20: burn amount exceeds balance");
      unchecked { _balances[account] = bal - amount; _totalSupply -= amount; } emit Transfer(account, address(0), amount);
    }
    function _approve(address owner, address spender, uint256 amount) internal virtual {
      require(owner != address(0) && spender != address(0), "ERC20: zero address"); _allowances[owner][spender] = amount; emit Approval(owner, spender, amount);
    }
    function _spendAllowance(address owner, address spender, uint256 amount) internal virtual {
      uint256 current = allowance(owner, spender); if (current != type(uint256).max) { require(current >= amount, "ERC20: insufficient allowance"); unchecked { _approve(owner, spender, current - amount); } }
    }
  }

  // Extensão com CAP (supply máximo fixo)
  abstract contract ERC20Capped is ERC20 {
    uint256 private immutable _cap;
    constructor(uint256 cap_) { require(cap_ > 0, "ERC20Capped: cap is 0"); _cap = cap_; }
    function cap() public view returns (uint256) { return _cap; }
    function _mint(address account, uint256 amount) internal virtual override {
      require(totalSupply() + amount <= _cap, "ERC20Capped: cap exceeded"); super._mint(account, amount);
    }
  }

  /// Token CALX BSC — supply fixo 10,000,000
  contract CalixtoToken is ERC20Capped, Ownable {
    event TokensBurned(address indexed from, uint256 amount);
    uint256 private constant MAX_SUPPLY = 10_000_000 * 10**18;
    constructor(address initialRecipient)
      ERC20("Calixto Super", "CALX")
      Ownable(msg.sender)
      ERC20Capped(MAX_SUPPLY)
    {
      require(initialRecipient != address(0), "Invalid recipient address");
      // Mint 100% do supply fixo para a carteira do tesouro
      _mint(initialRecipient, MAX_SUPPLY);
    }
    // Burn voluntário
    function burn(uint256 amount) public { _burn(msg.sender, amount); emit TokensBurned(msg.sender, amount); }
    // Burn com allowance
    function burnFrom(address from, uint256 amount) public { _spendAllowance(from, msg.sender, amount); _burn(from, amount); emit TokensBurned(from, amount); }
    function tokenInfo() public pure returns (string memory, string memory, uint8, string memory) {
      return ("Calixto Super", "CALX", 18, "BNB Smart Chain");
    }
  }
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
