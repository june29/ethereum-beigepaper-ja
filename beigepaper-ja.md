# ベージュペーパー: Ethereum の技術仕様

## 概要

Ethereum のプロトコルは決定論的であるが、ふたつの基本的な機能を持つ実用的に無制限なステートマシンでもある。ひとつには、どこからでもアクセスできるシングルトンステートであり、もうひとつには、そのステートに変更を適用できる仮想マシンである。

## 1. コンピュータとしての Bitcoin を想像する

Ethereum は、Bitcoin に端を発する分散台帳モデルを仮想コンピュータを形成するために別の方法で活用し、マシンレベルの opcode に Bitcoin のトランザクションと同レベルの確実性を与える。Bitcoin の台帳が正確で、また Bitcoin のコンセンサス機構によってタイムスタンプも正しく記録されるのと同様に、Ethereum 上で開始された機械命令が実行されることも確実である。

言い換えれば、Ethereum の Blockchain 上で実行されるプログラムは基本的に止まらない。これは Ethereum のプログラムがバグを有しないことを意味するのではない。Ethereum のプログラムは外部の非ネットワーク的な力の影響を受けることなく実行されることが保証される、という意味だ。この特性は暗号法的証明の上に構築された Blockchain 固有のセキュリティによってもたらされる。

### 1.1. ネイティブ通貨

Ethereum は通貨アプリケーションを主とするのではなく、あらゆるアプリケーションで使用されるため、過剰な計算経費でネットワークを乱用する可能性を軽減するために用いる基本的なネットワークコスト単位がある。これは gas と呼ばれ、§3 で詳しく説明される。gas は ether でのみ支払われる。Ethereum で使われる通貨の最小単位は Wei であり、これは 1 ether の 10^-18 に等しい。Ethereum のすべての通貨トランザクションは、マシンレベルでは Wei で計上される。他には 10^-6 の Szabo や 10^-3 の Finney もある。

Ethereum のネットワークは、Ethereum のネイティブ通貨である ether という一点でのみ、他に従属している。システムができるすべてのことは、gas と引き換えに ether を消費する能力に縛られている。gas を使って、特定の量の計算能力を購入することができる。

## 2. Memory and Storage

### 2.1. World State

The world state is divided by blocks; each new block representing a new world state. The structure of the world state is a mapping of

1. addresses and
2. account states

through the use of the recursive length prefix standard (RLP). This information is stored as a merkle patricia tree in a database backend.a that maintains a map- ping of bytearrays to bytearrays.b c As a whole, the state is the sum total of database relationships in the state database.

#### 2.1.1. merkle patricia trees

merkle patricia trees are modified merkletrees where nodes represent individual characters from hashes rather than each node representing an entire hash. This allows the state data structure itself to represent not only the intrinsically correct paths in the data, but also the req- uisite cryptographic proofs which go into making sure that a piece of data was valid in the first place. In other words, it keeps the blockchain valid by combining the structure of a standard merkletree with the structure of a Radix Tree. Since all searching and sorting algorithms in Ethereum must be filtered through this stringently correct database, accuracy of information is guaranteed.

The following is a search tree beginning with hex- adecimal values a and 4:

### 2.2. Tree Terminology

- `a)` Root Node – The top (first) node in a tree.
- `b)` Child Node – A node directly connected to an- other node when moving away from the Root.
- `c)` Parent Node – The converse notion of a child. d) Sibling Nodes – A group of nodes with the same parent.
- `e)` Descendant Node – A node reachable by re- peated proceeding from parent to child.
- `f)` Ancestor Node – A node reachable by repeated proceeding from child to parent.
- `g)` Leaf Node – A node with no children.
- `h)` Branch Node – A node with at least one child.
- `i)` Degree – The number of subtrees of a node.
- `j)` Edge – The connection between one node and another.
- `k)` Path – A sequence of nodes and edges connecting a node with a descendant.
- `l)` Level – The level of a node is defined by 1 + (the number of connections between the node and the root).
- `m)` Node Height – The height of a node is the num- ber of edges on the longest path between that node and a leaf.
- `n)` Tree Height – The height of a tree is the height of its root node.
- `o)` Depth – The depth of a node is the number of edges from the tree's root node to the node.
- `p)` Forest – A forest is a set of n ≥ 0 disjoint trees.

