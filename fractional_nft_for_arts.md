# System
You are a senior blockchain developer to create solidity smart contract given a specification.

- Describe general use case
- Describe interface and acceptance criteria
- Output a ethereum smart contract written in solidity

# NFT Smart Contract Spec
==== AI Generated Art===
Here is the spec of the smart contract for representing the AI Generated Art:

- The smart contract is a fractional NFT based off ERC721, referring to representing a Arts
- The NFT contains a list of referenced artists
- The NFT will accept a raw bytes of images, and call an external smart contract to return a list of referenced artist with estimated ownership percentages.
- When the NFT is minted, the NFT will charge the minter a certain amount of fees
- The fees will be distributed partially to the referenced artist based the percentages predefined.


==== Art referenced Artist===
Here is the spec of the smart contract for representing the referenced Artists

- The smart contract is a standard NFT based off ERC721, representing 1 unique artist per NFT
- This artist smart contract’s address is referenced by the Art smart contracted created before

=======================
Create these two smart contracts

## Smart Contracts
```
// Smart Contract for AI Generated Art

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

interface IReferenceArtists {
    function getReferenceArtists(bytes memory _image) external view returns (address[] memory, uint256[] memory);
}

contract AIGeneratedArt is ERC721 {

    uint256 public fee;
    address public referenceArtistsContract;

    struct Art {
        bytes image;
        address[] referencedArtists;
        uint256[] ownershipPercentages;
    }

    mapping (uint256 => Art) public arts;

    constructor(string memory _name, string memory _symbol, uint256 _fee, address _referenceArtistsContract) ERC721(_name, _symbol) {
        fee = _fee;
        referenceArtistsContract = _referenceArtistsContract;
    }

    function mint(bytes memory _image) external payable returns (uint256) {
        require(msg.value >= fee, "Insufficient fee");
        (address[] memory referencedArtists, uint256[] memory ownershipPercentages) = IReferenceArtists(referenceArtistsContract).getReferenceArtists(_image);
        require(referencedArtists.length > 0, "No referenced artists found");
        require(referencedArtists.length == ownershipPercentages.length, "Invalid ownership percentages");

        uint256 tokenId = totalSupply() + 1;
        _safeMint(msg.sender, tokenId);
        arts[tokenId] = Art(_image, referencedArtists, ownershipPercentages);

        uint256 distributedFees = 0;
        for (uint256 i = 0; i < referencedArtists.length; i++) {
            address artist = referencedArtists[i];
            uint256 percentage = ownershipPercentages[i];
            uint256 artistFee = (fee * percentage) / 100;
            payable(artist).transfer(artistFee);
            distributedFees += artistFee;
        }
        if (distributedFees < fee) {
            payable(owner()).transfer(fee - distributedFees);
        }

        return tokenId;
    }
}



// Smart Contract for Referenced Artists

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract ReferencedArtist is ERC721 {

    constructor(string memory _name, string memory _symbol) ERC721(_name, _symbol) {
    }
}
```
Please add a name property to the ReferencedArtist smart contract

Sure, here is the updated smart contract for ReferencedArtist with a name property:

```
// Smart Contract for Referenced Artists

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract ReferencedArtist is ERC721 {

    string public name;

    constructor(string memory _name, string memory _symbol) ERC721(_name, _symbol) {
        name = _name;
    }
}
```

## Unit Test
Create javascript unit tests for truffle for the smart contracts created above

Sure, here are the JavaScript unit tests for the AIGeneratedArt and ReferencedArtist smart contracts using Truffle:

```
// JavaScript unit tests for AIGeneratedArt smart contract using Truffle

const AIGeneratedArt = artifacts.require("AIGeneratedArt");
const ReferencedArtist = artifacts.require("ReferencedArtist");

contract("AIGeneratedArt", (accounts) => {
  let aiGeneratedArt;
  let referencedArtist;

  before(async () => {
    referencedArtist = await ReferencedArtist.new("Artist Name", "ART");
    aiGeneratedArt = await AIGeneratedArt.new("AI Generated Art", "AIGA", web3.utils.toWei("0.1"), referencedArtist.address);
  });

  it("should mint an AI Generated Art NFT", async () => {
    const image = "0x123456";
    const result = await aiGeneratedArt.mint(image, { from: accounts[0], value: web3.utils.toWei("0.1") });
    const tokenId = result.logs[0].args.tokenId;
    const art = await aiGeneratedArt.arts(tokenId);
    assert.equal(art.image, image);
    assert.equal(art.referencedArtists.length, 1);
    assert.equal(art.ownershipPercentages.length, 1);
  });

  it("should not mint an AI Generated Art NFT if fee is insufficient", async () => {
    const image = "0x123456";
    try {
      await aiGeneratedArt.mint(image, { from: accounts[0], value: web3.utils.toWei("0.05") });
      assert.fail("Expected minting to fail");
    } catch (error) {
      assert.include(error.message, "Insufficient fee");
    }
  });

  it("should not mint an AI Generated Art NFT if no referenced artists found", async () => {
    const image = "0xabcdef";
    try {
      await aiGeneratedArt.mint(image, { from: accounts[0], value: web3.utils.toWei("0.1") });
      assert.fail("Expected minting to fail");
    } catch (error) {
      assert.include(error.message, "No referenced artists found");
    }
  });

  it("should not mint an AI Generated Art NFT if invalid ownership percentages", async () => {
    const image = "0x123456";
    const referencedArtists = [accounts[1], accounts[2]];
    const ownershipPercentages = [50, 60];
    const referenceArtists = await ReferencedArtist.new("Artist Name", "ART");
    const aiGeneratedArt = await AIGeneratedArt.new("AI Generated Art", "AIGA", web3.utils.toWei("0.1"), referenceArtists.address);
    try {
      await aiGeneratedArt.mint(image, { from: accounts[0], value: web3.utils.toWei("0.1") });
      assert.fail("Expected minting to fail");
    } catch (error) {
      assert.include(error.message, "Invalid ownership percentages");
    }
  });
});


// JavaScript unit tests for ReferencedArtist smart contract using Truffle

const ReferencedArtist = artifacts.require("ReferencedArtist");

contract("ReferencedArtist", (accounts) => {
  let referencedArtist;

  before(async () => {
    referencedArtist = await ReferencedArtist.new("Artist Name", "ART");
  });

  it("should mint a Referenced Artist NFT", async () => {
    const result = await referencedArtist.mint(accounts[0]);
    const tokenId = result.logs[0].args.tokenId;
    const owner = await referencedArtist.ownerOf(tokenId);
    assert.equal(owner, accounts[0]);
  });
});
```

Note: The third test case in the AIGeneratedArt contract test is invalid as the ownership percentages array has two values that add up to more than 100%. The correct test case would be as follows:

```
it("should not mint an AI Generated Art NFT if invalid ownership percentages", async () => {
    const image = "0x123456";
    const referencedArtists = [accounts[1], accounts[2]];
    const ownershipPercentages = [50, 60];
    const referenceArtists = await ReferencedArtist.new("Artist Name", "ART");
    const aiGeneratedArt = await AIGeneratedArt.new("AI Generated Art", "AIGA", web3.utils.toWei("0.1"), referenceArtists.address);
    try {
      await aiGeneratedArt.mint(image, { from: accounts[0], value: web3.utils.toWei("0.1") });
      assert.fail("Expected minting to fail");
    } catch (error) {
      assert.include(error.message, "Invalid ownership percentages");
    }
  });
```