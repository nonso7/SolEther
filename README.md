# Ethereum Dishes

Ethereum Dishes is a reference list of points about various topics related to Blockchain, EVM, and solidity. It is intended to be a helpful resource for those who are interested in learning about these topics.



### Random knowledge about Ethereum and Solidity

## Notes on Smart Contract Security
1. The term "dark forest" originated from a scifi book by Lu Cixin, where identifying someone's location means certain death in the hands of aliens.
2. Dan Robinson and Georgios Konstantopoulos brought the term to the crypto space to describe the adversarial environment of the Ethereum mempool, where MEV bots watch transactions and see if there are any value that can be extracted by front-running the transaction or any inter-contract calls resulting from the transaction.
3. A smart contract generally has 4 properties: 
    1. Code
    2. Storage
    3. Balance
    4. Nonce

Normally, the code can't be updated, so smart contract security is about preventing unexpected changes to the last three properties, in particular the storage and balance. This includes the "selfdestruct" function and the contract getting stuck and unusable.
   
4. Since smart contracts are also tied to other parts of a whole application, an attacker can decide to attack other parts of the system like the website, etc. and not necessarily attack the smart contract.
5. Some contracts are used together with other contracts, so they need to be analyzed separately and as a whole too.
6. Smart contract security does not only rely on the security of the smart contracts but other factors connected to it, especially the users.
7. When doing a threat modeling, there's a need to identify all of the attack surface of the smart contracts and not just the contracts.
8. There's a difference between smart contract security and Solidity security, with the first being security defined by the properties of the EVM and the latter being security implications of writing smart contracts with Solidity.
9. When talking about smart contract security, it encompasses the following: Ethereum client software, EVM, ABI, and Solidity.

## Notes on Gas
### Gas And Gas Savings Techniques

10. The gas cost of a transaction in Ethereum can be affected by several factors:
     1. The size of the transaction data. Every non-zero byte of data costs 16 gas.
     2. The size and amount of memory covered. Each memory expansion costs 3 gas. Memory is divided into slots ranging from 0x00-0x20 (32 bytes), 0x20-0x40 (another 32 bytes), and so on. If you store a value in memory at offset 0x40 (which goes from 0x40 to 0x60), which is the third slot in memory, you will pay 3 gas * 2 slots (6 gas) for the two memory expansions because you have skipped the first two slots (0x00-0x20 and 0x20-0x40).
     3. The gas cost for the opcodes used in the transaction.
     4. The amount of storage used.

11. Payable functions are cheaper than non-payable functions because the EVM uses the ISZERO opcode to check if the CALLVALUE opcode is zero or not during a function in a non-payable function and reverts if value was sent along with it. This adds extra opcodes which are not present in a payable function and thus increases the gas cost.

12. The same principle applies to the unchecked keyword in Solidity, but in the opposite manner. The unchecked keyword removes extra opcodes that the EVM uses to check if the resulting value of an arithmetic plus/minus operation is greater than (for plus) or less than (for minus) any of the factors of the operation.