#### 2.2.1. Recursive Length Prefix Encoding

Recursive Length Prefix Encoding (RLP) imposes a structure on data that intrinsically considers a prefixed hex value to position the data in the state database tree. This hex value determines the depth of a certain piece of data. There are two types of fundamental items one can encode in RLP:

1. Strings of bytes
2. Lists of other items

RLP encodes arrays of nested binary data to an ar- bitrary depth; it is the main serialization method for data in Ethereum. RLP encodes structure of data only, so it does not pay heed to the particular types of data being encoded.

Positive RLP integers are represented with the most significant value stored at the lowest memory address (big endian) and without any leading zeroes. As a re- sult, the RLP integer value for 0 is represented by an empty byte-array. If a non-empty deserialized integer begins with leading zeroes it is invalid.

The global state database is encoded as RLP for fast traversal and inspection of data. RLP encoding cre- ates a mapping between addresses and account states. Since it is stored on node operator's computers, the tree can be indexed and searched without network de- lay. RLP encodes values as byte-arrays, or as sequences of further values, which are subsequently encoded as byte-arrays.

### 2.3. The Block

A block is made up of 17 different elements. The first 15 elements are part of what is called the block header.

#### 2.3.1. The Block Header

Description : The information contained in a block besides the transactions list. This consists of:

1. Parent Hash – This is the Keccak-256 hash of the parent block's header.
2. Ommers Hash – This is the Keccak-256 hash of the ommer's list portion of this block.
3. Beneficiary – This is the 20-byte address to which all block rewards are transferred.
4. State Root – This is the Keccak-256 hash of the root node of the state trie, after a block and its transactions are finalized.
5. Transactions Root – This is the Keccak-256 hash of the root node of the trie structure popu- lated with each transaction from a Block's trans- action list.
6. Receipts Root – This is the Keccak-256 hash of the root node of the trie structure populated with the receipts of each transaction in the transactions list portion of the block.
7. Logs Bloom – This is the bloom filter composed from indexable information (log address and log topic) contained in the receipt for each transaction in the transactions list portion of a block.
8. Difficulty – This is the difficulty of this block – a quantity calculated from the previous block's difficulty and its timestamp.
9. Number – This is a quantity equal to the number of ancestor blocks behind the current block.
10. Gas Limit – This is a quantity equal to the cur- rent maximum gas expenditure per block.
11. Gas Used – This is a quantity equal to the total gas used in transactions in this block.
12. Timestamp – This is a record of Unix's time at this block's inception.
13. Extra Data – This byte-array of size 32 bytes or less contains extra data relevant to this block.
14. Mix Hash – This is a 32-byte hash that verifies a sufficient amount of computation has been done on this block.
15. Nonce – This is an 8-byte hash that verifies a sufficient amount of computation has been done on this block.
16. Ommer Block Headers – These are the same components listed above for any ommers.

#### 2.3.2. Block Footer

Transaction Series – This is the only non-header con- tent in the block.

#### 2.3.3. Block Number and Difficulty

Note that is the difficulty of the genesis block. The Homestead difficulty parameter, is used to affect a dynamic homeostasis of time between blocks, as the time between blocks varies, as discussed below, as im- plemented in EIP-2. In the Homestead release, the exponential difficulty symbol, causes the difficulty to slowly increase (every 100,000 blocks) at an exponential rate, and thus increasing the block time difference, and putting time pressure on transitioning to proof-of-stake. This effect, known as the "difficulty bomb", or "ice age", was explained in EIP-649 and delayed and implemented earlier in EIP-2, was also modified in EIP-100 with the use of x, the adjustment factor, and the denominator 9, in order to target the mean block time including uncle blocks. Finally, in the Byzantium release, with EIP-649, the ice age was delayed by creating a fake block number, which is obtained by substracting three million from the actual block number, which in other words reduced the time difference between blocks, in order to allow more time to develop proof-of-stake and preventing the network from "freezing" up.

