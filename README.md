- 👋 Hi, I’m @Elion34
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
Elion34/Elion34 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Stumble guys hack hack name

// SPDX-License-Identifier: MIT OR Apache-2.0
pragma solidity ^0.8.3;

import "@openzeppelin/contracts/utils/Counters.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

import "hardhat/console.sol";

contract NFTMarket is ReentrancyGuard {
  using Counters for Counters.Counter;
  Counters.Counter private _itemIds;
  Counters.Counter private _itemsSold;

  address payable owner;
  uint256 listingPrice = 0.025 ether;

  constructor() {
    owner = payable(msg.sender);
  }

  struct MarketItem {
    uint itemId;
    address nftContract;
    uint256 tokenId;
    address payable seller;
    address payable owner;
    uint256 price;
    bool sold;
  }

  mapping(uint256 => MarketItem) private idToMarketItem;

  event MarketItemCreated (
    uint indexed itemId,
    address indexed nftContract,
    uint256 indexed tokenId,
    address seller,
    address owner,
    uint256 price,
    bool sold
  );

  /* Returns the listing price of the contract */
  function getListingPrice() public view returns (uint256) {
    return listingPrice;
  }
  
  /* Places an item for sale on the marketplace */
  function createMarketItem(
    address nftContract,
    uint256 tokenId,
    uint256 price
  ) public payable nonReentrant {
    require(price > 0, "Price must be at least 1 wei");
    require(msg.value == listingPrice, "Price must be equal to listing price");

    _itemIds.increment();
    uint256 itemId = _itemIds.current();
  
    idToMarketItem[itemId] =  MarketItem(
      itemId,
      nftContract,
      tokenId,
      payable(msg.sender),
      payable(address(0)),
      price,
      false
    );

    IERC721(nftContract).transferFrom(msg.sender, address(this), tokenId);

    emit MarketItemCreated(
      itemId,
      nftContract,
      tokenId,
      msg.sender,
      address(0),
      price,
      false
    );
  }

  /* Creates the sale of a marketplace item */
  /* Transfers ownership of the item, as well as funds between parties */
  function createMarketSale(
    address nftContract,
    uint256 itemId
    ) public payable nonReentrant {
    uint price = idToMarketItem[itemId].price;
    uint tokenId = idToMarketItem[itemId].tokenId;
    require(msg.value == price, "Please submit the asking price in order to complete the purchase");

    idToMarketItem[itemId].seller.transfer(msg.value);
    IERC721(nftContract).transferFrom(address(this), msg.sender, tokenId);
    idToMarketItem[itemId].owner = payable(msg.sender);
    idToMarketItem[itemId].sold = true;
    _itemsSold.increment();
    payable(owner).transfer(listingPrice);
  }

  /* Returns all unsold market items */
  function fetchMarketItems() public view returns (MarketItem[] memory) {
    uint itemCount = _itemIds.current();
    uint unsoldItemCount = _itemIds.current() - _itemsSold.current();
    uint currentIndex = 0;

    MarketItem[] memory items = new MarketItem[](unsoldItemCount);
    for (uint i = 0; i < itemCount; i++) {
      if (idToMarketItem[i + 1].owner == address(0)) {
        uint currentId = i + 1;
        MarketItem storage currentItem = idToMarketItem[currentId];
        items[currentIndex] = currentItem;
        currentIndex += 1;
      }
    }
    return items;
  }

  /* Returns onlyl items that a user has purchased */
  function fetchMyNFTs() public view returns (MarketItem[] memory) {
    uint totalItemCount = _itemIds.current();
    uint itemCount = 0;
    uint currentIndex = 0;

    for (uint i = 0; i < totalItemCount; i++) {
      if (idToMarketItem[i + 1].owner == msg.sender) {
        itemCount += 1;
      }
    }

    MarketItem[] memory items = new MarketItem[](itemCount);
    for (uint i = 0; i < totalItemCount; i++) {
      if (idToMarketItem[i + 1].owner == msg.sender) {
        uint currentId = i + 1;
        MarketItem storage currentItem = idToMarketItem[currentId];
        items[currentIndex] = currentItem;
        currentIndex += 1;
      }
    }
    return items;
  }

  /* Returns only items a user has created */
  function fetchItemsCreated() public view returns (MarketItem[] memory) {
    uint totalItemCount = _itemIds.current();
    uint itemCount = 0;
    uint currentIndex = 0;

    for (uint i = 0; i < totalItemCount; i++) {
      if (idToMarketItem[i + 1].seller == msg.sender) {
        itemCount += 1;
      }
    }

    MarketItem[] memory items = new MarketItem[](itemCount);
    for (uint i = 0; i < totalItemCount; i++) {
      if (idToMarketItem[i + 1].seller == msg.sender) {
        uint currentId = i + 1;
        MarketItem storage currentItem = idToMarketItem[currentId];
        items[currentIndex] = currentItem;
        currentIndex += 1;
      }
    }
    return items;
  }
}
Skip to content

OpenZeppelin/openzeppelin-contractsPublic

CodeIssues116Pull requests30ActionsSecurity11Insights

 master 

openzeppelin-contracts/contracts/token/ERC20/ERC20.sol

tonynoce " class="Link--secondary" href="https://github.com/OpenZeppelin/openzeppelin-contracts/commit/5e7e9acfa4630c1f933f7e8435ccc4490cacb274" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-muted) !important; text-decoration: none;">Fix broken forum URL (#3537" class="Link--secondary" href="https://github.com/OpenZeppelin/openzeppelin-contracts/commit/5e7e9acfa4630c1f933f7e8435ccc4490cacb274" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-muted) !important; text-decoration: none;">)

…

 42 contributors+26

389 lines (351 sloc)  12.7 KB

// SPDX-License-Identifier: MIT// OpenZeppelin Contracts (last updated v4.7.0) (token/ERC20/ERC20.sol) pragma solidity ^0.8.0; import "./IERC20.sol";import "./extensions/IERC20Metadata.sol";import "../../utils/Context.sol"; /** * @dev Implementation of the {IERC20} interface. * * This implementation is agnostic to the way tokens are created. This means * that a supply mechanism has to be added in a derived contract using {_mint}. * For a generic mechanism see {ERC20PresetMinterPauser}. * * TIP: For a detailed writeup see our guide * https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226[How * to implement supply mechanisms]. * * We have followed general OpenZeppelin Contracts guidelines: functions revert * instead returning `false` on failure. This behavior is nonetheless * conventional and does not conflict with the expectations of ERC20 * applications. * * Additionally, an {Approval} event is emitted on calls to {transferFrom}. * This allows applications to reconstruct the allowance for all accounts just * by listening to said events. Other implementations of the EIP may not emit * these events, as it isn't required by the specification. * * Finally, the non-standard {decreaseAllowance} and {increaseAllowance} * functions have been added to mitigate the well-known issues around setting * allowances. See {IERC20-approve}. */contract ERC20 is Context, IERC20, IERC20Metadata { mapping(address => uint256) private _balances; mapping(address => mapping(address => uint256)) private _allowances; uint256 private _totalSupply; string private _name; string private _symbol; /** * @dev Sets the values for {name} and {symbol}. * * The default value of {decimals} is 18. To select a different value for * {decimals} you should overload it. * * All two of these values are immutable: they can only be set once during * construction. */ constructor(string memory name_, string memory symbol_) { _name = name_; _symbol = symbol_; } /** * @dev Returns the name of the token. */ function name() public view virtual override returns (string memory) { return _name; }

