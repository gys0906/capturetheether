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