#### 2.3.4. Account Creation

Account creation definitively occurs with contract cre- ation. Is related to: init. Lastly, there is the body which is the EVM-code that executes if/when the ac- count containing it receives a message call.

#### 2.3.5. Account State

The account state contains details of any particular account during some specified world state. The account state is made up of four variables:

1. nonce The number of transactions sent from this address, or the number of contract creations made by the account associated with this address.
2. balance The amount of Wei owned by this ac- count. Stored as a key/value pair inside the state database.
3. storage_root A 256-bit (32-byte) hash of the root node of a merkle patricia tree that encodes the storage contents of the account.
4. code_hash The hash of the EVM code of this account's contract. Code hashes are stored in the state database. Code hashes are permanent and they are executed when the address belonging to that account receives a message call.

#### 2.3.6. Bloom Filter

The Bloom Filter is composed from indexable informa- tion (logger address and log topics) contained in each log entry from the receipt of each transaction in the transactions list.

#### 2.3.7. Transaction Receipts

## 3. Processing and Computation

### 3.1. The Transaction

The basic method for Ethereum accounts to interact with each other. The transaction is a single cryptograph- ically signed instruction sent to the Ethereum network. There are two types of transactions: message calls and contract creations. Transactions lie at the heart of Ethereum, and are entirely responsible for the dynamism and flexibility of the platform. Transactions are the bread and butter of state transitions, that is of block additions, which contain all of the computation performed in one block. Each transaction applies the execution changes to the machine state, a temporary state which consists of all the required changes in com- putation that must be made before a block is finalized and added to the world state.

#### 3.1.1. Transactions Root

Notation : listhash

Alternatively: Transactions Root

Description : The Keccak-256 hash of the root node that precedes the transactions in the transactions_list section of a Block.

1. Nonce – The number of transactions sent by the sender.
2. Gas Price – The number of Wei to pay the net- work for unit of gas.
3. Gas Limit – The maximum amount of gas to be used in while executing a transaction.
4. To – The 20-character recipient of a message call.
5. Value The number of Wei to be transferred to the recipient of a message call.
6. v,r,s

### 3.2. State Transition Function

State Transitions come about through the State Transi- tion Function; this is a high-level abstraction of several operations in Ethereum which comprise the overall act of taking changes from the machine state and adding them to the world state.

### 3.3. Mining

The Block Beneficiary is the 160-bit (20-byte) ad- dress to which all fees collected from the successful mining of a block are transferred. Apply Rewards is the third process in block_finalization that sends the mining reward to an account's address. This is a scalar value corresponding to the difficulty level of a current block.

### 3.4. Verification

The process in The EVM that verifies Ommer Headers

### 3.5. Sender Function

A description that maps transactions to their sender using ECDSA of the SECP-256k1 curve,

### 3.6. Serialization/Deserialization

This function expands a positive-integer value to a big- endian byte-array of minimal length. When accompa- nied by a · operator, it signals sequence concatenation. The big_endian function accompanies RLP serializa- tion and deserialization.

### 3.7. Ethereum Virtual Machine

The EVM has a simple stack-based architecture. The word size of the machine and thus size of stack is 256- bit. This was chosen to facilitate the Keccak-256 hash scheme and elliptic-curve based computation. The mem- ory model is a simple word-addressed byte-array. The memory stack has a maximum size of 1024-bits. The machine also has an independent storage model; this is similar in concept to the memory but rather than a byte array, it is a word-addressable word array. Unlike memory, which is volatile, storage is non-volatile and is maintained as part of the system state.

All locations in both storage and memory are well- defined initially as zero. The machine does not follow the standard von Neumann architecture. Rather than storing program code in generally-accessible memory or storage, it is stored separately in a virtual ROM interactable only through specialized instructions.

The machine can have exceptional execution for sev- eral reasons, including stack underflows and invalid instructions. Like the out-of-gas exception, they do not leave state changes intact. Rather, the machine halts immediately and reports the issue to the execution agent (either the transaction processor or, recursively, the spawning execution environment) which will deal with it separately.

