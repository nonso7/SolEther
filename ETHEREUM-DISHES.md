# Ethereum-Dishes
Random knowledge about Ethereum and Solidity


# NOTES ON GAS
#### GAS AND GAS SAVINGS TECHNIQUES

1. The gas cost of a transaction in Ethereum can be affected by several factors:

   - The size of the transaction data. Every non-zero byte of data costs 16 gas.

   - The size and amount of memory covered. Each memory expansion costs 3 gas. Memory is divided into slots ranging from 0x00-0x20 (32 bytes), 0x20-0x40 (another 32 bytes), and so on. If you store a value in memory at offset 0x40 (which goes from 0x40 to 0x60), which is the third slot in memory, you will pay 3 gas * 2 slots (6 gas) for the two memory expansions because you have skipped the first two slots (0x00-0x20 and 0x20-0x40).

   - The gas cost for the opcodes used in the transaction.

   - The amount of storage used.

2. Payable functions are cheaper than non-payable functions because the EVM uses the ISZERO opcode to check if the CALLVALUE opcode is zero or not during a function in a non-payable function and reverts if value was sent along with it. This adds extra opcodes which are not present in a payable function and thus increases the gas cost.

3. The same principle applies to the unchecked keyword in Solidity, but in the opposite manner. The unchecked keyword removes extra opcodes that the EVM uses to check if the resulting value of an arithmetic plus/minus operation is greater than (for plus) or less than (for minus) any of the factors of the operation.

4. Every transaction (except contract deployment) in Ethereum costs at least 21,000 gas (plus any other execution cost, such as the gas cost of running opcodes if it is a smart contract interaction). This value is imposed by the protocol by default. This is because when a transaction is carried out, it goes through an initial test of intrinsic validity and the following actions are carried out:

   - The transaction is well-formed RLP (recursive length prefix) with no additional trailing bytes.

   - The transaction signature is valid.

   - The transaction nonce is valid (equivalent to the sender account's current nonce, to prevent signature replay).

   - The sender account has no contract code deployed (see EIP-3607).

   - The gas limit is no smaller than the intrinsic gas used by the transaction.

   - The sender account balance contains at least the cost required in up-front payment for the transaction.
   (All of these can be found on page 8 of the Yellowpaper.)

5. You can access the block base fee from Solidity ^0.8.7 using the block.basefee global variable.

#### SOLIDITY EIP-1559 AND GAS FEE TYPES

6. The transaction fee is calculated by:
    - max gas fee or gas price * gas used / 1 gwei (1 followed by 9 zeros, which is equal to 1 billion in number)

7. According to EIP-1559, the gas fees go up and down depending on the network condition of the protocol. There are several gas terms you should consider because of EIP-1559:

    - Max fee: The maximum amount of fee you are willing to pay for each unit of gas for a transaction.

    - Max priority fee: The fee set aside from your max fee that is used as a tip for the miner of your transaction.

    - Base fee: The minimum fee (all in gwei) required to carry out a transaction in a block. It is set at the protocol level and not by the transaction initiator. The algorithm for gas increases the base gas fee of a block by 12% if the 3 million block gas limit is reached, and decreases it by 12% if the block gas limit was not reached.

    - Priority fee: In the case where there is a leftover gas fee after executing a transaction and taking the base fee (the leftover fee is the max fee minus the base fee, in the case where the base fee is less than the max fee and not equal to it), the max priority fee is taken from this leftover fee and sent to the miner as a tip. If the leftover fee is less than the max priority fee, then the whole leftover fee is sent to the miner as the "priority fee" and no fee is refunded. The remainder of the leftover fee is refunded to the transaction initiator.
    
#### STORAGE AND GAS COST IN ETHEREUM
8. Setting empty storage to a non-zero value costs 20,000 gas.
9. Setting a non-zero storage to a non-zero value costs 5,000 gas.
10. Setting a non-zero storage to the same value costs 100 gas.
11. Setting a storage from non-zero to zero results in a refund because you are freeing up storage. Ethereum rewards people who free up storage because these values take up real storage on nodes.
12. The first time you access a variable in storage costs an additional 2,100 gas (called "cold storage").
13. Subsequent accesses to a variable in storage cost an additional 100 gas. These fees exist to discourage people from abusing the storage system by storing values indefinitely, which can take up storage on nodes.
14. You cannot save on storage gas by using smaller integers or variable sizes because Ethereum treats all values as 32 bytes.

#### SOLIDITY OPTIMIZER
15. The Solidity optimizer, when used, affects the deployed code and gas cost to run the code. 
16. The number of runs (--optimizer runs) specifies roughly how often each opcode of the deployed code will be executed across the lifetime of the contract. This is a trade-off between code size (larger code size means a larger deployment cost) and code execution (more or fewer opcodes affect execution cost, so it's a trade-off between deployment cost and execution cost).
17. A "runs" parameter of "1" will be interpreted by the optimizer as the code being called just once after deployment, resulting in shorter code for cheaper deployment but with higher execution cost. 
18. A larger "runs" parameter will result in longer deployed code but more gas-efficient code for future execution (the opposite of the first statement). Simply put, a larger "runs" parameter optimizes for execution cost and not deployment cost, while a smaller "runs" parameter optimizes for deployment cost and not execution cost.
19. The maximum value for the "runs" parameter is 2**32-1.