13. Every transaction (except contract deployment) in Ethereum costs at least 21,000 gas (plus any other execution cost, such as the gas cost of running opcodes if it is a smart contract interaction). This value is imposed by the protocol by default. This is because when a transaction is carried out, it goes through an initial test of intrinsic validity and the following actions are carried out:
     1. The transaction is well-formed RLP (recursive length prefix) with no additional trailing bytes.

     2. The transaction signature is valid.

     3. The transaction nonce is valid (equivalent to the sender account's current nonce, to prevent signature replay).

     4. The sender account has no contract code deployed (see EIP-3607).

     5. The gas limit is no smaller than the intrinsic gas used by the transaction.

     6. The sender account balance contains at least the cost required in up-front payment for the transaction.
     (All of these can be found on page 8 of the Yellowpaper.)

14. You can access the block base fee from Solidity ^0.8.7 using the block.basefee global variable.

### EIP-1559 And Gas Fee Types
15. The transaction fee is calculated by:
     1. max gas fee or gas price * gas used / 1 gwei (1 followed by 9 zeros, which is equal to 1 billion in number)

16. According to EIP-1559, the gas fees go up and down depending on the network condition of the protocol. There are several gas terms you should consider because of EIP-1559:

    1. Max fee: The maximum amount of fee you are willing to pay for each unit of gas for a transaction.

    2. Max priority fee: The fee set aside from your max fee that is used as a tip for the miner of your transaction.

    3. Base fee: The minimum fee (all in gwei) required to carry out a transaction in a block. It is set at the protocol level and not by the transaction initiator. The algorithm for gas increases the base gas fee of a block by 12% if the 3 million block gas limit is reached, and decreases it by 12% if the block gas limit was not reached.

    4. Priority fee: In the case where there is a leftover gas fee after executing a transaction and taking the base fee (the leftover fee is the max fee minus the base fee, in the case where the base fee is less than the max fee and not equal to it), the max priority fee is taken from this leftover fee and sent to the miner as a tip. If the leftover fee is less than the max priority fee, then the whole leftover fee is sent to the miner as the "priority fee" and no fee is refunded. The remainder of the leftover fee is refunded to the transaction initiator.
    
### Storage Operations And Gas Cost In Ethereum
17. Setting empty storage to a non-zero value costs 20,000 gas.
18. Setting a non-zero storage to a non-zero value costs 5,000 gas.
19. Setting a non-zero storage to the same value costs 100 gas.
20. Setting a storage from non-zero to zero results in a refund because you are freeing up storage. Ethereum rewards people who free up storage because these values take up real storage on nodes.
21. The first time you access a variable in storage costs an additional 2,100 gas (called "cold storage").
22. Subsequent accesses to a variable in storage cost an additional 100 gas. These fees exist to discourage people from abusing the storage system by storing values indefinitely, which can take up storage on nodes.
23. You cannot save on storage gas by using smaller integers or variable sizes because Ethereum treats all values as 32 bytes.


### Storage Arrays And Gas Costs
24. When an array is written to storage, the following operation occurs: the value of the array, and the length of the array is saved in storage.
25. When an array is written to storage, the total gas cost of that operation is derived by: 
     1. Gas cost for setting zero to non-zero value of the array (20,000 + 2,100 cold storage access gas = 22,000 gas) 
     2. Gas cost for non-zero to non-zero (2,900 + 2,100 = Gsreset + Gcoldsload cold storage access gas) 
     3. The length of the array is also set on the slot where the dynamic array was declared (so, additional, zero to non-zero length cost 22,100 total gas or non-zero to non-zero length cost 5000 total gas = total gas means with cold access)
26. This involves push, pop, or anything that affects the length of the array or its values.
27. This is why arrays are said to be expensive when dealing with them in storage.
28. Arrays in storage also follow the same gas cost mechanism as normal storage variables, where: 
     1. Setting the same data in the same storage location cost 100 gas
     2. Updating from non-zero to non-zero value cost 5000 gas
     3. Updating from zero to non-zero cost 20,000 gas for setting and additional 2,100 for cold storage access (first read in transaction)

### Gas Refunds
29. When a value is set from non-zero to zero, there is a gas refund given by the Ethereum network.
30. This includes going from a true to false, non-zero

### Solidity Optimizer And Its Effect On Gas
31. The Solidity optimizer, when used, affects the deployed code and gas cost to run the code. 
32. The number of runs (--optimizer runs) specifies roughly how often each opcode of the deployed code will be executed across the lifetime of the contract. This is a trade-off between code size (larger code size means a larger deployment cost) and code execution (more or fewer opcodes affect execution cost, so it's a trade-off between deployment cost and execution cost).
33. A "runs" parameter of "1" will be interpreted by the optimizer as the code being called just once after deployment, resulting in shorter code for cheaper deployment but with higher execution cost. 
34. A larger "runs" parameter will result in longer deployed code but more gas-efficient code for future execution (the opposite of the first statement). Simply put, a larger "runs" parameter optimizes for execution cost and not deployment cost, while a smaller "runs" parameter optimizes for deployment cost and not execution cost.
35. The maximum value for the "runs" parameter is 2**32-1.


# Contribute
We welcome contributions to Ethereum Dishes! If you have suggestions for improvements or new points to add, please see our [Contribution](https://github.com/Jesserc/Ethereum-Dishes/blob/main/CONTRIBUTIONS.md) Guidelines for more information.

# License
Ethereum Dishes is licensed under the [MIT](https://github.com/Jesserc/Ethereum-Dishes/tree/main) License.