#### 3.7.1. Fees

Fees (denominated in gas) are charged under three distinct circumstances, all three as prerequisite to the execution of an operation. The first and most common is the fee intrinsic to the computation of the operation. Secondly, gas may be deducted in order to form the payment for a subordinate message call or contract cre- ation; this forms part of the payment for the CREATE, CALL and CALLCODE operations. Finally, gas may be paid due to an increase in the usage of the memory.

Over an account's execution, the total fee for memory- usage payable is proportional to smallest multiple of 32 bytes that are required such that all memory indices (whether for read or write) are included in the range. This is paid for on a just-in-time basis; as such, refer- encing an area of memory at least 32 bytes greater than any previously indexed memory will certainly result in an additional memory usage fee. Due to this fee it is highly unlikely that addresses will trend above 32-bit bounds.

Implementations must be able to manage this even- tuality. Storage fees have a slightly nuanced behaviour to incentivize minimization of the use of storage (which corresponds directly to a larger state database on all nodes), the execution fee for an operation that clears an entry in the storage is not only waived, a qualified refund is given; in fact, this refund is effectively paid up-front since the initial usage of a storage location costs substantially more than normal usage.

### 3.8. 実行

トランザクションの実行は、状態遷移関数stfを定義する。しかし、実行されるどんなトランザクションも初めに内部検証に合格する必要がある。

#### 3.8.1. 内部検証

検証の合格条件(トランザクションが満たすべき条件)

- トランザクションは、RLPフォーマットに従っている。(RLPはrecursive length prefixの略)
- トランザクションの署名が有効である。
- トランザクションのnonceが有効である。送信者の現在のnonceと等しいかどうか検証する。
- トランザクションに事前に定められているGas量よりも、gas limitが大きく設定されている。
- 送信者のアカウントの残高が、支払いの費用よりも上回っている。

#### 3.8.2. Transaction Receipt

While the amount of gas used in the execution and the accrued log items belonging to the transaction are stored, information concering the result of a transac- tion's execution is stored in the transaction receipt tx_receipt. The set of log events which are created through the execution of the transaction, logs_set in addition to the bloom filter which contains the ac- tual information from those log events logs_bloom are located in the transaction receipt. In addition, the post-transaction state post_transaction(state) and the amount of gas used in the block containing the transaction receipt post(gas_used) are stored in the transaction receipt. As a result, the transaction receipt is a record of any given execution.

A valid transaction execution begins with a perma- nent change to the state: the nonce of the sender ac- count is increased by one and the balance is decreased by the collateral_gas which is the amount of gas a transaction is required to pay prior to its execution. The original transactor will differ from the sender if the message call or contract creation comes from a contract account executing code.

After a transaction is executed, there comes a provi- sional state, which is a pre-final state. Gas used for the execution of individual EVM opcodes prior to their potential addition to the world_state creates:

- provisional state
- intrinsic gas, and an associated substate
- The accounts tagged for self-destruction following the transaction's completion. self_destruct(accounts)
- The logs_series, which creates checkpoints in EVM code execution for frontend applications to explore, and is made up of thelogs_set and logs_bloom from the tx_receipt.
- The refund balance.

Code execution always depletes gas. If gas runs out, an out-of-gas error is signaled (oog) and the resulting state defines itself as an empty set; it has no effect on the world state. This describes the transactional nature of Ethereum. In order to affect the world state, a transaction must go through completely or not at all.

#### 3.8.3. Code Deposit

If the initialization code completes successfully, a final contract-creation cost is paid, the code-deposit cost, c, proportional to the size of the created contract's code.

#### 3.8.4. Execution Model

Basics : The stack-based virtual machine which lies at the heart of the Ethereum and performs the actions of a computer. This is actually an instantial runtime that executes several substates, as EVM computation instances, before adding the finished result, all calcula- tions having been completed, to the final state via the finalization function.

In addition to the system state and the remaining gas for computation there are several pieces of impor- tant information used in the execution environment that the execution agent must provide:

- account_address, the address of the account which owns the code that is executing.
- sender_address the sender address of the trans- action that originated this execution.
- originator_price the price of gas in the transaction that originated this execution.
- input_data, a byte array that is the input data to this execution; if the execution agent is a trans- action, this would be the transaction data.
- account_address the address of the account which caused the code to be executing; if the ex- ecution agent is a transaction, this would be the transaction sender.
- newstate_value the value, in Wei, passed to this account if the execution agent is a transaction, this would be the transaction value.
- code array the byte array that is the machine code to be executed.
- block_header the block header of the present block.
- stack_depth the depth of the present message-call or contract-creation (i.e. the number of CALLs or CREATEs being executed at present).

The execution model defines the state_transition function, which can compute the resultant state, the remaining_gas, the accrued_substate and the resultant_output, given these definitions. For the present context, we will define it where the accrued sub- state is defined as the tuple of the self-destructs_set, the log_series, the touched_accounts and the refunds.

#### 3.8.5. Execution Overview

The execution_function, in most practical implemen- tations, will be modeled as an iterative progres- sion of the pair comprising the full system_state and the machine_state. It's defined recursively with the iterator_function, which defines the result of a single cycle of the state machine, together with the halting_check function, which determines if the present state is an exceptional halting state of the ma- chine and output_data of the instruction if the present state is a controlled_halt of the machine. An empty sequence/series indicates that execution should halt, while the empty set indicates that execution should continue.

When evaluating execution, we extract the remaining gas from the resultant machine state. It is thus cy- cled (recursively or with an iterative loop) until either exceptional_halt becomes true indicating that the present state is exceptional and that the machine must be halted and any changes discarded or until H becomes a series (rather than the empty set) indicating that the machine has reached a controlled halt.

The machine state is defined as the tuple which are the gas available, the program counter, the memory contents, the active number of words in memory (counting continuously from position 0), and the stack contents. The memory contents are a series of zeroes of size 2^256.

#### 3.8.6. The Execution Cycle

Stack items are added or removed from the left-most, lower-indexed portion of the series; all other items re- main unchanged: The gas is reduced by the instruction's gas cost and for most instructions, the program counter increments on each cycle, for the three exceptions, we assume a function J, subscripted by one of two instruc- tions, which evaluates to the according value: otherwise In general, we assume the memory, self-destruct set and system state don't change: however, instructions do typically alter one or several components of these values.

**Provisional State** A smaller, temporary state that is generated during transaction execution. It contains three sets of data.

#### 3.8.7. Message Calls

A message call can come from a transaction or inter- nally from contract code execution. It contains the field data, which consists of user data input to a message call. Messages allow communication between accounts (whether contract or external.) Messages can come in the form of msg_calls which give output data. If it is a contract account, this code gets executed when the account receives a message call. Message calls and contract creations are both transactions, but contract creations are never considered the same as message calls. Message calls always transfer some amount of value to another account. If the message call is an account cre- ation transaction then the value given takes on the role of an endowment towards the new account. Every time an account receives a message call it returns the body, something which is triggered by the init func- tion. User data input to a message_call, structured as an unlimited size byte-array.

**Universal Gas** Message calls always have a universally agreed-upon cost in gas. There is a strong distinction between contract creation transactions and message call transactions. Computation performed, whether it is a contract creation or a message call, represents the currently legal valid state. There can be no invalid transactions from this point.4 There is also a message call/contract creation stack. This stack has a depth, depending on how many transactions are in it. Contract creations and message calls have entirely different ways of executing, and are entirely different in their roles in Ethereum. The concepts can be conflated. Message calls can result in computation that occurs in the next state rather than the current one. If an account that is currently executing receives a message call, no code will execute, because the account might exist but has no code in it yet. To execute a message call transactions are required:

- sender
- transaction originator
- recipient
- account (usually the same as the recipient)
- available gas
- value
- gas price
- An arbitrary length byte-array. arb array
- present depth of the message call/contract cre- ation stack.

#### 3.8.8. Contract Creation

To initiate contract creation you need to send trans- action to nothing. This executes init and re- turns the body. Init is executed only once at ac- count_creation, and permanently discarded after that.

#### 3.8.9. Execution Environment

The Ethereum Runtime Environment is the environ- ment under which Autonomous Objects execute in the EVM: the EVM runs as a part of this environment.

#### 3.8.10. Big Endian Function

This function expands a positive-integer value to a big- endian byte array of minimal length. When accompa- nied by a · operator, it signals sequence concatenation. The big_endian function accompanies RLP serializa- tion and deserialization.

### 3.9. Gas

Gas is the fundamental network cost unit converted to and from ether as needed to complete the transaction while it is sent. Gas is arbitrarily determined at the moment it is needed, by the block and according to the total network's miners decision to charge certain fees. Each miner choose individually which gas prices they want to accept and which they want to reject.

#### 3.9.1. Gas Price/Gas Limit

Gas price is a value equal to the current limit of gas expenditure per block, according to the miners. Any unused gas is refunded to the sender. The canonical gas limit of a block is expressed and is stabilized by the time_stamp of the block.

**Gas Price Stability** Where new_header is the new block's header, but without the nonce and mix-hash components, d being the current DAG, a large data set needed to compute the mix-hash, and PoW is the proof-of-work function this evaluates to an array with the first item being the mix-hash, to prove that a cor- rect DAG has been used, and the second item being a pseudo-random number cryptographically dependent on it. Given an approximately uniform distribution in the range the expected time to find a solution is proportional to the difficulty.

This is the foundation of the security of the blockchain and is the fundamental reason why a malicious node cannot propagate newly created blocks that would oth- erwise overwrite ("rewrite") history. Because the nonce must satisfy this requirement, and because its satisfac- tion depends on the contents of the block and in turn its composed transactions, creating new, valid, blocks is difficult and, over time, requires approximately the total compute power of the trustworthy portion of the mining peers. Thus we are able to define the block header validity function.

**Gasused** A value equal to the total gas used in trans- actions in this block.

#### 3.9.2. Machine State

The machine state is a tuple consisting of five elements:

1. gas_available
2. program_counter
3. memory_contents A series of zeroes of size 2256 4. memory_words.count
5. stack_contents

There is also, [to_execute]: the current operation to be executed

#### 3.9.3. Exceptional Halting

An exceptional halt may be caused by four conditions existing on the stack with regard to the next opcode in line for execution:

```
if
out_of_gas = true
or
bad_instruction = true
or
bad_stack_size = true
or
bad_jumpdest = true
then throw exception
else exec opcode x
then init control_halt
```

Exceptional halts are reserved for opcodes that fail to execute. They can never be caused through an opcode's actual execution.

- The amount of remaining gas in each transaction is extracted from information contained in the machine_state
- A simple iterative recursive loop4 with a Boolean value:
  - true indicating that in the run of computa- tion, an exception was signaled
  - false indicating in the run of computation, no exceptions were signaled. If this value re- mains false for the duration of the execution until the set of transactions becomes a series (rather than an empty set.) This means that the machine has reached a controlled halt.

**Substate** A smaller, temporary state that is gener- ated during transaction execution and runs parallel to machine state. It contains three sets of data:

- The accounts tagged for self-destruction following the transaction's completion. self_destruct(accounts)
- The logs_series, which creates checkpoints in EVM code execution for frontend applications to explore, and is made up of thelogs_set and l ogs_bloom from the tx_receipt.
- The refund balance.

#### 3.9.4. EVM Code

The bytecode that the EVM can natively execute. Used to explicitly specify the meaning of a message to an account. A contract is a piece of EVM Code that may be associated with an Account or an Autonomous Object. EVM Assembly is the human readable version of EVM Code.

### 3.10. Blocktree to Blockchain

The canonical blockchain is a path from root to leaf through the entire block tree. In order to have consen- sus over which path it is, conceptually we identify the path that has had the most computation done upon it, or, the heaviest path. Clearly one factor that helps determine the heaviest path is the block number of the leaf, equivalent to the number of blocks, not counting the unmined genesis block, in the path. The longer the path, the greater the total mining effort that must have been done in order to arrive at the leaf. This is akin to existing schemes, such as that employed in Bitcoin-derived protocols. Since a block header includes the difficulty, the header alone is enough to validate the computation done. Any block contributes toward the total computation or total difficulty of a chain. Thus we define the total difficulty of this_block re- cursively by the difficulty of its parent block and the block itself. The jobs of miners and validators are as follows: Validate (or, if mining, determine) ommers; validate (or, if mining, determine) transactions; apply rewards; verify (or, if mining, compute a valid) state and nonce.

### 3.11. Ommer Validation

The validation of ommer headers means nothing more than verifying that each ommer header is both a valid header and satisfies the relation of Nth-generation om- mer to the present block. The maximum of ommer headers is two.

### 3.12. Transaction Validation

The given gasUsed must correspond faithfully to the transactions listed, the total gas used in the block, must be equal to the accumulated gas used according to the final transaction.

### 3.13. Reward Application

The application of rewards to a block involves rais- ing the balance of the accounts of the beneficiary address of the block and each ommer by a certain amount. We raise the block's beneficiary account; for each ommer, we raise the block's beneficiary by 1 an additional 32 of the block reward and the beneficiary of the ommer gets rewarded depending on the block number. This constitutes the block_finalization state_transition_function. If there are collisions of the beneficiary addresses between ommers and the block two ommers with the same beneficiary address or an ommer with the same beneficiary address as the present block, additions are applied cumulatively. The block reward is three ether per block.

paragraphState & Nonce Validation The function that maps a block B to its initiation state, that is,the hash of the root node of a trie of state x. This value is stored in the state database trivial and efficient since the trie is by nature a resilient data structure. And finally define the block_transition_function, which maps an incomplete block to a complete block with a specified dataset. As specified at the beginning of the present work, the state_transition_function, which is de- fined in terms of, the block_finalisation_function and, the transaction_evaluation_function. As previously detailed, there is the nth corresponding status code, logs and cumulative gas used after each transac- tion, the fourth component in the tuple, has already been defined in terms of the logs).

The nth state is given from applying the correspond- ing transaction to the state resulting from the previous transaction (or the block's initial state in the case of the first BYZANTIUM VERSION 3475aa8 – 2018-01-26 14 such transaction): otherwise in certain cases there is a similar approach defining each item as the gas used in evaluating the corresponding transaction summed with the previous item (or zero, if it is the first), giv- ing us a running total: the function is used that was defined in the transaction execution function. Finally new state exists in the context of the block reward function applied to the final transaction's resultant state, thus the complete block-transition mechanism, less PoW, the proof-of-work function is defined.

### 3.14. Mining Proof-of-Work

Proof that a certain amount of mining has been done exists as a cryptographic probability statement which as- serts beyond reasonable doubt that a particular amount of computation has been expended in the determination of some token value pow_token. It is utilised to enforce the security of the blockchain. Since mined blocks pro- duce a reward, the proof-of-work also serves as a wealth distribution mechanism. For this reason, the proof of work function is designed to be as accessible as possible to as many people as possible.

A very basic application of this principle of accessibil- ity is found in combining the traditional Proof-of-Work function with a Memory-Hardness function. By forc- ing the hashing algorithm to use memory as well as CPU, miners are more likely to use computers than ASICs, meaning that ASIC efficiency will not obsolete the person who wants to mine on their home computer from participating in the mining process. To make the Ethereum Blockchain ASIC resistant, the Proof-of- Work mechanism has been designed to be sequential and memory-hard. This means that the nonce requires high amounts of memory and bandwidth such that the memory cannot be used in parallel to discover multiple nonces simultaneously. Therefore, the proof-of-work function takes the form of 2256 the new block's header but without the nonce and mix-hash components. There is the header_nonce, and data_set which are required to compute the mix hash and block_difficulty, the difficulty value of the new block. The proof-of-work function evaluates to an array with the first item being the mix hash and the second item being a pseudoran- dom number which is cryptographically dependent on the header_nonce and the data_set. The name for this algorithm is **Ethash**.

#### 3.14.1. Ethash: Seed→Cache→Dataset→Slice

Ethash is the Proof-of-Work algorithm which was used to launch the Ethereum network and bring it through its first few releases. It is in the process of being gradually phased out and replaced with a Proof-of-Stake model. For now it is the latest version of Dagger-Hashimoto, introduced by Vitalik Buterin. The general route that the algorithm takes is as follows: There exists a seed which can be computed for each block by scanning through the block headers up until that point. From the seed, one can compute a pseudorandom cache, that is cache_init bytes in initial size. Light clients store the cache. From the cache, a dataset is generated, dataset_size bytes in initial size, with the property that each item in the dataset depends on only a small number of items from the cache. Full clients and miners store the dataset. The dataset grows linearly with time. Mining involves grabbing random slices of the dataset and hashing them together. Verification can be done with low memory by using the cache to regenerate the specific pieces of the dataset that you need, so you only need to store the cache. The large dataset is updated once every 1 epoch (10,000) blocks, so the vast major- ity of a miner's effort is spent on reading the dataset, rather than on making changes to it.

#### 3.14.2. Difficulty Mechanism

This mechanism enforces a relative predictability in terms of the time-window between blocks; a smaller pe- riod between the last two blocks results in an increase in the difficulty level and thus additional computation re- quired, lengthening the next time-window. Conversely, if the time-window is too large, the difficulty is reduced, reducing the amount of time to the next block. The total_difficultya is the difficulty_state of the entire Ethereum blockchain. The block_difficulty, in contrast, is not a state of the blockchain, but is local– particular to each specific block. You reach the total difficulty by summing the individual difficulty of all previous blocks and then adding the difficulty of the present block.

The **GHOST Protocol** provides an alternative solu- tion to double-spend attacks from the original solution in Satoshi Nakamoto's Bitcoin Whitepaper. Nakamoto solved the problem of double-spending by requiring the network to agree on a single block in order to function. For that reason, in the Bitcoin protocol, it's impossible to submit a "double-spend" block without having at least 50% of the network's mining power to force the longest chain. This is because the network automati- cally chooses the longest chain. So even if one wanted to submit two spend transactions in a row, the network simply picks whichever one comes first, ignoring the second because it no longer pertains to the longest chain (which now contains the first block that was sent) so the would-be hacker needs to submit a new block, as the first double block is no longer feasible.

The "GHOST Protocol" (which stands for Greedy Heaviest Object subTree) rather requires that miners begin mining whichever chain the most other miners are on. Because of differences in network propagation of data about which miners are mining which block, this has a tendency to create more uncles. Nevertheless, in spite of the increased amount of uncle blocks, the chain itself is equally secure, and this method allows for higher throughput of transactions than Satoshi's solution to double-spending does.

### 3.15. Pseudorandom Numbers

Pseudo-random numbers may be generated by utilizing data which is generally unknowable at the time of transacting. This constitutes anything based off of factors which are unknowable under regular circumstances, but become knowable through the regular operation and growth of the chain. Such data might include a current (or relatively current) block's hash, timestamp, or beneficiary address. The blockhash opcode uses the previous 256 blocks as pseudo-random numbers. One could further automate this randomness by adding two blockhash operations and hashing the result.

### 3.16. Chainsize Limits

The state database usually won't store every single tree structure in the history of the blockchain. One idea has been proposed to simply maintain node checkpoints for each age (10,000 blocks) and eventually discard check- points which no longer contain necessary state data. This would be a variation on a compression scheme.

### 3.17. Scalability

Scalability is a constant concern. Because Ethereum's state transitions are so broad in terms of possible con- tent, and because its applications and use-cases are so numerous in the number of potential transactions re- quired, scalability is inherently necessary for increased transaction throughput and for more efficient storage and traversal of the chain.

#### 3.17.1. Sharding

Parallelization of transaction combination and block building.

#### 3.17.2. Casper

#### 3.17.3. Plasma
