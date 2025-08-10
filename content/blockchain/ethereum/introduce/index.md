---
title: How does Ethereum work,anyway?
date: 2018-03-21T04:30:40+00:00
categories:
  - 区块链
tags:
  - 以太坊
series:
  - 以太坊
series_order: 1
---

### <span class="ez-toc-section" id="Introduction"></span>Introduction<span class="ez-toc-section-end"></span> {#0c04.graf.graf--h3.graf-after--figure}

<p class="graf graf--p graf-after--h3">
  Odds are you’ve heard about the Ethereum blockchain, whether or not you know what it is. It’s been in the news a lot lately, including the cover of some major magazines, but reading those articles can be like gibberish if you don’t have a foundation for what exactly Ethereum is. So what is it? In essence, a public database that keeps a permanent record of digital transactions. Importantly, this database doesn’t require any central authority to maintain and secure it. Instead it operates as a “trustless” transactional system — a framework in which individuals can make peer-to-peer transactions without needing to trust a third party OR one another.
</p>

<p id="f810" class="graf graf--p graf-after--p">
  Still confused? That’s where this post comes in. My aim is to explain how Ethereum functions at a technical level, without complex math or scary-looking formulas. Even if you’re not a programmer, I hope you’ll walk away with at least better grasp of the tech. If some parts are too technical and difficult to grok, that’s totally fine! There’s really no need to understand every little detail. I recommend just focusing on understanding things at a broad level.
</p>

Many of the topics covered in this post are a breakdown of the concepts discussed in the yellow paper. I’ve added my own explanations and diagrams to make understanding Ethereum easier. Those brave enough to take on the technical challenge can also read the Ethereum yellow paper.

<p id="4886" class="graf graf--p graf-after--p">
  Let’s get started!
</p>

### <span class="ez-toc-section" id="Blockchain_definition"></span>Blockchain definition<span class="ez-toc-section-end"></span> {#aa57.graf.graf--h3.graf-after--p}

<p id="a7c9" class="graf graf--p graf-after--h3">
  A blockchain is a “<strong class="markup--strong markup--p-strong">cryptographically secure transactional singleton machine with shared-state.” </strong>[1] That’s a mouthful, isn’t it? Let’s break it down.
</p>

<ul class="postList">
  <li id="eb22" class="graf graf--li graf--startsWithDoubleQuote graf-after--p">
    <strong class="markup--strong markup--li-strong">“Cryptographically secure” </strong>means that the creation of digital currency is secured by complex mathematical algorithms that are obscenely hard to break. Think of a firewall of sorts. They make it nearly impossible to cheat the system (e.g. create fake transactions, erase transactions, etc.)
  </li>
  <li id="78ac" class="graf graf--li graf--startsWithDoubleQuote graf-after--li">
    <strong class="markup--strong markup--li-strong">“Transactional singleton machine”</strong> means that there’s a single canonical instance of the machine responsible for all the transactions being created in the system. In other words, there’s a single global truth that everyone believes in.
  </li>
  <li id="be83" class="graf graf--li graf--startsWithDoubleQuote graf-after--li">
    <strong class="markup--strong markup--li-strong">“With shared-state”</strong> means that the state stored on this machine is shared and open to everyone.
  </li>
</ul>

<p id="3a10" class="graf graf--p graf-after--li">
  Ethereum implements this blockchain paradigm.
</p>

### <span class="ez-toc-section" id="The_Ethereum_blockchain_paradigm_explained"></span>The Ethereum blockchain paradigm explained<span class="ez-toc-section-end"></span> {#e17d.graf.graf--h3.graf-after--p}

<p class="graf graf--p graf-after--h3">
  The Ethereum blockchain is essentially a <strong class="markup--strong markup--p-strong">transaction-based</strong> <strong class="markup--strong markup--p-strong">state machine</strong>. In computer science, a <em class="markup--em markup--p-em">state machine</em> refers to something that will read a series of inputs and, based on those inputs, will transition to a new state.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-151 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-HYjnPbo-_fqpgWhPHrmrow.png" alt="" width="800" height="586" /> 

With Ethereum’s state machine, we begin with a “genesis state.” This is analogous to a blank slate, before any transactions have happened on the network. When transactions are executed, this genesis state transitions into some final state. At any point in time, this final state represents the current state of Ethereum.

<img decoding="async" loading="lazy" class="aligncenter wp-image-154 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-jZ-VRXBJtOnePofB0z2Q8A.png" alt="" width="800" height="188" /> 

The state of Ethereum has millions of transactions. These transactions are grouped into “blocks.” A block contains a series of transactions, and each block is chained together with its previous block.

<img decoding="async" loading="lazy" class="aligncenter wp-image-155 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-l_H58_pGm3XGwGoQlO9FVQ.png" alt="" width="800" height="255" /> 

<p id="2d90" class="graf graf--p graf-after--figure">
  To cause a transition from one state to the next, a transaction must be valid. <strong class="markup--strong markup--p-strong">For a transaction to be considered valid, it must go through a validation process known as mining. </strong>Mining is when a group of nodes (i.e. computers) expend their compute resources to create a block of valid transactions.
</p>

<p id="17b0" class="graf graf--p graf-after--p">
  Any node on the network that declares itself as a miner can attempt to create and validate a block. Lots of miners from around the world try to create and validate blocks at the same time. Each miner provides a mathematical “proof” when submitting a block to the blockchain, and this proof acts as a guarantee: if the proof exists, the block must be valid.
</p>

<p id="08bc" class="graf graf--p graf-after--p">
  For a block to be added to the main blockchain, the miner must prove it faster than any other competitor miner. The process of validating each block by having a miner provide a mathematical proof is known as a <strong class="markup--strong markup--p-strong">“proof of work.”</strong>
</p>

<p id="d463" class="graf graf--p graf-after--p">
  A miner who validates a new block is rewarded with a certain amount of value for doing this work. What is that value? The Ethereum blockchain uses an intrinsic digital token called “Ether.” Every time a miner proves a block, new Ether tokens are generated and awarded.
</p>

<p id="5333" class="graf graf--p graf-after--p">
  You might wonder: what guarantees that everyone sticks to one chain of blocks? How can we be sure that there doesn’t exist a subset of miners who will decide to create their own chain of blocks?
</p>

<p id="3579" class="graf graf--p graf-after--p">
  Earlier, we defined a blockchain as a<strong class="markup--strong markup--p-strong"> transactional singleton machine with shared-state</strong>. Using this definition, we can understand the correct current state is a single global truth, which everyone must accept. Having multiple states (or chains) would ruin the whole system, because it would be impossible to agree on which state was the correct one. If the chains were to diverge, you might own 10 coins on one chain, 20 on another, and 40 on another. In this scenario, there would be no way to determine which chain was the most “valid.”
</p>

<p id="da30" class="graf graf--p graf-after--p">
  Whenever multiple paths are generated, a “fork” occurs. We typically want to avoid forks, because they disrupt the system and force people to choose which chain they “believe” in.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-162 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-O1fLdUJhfL5YOrUw0C0ivA.png" alt="" width="1000" height="521" /> 

<div class="section-inner sectionLayout--insetColumn">
  <p id="dcb7" class="graf graf--p graf-after--figure">
    To determine which path is most valid and prevent multiple chains, Ethereum uses a mechanism called the <strong class="markup--strong markup--p-strong">“GHOST protocol.”</strong>
  </p>
  
  <p id="916b" class="graf graf--p graf--startsWithDoubleQuote graf-after--p">
    <span class="markup--quote markup--p-quote is-other" data-creator-ids="anon"><strong class="markup--strong markup--p-strong">“GHOST” = “Greedy Heaviest Observed Subtree”</strong></span>
  </p>
  
  <p id="d374" class="graf graf--p graf-after--p">
    In simple terms, <strong class="markup--strong markup--p-strong">the GHOST protocol says we must pick the path that has had the most computation done upon it</strong>. One way to determine that path is to use the block number of the most recent block (the “leaf block”), which represents the total number of blocks in the current path (not counting the genesis block). The higher the block number, the longer the path and the greater the mining effort that must have gone into arriving at the leaf. Using this reasoning allows us to agree on the canonical version of the current state.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-160 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-MZCUJTeZvgQsTDYPQOw_Vg.png" alt="" width="1000" height="277" />
  </p>
  
  <p id="6486" class="graf graf--p graf-after--figure">
    Now that you’ve gotten the 10,000-foot overview of what a blockchain is, let’s dive deeper into the main components that the Ethereum system is comprised of:
  </p>
  
  <ul class="postList">
    <li id="de64" class="graf graf--li graf-after--p">
      accounts
    </li>
    <li id="950c" class="graf graf--li graf-after--li">
      state
    </li>
    <li id="4563" class="graf graf--li graf-after--li">
      gas and fees
    </li>
    <li id="dbde" class="graf graf--li graf-after--li">
      transactions
    </li>
    <li id="00cc" class="graf graf--li graf-after--li">
      blocks
    </li>
    <li id="5eaf" class="graf graf--li graf-after--li">
      transaction execution
    </li>
    <li id="5c86" class="graf graf--li graf-after--li">
      mining
    </li>
    <li id="97a5" class="graf graf--li graf-after--li">
      proof of work
    </li>
  </ul>
  
  <p id="71fb" class="graf graf--p graf-after--li">
    <em class="markup--em markup--p-em">One note before getting started: whenever I say “hash” of X, I am referring to the </em><a class="markup--anchor markup--p-anchor" href="https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use" target="_blank" rel="nofollow noopener" data-href="https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use"><em class="markup--em markup--p-em">KECCAK-256</em></a><em class="markup--em markup--p-em"> hash, which Ethereum uses.</em>
  </p>
  
  <h3 id="4308" class="graf graf--h3 graf-after--p">
    <span class="ez-toc-section" id="Accounts"></span>Accounts<span class="ez-toc-section-end"></span>
  </h3>
  
  <p id="92bc" class="graf graf--p graf-after--h3">
    The global “shared-state” of Ethereum is comprised of many small objects (“accounts”) that are able to interact with one another through a message-passing framework. Each account has a <strong class="markup--strong markup--p-strong">state</strong> associated with it and a 20-byte <strong class="markup--strong markup--p-strong">address</strong>. An address in Ethereum is a 160-bit identifier that is used to identify any account.
  </p>
  
  <p id="4191" class="graf graf--p graf-after--p">
    There are two types of accounts:
  </p>
  
  <ul class="postList">
    <li id="f8ae" class="graf graf--li graf-after--p">
      Externally owned accounts, which are <strong class="markup--strong markup--li-strong">controlled by private keys </strong>and have <strong class="markup--strong markup--li-strong">no code associated with them.</strong>
    </li>
    <li id="9d36" class="graf graf--li graf-after--li">
      Contract accounts, which are <strong class="markup--strong markup--li-strong">controlled by their contract code </strong>and <strong class="markup--strong markup--li-strong">have code associated with them.</strong>
    </li>
  </ul>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-150 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-HIpgnjvHE9Jpb58icF5MgQ.png" alt="" width="758" height="380" />
  </p>
  
  <h4 id="88e8" class="graf graf--h4 graf-after--figure">
    <span class="ez-toc-section" id="Externally_owned_accounts_vs_contract_accounts"></span>Externally owned accounts vs. contract&nbsp;accounts<span class="ez-toc-section-end"></span>
  </h4>
  
  <p id="2a9a" class="graf graf--p graf-after--h4">
    It’s important to understand a fundamental difference between externally owned accounts and contract accounts. <strong class="markup--strong markup--p-strong">An externally owned account can send messages to other externally owned accounts OR to other contract accounts by creating and signing a transaction using its private key.</strong> A message between two externally owned accounts is simply a value transfer. But a message from an externally owned account to a contract account activates the contract account’s code, allowing it to perform various actions (e.g. transfer tokens, write to internal storage, mint new tokens, perform some calculation, create new contracts, etc.).
  </p>
  
  <p id="48d7" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">Unlike externally owned accounts, contract accounts can’t initiate new transactions on their own.</strong> Instead, contract accounts can only fire transactions in response to other transactions they have received (from an externally owned account or from another contract account). We’ll learn more about contract-to-contract calls in the “<em class="markup--em markup--p-em">Transactions and Messages</em>” section.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-152 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-I635Y9btMh667inOhDBQ_g.png" alt="" width="800" height="394" />
  </p>
  
  <p>
    <strong class="markup--strong markup--p-strong">Therefore, any action that occurs on the Ethereum blockchain is always set in motion by transactions fired from externally controlled accounts.</strong>
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-156 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-LbHK28pCiB5dHRwpTMr30w.png" alt="" width="800" height="673" />
  </p>
  
  <h4 id="e73a" class="graf graf--h4 graf-after--figure">
    <span class="ez-toc-section" id="Account_state"></span>Account state<span class="ez-toc-section-end"></span>
  </h4>
  
  <p id="2fea" class="graf graf--p graf-after--h4">
    The account <strong class="markup--strong markup--p-strong">state</strong> consists of four components, which are present regardless of the type of account:
  </p>
  
  <ul class="postList">
    <li id="219a" class="graf graf--li graf-after--p">
      <strong class="markup--strong markup--li-strong">nonce</strong>: If the account is an externally owned account, this number represents the number of transactions sent from the account’s address. If the account is a contract account, the nonce is the number of contracts created by the account.
    </li>
    <li id="83d9" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">balance</strong>: The number of Wei owned by this address. There are 1e+18 Wei per Ether.
    </li>
    <li id="0d08" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">storageRoot</strong>: A hash of the root node of a Merkle Patricia tree (we’ll explain Merkle trees later on). This tree encodes the hash of the storage contents of this account, and is empty by default.
    </li>
    <li id="2fec" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">codeHash</strong>: The hash of the EVM (Ethereum Virtual Machine — more on this later) code of this account. For contract accounts, this is the code that gets hashed and stored as the <strong class="markup--strong markup--li-strong">codeHash</strong>. For externally owned accounts, the <strong class="markup--strong markup--li-strong">codeHash</strong> field is the hash of the empty string.
    </li>
  </ul>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-149 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-fYY7nqa6NZkDS4WSkXmQoQ.png" alt="" width="800" height="324" />
  </p>
  
  <h4 id="9db4" class="graf graf--h4 graf-after--figure">
    <span class="ez-toc-section" id="World_state"></span>World state<span class="ez-toc-section-end"></span>
  </h4>
  
  <p id="b40a" class="graf graf--p graf-after--h4">
    Okay, so we know that Ethereum’s global state consists of a mapping between account addresses and the account states. This mapping is stored in a data structure known as a <strong class="markup--strong markup--p-strong">Merkle Patricia tree</strong>.
  </p>
  
  <p id="d1de" class="graf graf--p graf-after--p">
    A Merkle tree (or also referred as “Merkle trie”) is a type of <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Binary_tree" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Binary_tree">binary tree</a> composed of a set of nodes with:
  </p>
  
  <ul class="postList">
    <li id="0c74" class="graf graf--li graf-after--p">
      a large number of leaf nodes at the bottom of the tree that contain the underlying data
    </li>
    <li id="0261" class="graf graf--li graf-after--li">
      a set of intermediate nodes, where each node is the hash of its two child nodes
    </li>
    <li id="df62" class="graf graf--li graf-after--li">
      a single root node, also formed from the hash of its two child node, representing the top of the tree
    </li>
  </ul>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-144 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-5xWwPX2R8d37MeWSFOJPnA.png" alt="" width="800" height="388" />
  </p>
  
  <p>
    The data at the bottom of the tree is generated by splitting the data that we want to store into <em class="markup--em markup--p-em">chunks</em>, then splitting the chunks into <em class="markup--em markup--p-em">buckets</em>, and then taking the hash of each bucket and repeating the same process until the total number of hashes remaining becomes only one: <strong class="markup--strong markup--p-strong">the root hash.</strong>
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-165 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-TXlzvc6bh565UYLZC3Q6Vw.png" alt="" width="800" height="436" />
  </p>
  
  <p>
    This tree is required to have a key for every value stored inside it. Beginning from the root node of the tree, the key should tell you which child node to follow to get to the corresponding value, which is stored in the leaf nodes. In Ethereum’s case, the key/value mapping for the state tree is between addresses and their associated accounts, including the balance, nonce, codeHash, and storageRoot for each account (where the storageRoot is itself a tree).
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-161 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-N8YtAcAqdtHtzUaZAJgUJQ.png" alt="" width="800" height="653" />
  </p>
  
  <p id="c41e" class="graf graf--p graf-after--figure">
    This same trie structure is used also to store transactions and receipts. More specifically, every block has a “header” which stores the hash of the root node of three different Merkle trie structures, including:
  </p>
  
  <ol class="postList">
    <li id="5dfb" class="graf graf--li graf-after--p">
      State trie
    </li>
    <li id="f9a3" class="graf graf--li graf-after--li">
      Transactions trie
    </li>
    <li id="5ea3" class="graf graf--li graf-after--li">
      Receipts trie
    </li>
  </ol>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-142 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-4EQFjXD2-dbiVgVv-8Si8g.png" alt="" width="800" height="553" />
  </p>
  
  <p id="f484" class="graf graf--p graf-after--figure">
    The ability to store all this information efficiently in Merkle tries is incredibly useful in Ethereum for what we call “light clients” or “light nodes.” Remember that a blockchain is maintained by a bunch of nodes. Broadly speaking, there are two types of nodes: full nodes and light nodes.
  </p>
  
  <p id="9d72" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">A full archive node synchronizes the blockchain by downloading the full chain, from the genesis block to the current head block, executing all of the transactions contained within.</strong> Typically, miners store the full archive node, because they are required to do so for the mining process. It is also possible to download a full node without executing every transaction. Regardless, any full node contains the entire chain.
  </p>
  
  <p id="4ea6" class="graf graf--p graf-after--p">
    But unless a node needs to execute every transaction or easily query historical data, there’s really no need to store the entire chain. This is where the concept of a light node comes in. <strong class="markup--strong markup--p-strong">Instead of downloading and storing the full chain and executing all of the transactions, light nodes download only the chain of headers, from the genesis block to the current head, without executing any transactions or retrieving any associated state. </strong>Because light nodes have access to block headers, which contain hashes of three tries, they can still easily generate and receive verifiable answers about transactions, events, balances, etc.
  </p>
  
  <p id="dfbd" class="graf graf--p graf-after--p">
    The reason this works is because hashes in the Merkle tree propagate upward — if a malicious user attempts to swap a fake transaction into the bottom of a Merkle tree, this change will cause a change in the hash of the node above, which will change the hash of the node above that, and so on, until it eventually changes the root of the tree.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-159 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-MwnTtA5Q-EOAX8JCQEJp5g.png" alt="" width="800" height="878" />
  </p>
  
  <p id="9502" class="graf graf--p graf-after--figure">
    Any node that wants to verify a piece of data can use something called a “Merkle proof” to do so. A Merkle proof consists of:
  </p>
  
  <ol class="postList">
    <li id="5ab6" class="graf graf--li graf-after--p">
      A chunk of data to be verified and its hash
    </li>
    <li id="a6d9" class="graf graf--li graf-after--li">
      The root hash of the tree
    </li>
    <li id="24c4" class="graf graf--li graf-after--li">
      The “branch” (all of the partner hashes going up along the path from the chunk to the root)
    </li>
  </ol>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-171 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-z9x-TsEK0aoZE4fp5wh49Q.png" alt="" width="758" height="942" />
  </p>
  
  <p id="a53b" class="graf graf--p graf-after--figure">
    Anyone reading the proof can verify that the hashing for that branch is consistent all the way up the tree, and therefore that the given chunk is actually at that position in the tree.
  </p>
  
  <p id="becb" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">In summary, the benefit of using a Merkle Patricia tree is that the root node of this structure is cryptographically dependent on the data stored in the tree, and so the hash of the root node can be used as a secure identity for this data. Since the block header includes the root hash of the state, transactions, and receipts trees, any node can validate a small part of state of Ethereum without needing to store the entire state, which can be potentially unbounded in size.</strong>
  </p>
  
  <h3 id="7e2e" class="graf graf--h3 graf-after--p">
    <span class="ez-toc-section" id="Gas_and_payment"></span>Gas and&nbsp;payment<span class="ez-toc-section-end"></span>
  </h3>
  
  <p id="1527" class="graf graf--p graf-after--h3">
    One very important concept in Ethereum is the concept of fees. <strong class="markup--strong markup--p-strong">Every computation that occurs as a result of a transaction on the Ethereum network incurs a fee — there’s no free lunch!</strong> This fee is paid in a denomination called “gas.”
  </p>
  
  <p id="6ae1" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">Gas</strong> is the unit used to measure the fees required for a particular computation. <strong class="markup--strong markup--p-strong">Gas price</strong> is the amount of Ether you are willing to spend on every unit of gas, and is measured in “gwei.” “Wei” is the smallest unit of Ether, where 1⁰¹⁸ Wei represents 1 Ether. One gwei is 1,000,000,000 Wei.
  </p>
  
  <p id="3a74" class="graf graf--p graf-after--p">
    With every transaction, a sender sets a <strong class="markup--strong markup--p-strong">gas limit</strong> and<strong class="markup--strong markup--p-strong"> gas price</strong>. The product of <strong class="markup--strong markup--p-strong">gas price</strong> and <strong class="markup--strong markup--p-strong">gas limit</strong> represents the maximum amount of Wei that the sender is willing to pay for executing a transaction.
  </p>
  
  <p id="0817" class="graf graf--p graf-after--p">
    For example, let’s say the sender sets the gas limit to 50,000 and a gas price to 20 gwei. This implies that the sender is willing to spend at most 50,000 x 20 gwei = 1,000,000,000,000,000 Wei = 0.001 Ether to execute that transaction.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-170 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-xSotR_QvCSKCF5C3APmIpA.png" alt="" width="800" height="237" />
  </p>
  
  <p>
    Remember that the gas limit represents the maximum gas the sender is willing to spend money on. If they have enough Ether in their account balance to cover this maximum, they’re good to go. The sender is refunded for any unused gas at the end of the transaction, exchanged at the original rate.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-167 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-UNCaS12SsPln7DEnRvcONQ.png" alt="" width="800" height="260" />
  </p>
  
  <p>
    In the case that the sender does not provide the necessary gas to execute the transaction, the transaction runs “out of gas” and is considered invalid. In this case, the transaction processing aborts and any state changes that occurred are reversed, such that we end up back at the state of Ethereum prior to the transaction. Additionally, a record of the transaction failing gets recorded, showing what transaction was attempted and where it failed. And since the machine already expended effort to run the calculations before running out of gas, logically, <strong class="markup--strong markup--p-strong">none of the gas is refunded to the sender.</strong>
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-158 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-M_hHF3ML8gx46pIU3kJV6A.png" alt="" width="800" height="219" />
  </p>
  
  <p>
    Where exactly does this gas money go?<strong class="markup--strong markup--p-strong"> All the money spent on gas by the sender is sent to the “beneficiary” address, which is typically the miner’s address.</strong> Since miners are expending the effort to run computations and validate transactions, miners receive the gas fee as a reward.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-146 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-BGZrZkYyRmLfehz3-Djv6w.png" alt="" width="800" height="345" />
  </p>
  
  <p>
    Typically, the higher the gas price the sender is willing to pay, the greater the value the miner derives from the transaction. Thus, the more likely miners will be to select it. In this way, miners are free to choose which transactions they want to validate or ignore. In order to guide senders on what gas price to set, miners have the option of advertising the minimum gas price for which they will execute transactions.
  </p>
  
  <h4 id="bf1d" class="graf graf--h4 graf-after--p">
    <span class="ez-toc-section" id="There_are_fees_for_storage_too"></span>There are fees for storage,&nbsp;too<span class="ez-toc-section-end"></span>
  </h4>
  
  <p id="1732" class="graf graf--p graf-after--h4">
    <strong class="markup--strong markup--p-strong">Not only is gas used to pay for computation steps, it is also used to pay for storage usage. </strong>The total fee for storage is proportional to the smallest multiple of 32 bytes used.
  </p>
  
  <p id="3a41" class="graf graf--p graf-after--p">
    Fees for storage have some nuanced aspects. For example, since increased storage increases the size of the Ethereum state database on <em class="markup--em markup--p-em">all</em> nodes, there’s an incentive to keep the amount of data stored small. For this reason, if a transaction has a step that clears an entry in the storage, the fee for executing that operation of is waived, AND a refund is given for freeing up storage space.
  </p>
  
  <h4 id="a3a9" class="graf graf--h4 graf-after--p">
    <span class="ez-toc-section" id="Whats_the_purpose_of_fees"></span><strong class="markup--strong markup--h4-strong">What’s the purpose of&nbsp;fees?</strong><span class="ez-toc-section-end"></span>
  </h4>
  
  <p id="71be" class="graf graf--p graf-after--h4">
    One important aspect of the way the Ethereum works is that <strong class="markup--strong markup--p-strong">every single operation executed by the network is simultaneously effected by every full node.</strong> However, computational steps on the Ethereum Virtual Machine are very expensive. Therefore, Ethereum smart contracts are best used for simple tasks, like running simple business logic or verifying signatures and other cryptographic objects, rather than more complex uses, like file storage, email, or machine learning, which can put a strain on the network. <strong class="markup--strong markup--p-strong">Imposing fees prevents users from overtaxing the network.</strong>
  </p>
  
  <p id="773f" class="graf graf--p graf-after--p">
    Ethereum is a Turing complete language. (In short, a Turing machine is a machine that can simulate any computer algorithm (for those not familiar with Turing machines, check out <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Turing_machine" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Turing_machine">this</a> and <a class="markup--anchor markup--p-anchor" href="http://mathworld.wolfram.com/TuringMachine.html" target="_blank" rel="nofollow noopener" data-href="http://mathworld.wolfram.com/TuringMachine.html">this</a>). This allows for loops and makes Ethereum susceptible to the <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Halting_problem" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Halting_problem">halting problem</a>, a problem in which you cannot determine whether or not a program will run infinitely. If there were no fees, a malicious actor could easily try to disrupt the network by executing an infinite loop within a transaction, without any repercussions. Thus, fees protect the network from deliberate attacks.
  </p>
  
  <p id="d0d4" class="graf graf--p graf-after--p">
    You might be thinking, “why do we also have to pay for storage?” Well, just like computation, storage on the Ethereum network is a cost that the entire network has to take the burden of.
  </p>
  
  <h3 id="49e9" class="graf graf--h3 graf-after--p">
    <span class="ez-toc-section" id="Transaction_and_messages"></span>Transaction and&nbsp;messages<span class="ez-toc-section-end"></span>
  </h3>
  
  <p id="a505" class="graf graf--p graf-after--h3">
    We noted earlier that Ethereum is a <strong class="markup--strong markup--p-strong">transaction-based state machine.</strong> In other words, transactions occurring between different accounts are what move the global state of Ethereum from one state to the next.
  </p>
  
  <p id="4b06" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">In the most basic sense, a transaction is a cryptographically signed piece of instruction that is generated by an externally owned account, serialized, and then submitted to the blockchain.</strong>
  </p>
  
  <p id="bc39" class="graf graf--p graf-after--p">
    There are two types of transactions: <strong class="markup--strong markup--p-strong">message calls</strong> and <strong class="markup--strong markup--p-strong">contract creations </strong>(i.e. transactions that create new Ethereum contracts).
  </p>
  
  <p>
    All transactions contain the following components, regardless of their type:
  </p>
  
  <ul class="postList">
    <li id="d88d" class="graf graf--li graf-after--p">
      <strong class="markup--strong markup--li-strong">nonce</strong>: a count of the number of transactions sent by the sender.
    </li>
    <li id="c07a" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">gasPrice</strong>: the number of Wei that the sender is willing to pay per unit of gas required to execute the transaction.
    </li>
    <li id="b7da" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">gasLimit</strong>: the maximum amount of gas that the sender is willing to pay for executing this transaction. This amount is set and paid upfront, before any computation is done.
    </li>
    <li id="95c9" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">to</strong>: the address of the recipient. In a contract-creating transaction, the contract account address does not yet exist, and so an empty value is used.
    </li>
    <li id="2fbf" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">value</strong>: the amount of Wei to be transferred from the sender to the recipient. In a contract-creating transaction, this value serves as the starting balance within the newly created contract account.
    </li>
    <li id="5894" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">v, r, s</strong>: used to generate the signature that identifies the sender of the transaction.
    </li>
    <li id="1d23" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">init</strong> (only exists for contract-creating transactions): An EVM code fragment that is used to initialize the new contract account. <strong class="markup--strong markup--li-strong">init</strong> is run only once, and then is discarded. When <strong class="markup--strong markup--li-strong">init</strong> is first run, it returns the body of the account code, which is the piece of code that is permanently associated with the contract account.
    </li>
    <li id="05c2" class="graf graf--li graf-after--li">
      <strong class="markup--strong markup--li-strong">data</strong> (optional field that only exists for message calls): the input data (i.e. parameters) of the message call. For example, if a smart contract serves as a domain registration service, a call to that contract might expect input fields such as the domain and IP address.
    </li>
  </ul>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-141 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-04kmylfdL1_kstojK65asQ.png" alt="" width="436" height="504" />
  </p>
  
  <p>
    We learned in the “<em class="markup--em markup--p-em">Accounts</em>” section that transactions — both message calls and contract-creating transactions — are always initiated by externally owned accounts and submitted to the blockchain. Another way to think about it is that transactions are what bridge the external world to the internal state of Ethereum.
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-148 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-fGnVgckky5q91iFuYKLthA.png" alt="" width="800" height="632" />
  </p>
  
  <p id="bc24" class="graf graf--p graf-after--figure">
    But this doesn’t mean that contracts can’t talk to other contracts. <strong class="markup--strong markup--p-strong">Contracts that exist within the global scope of Ethereum’s state can talk to other contracts within that same scope. The way they do this is via “messages” or “internal transactions” to other contracts. </strong>We can think of messages or internal transactions as being similar to transactions, with the major difference that they are NOT generated by externally owned accounts. Instead, they are generated by contracts. They are virtual objects that, unlike transactions, are not serialized and only exist in the Ethereum execution environment.
  </p>
  
  <p id="a857" class="graf graf--p graf-after--p">
    <strong class="markup--strong markup--p-strong">When one contract sends an internal transaction to another contract, the associated code that exists on the recipient contract account is executed.</strong>
  </p>
  
  <p>
    <img decoding="async" loading="lazy" class="aligncenter wp-image-145 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-8wbH03HO4jL9bhT7Sa9ZCA.png" alt="" width="800" height="307" />
  </p>
</div>

One important thing to note is that internal transactions or messages don’t contain a <strong class="markup--strong markup--p-strong">gasLimit. </strong>This is because the gas limit is determined by the external creator of the original transaction (i.e. some externally owned account). The gas limit that the externally owned account sets must be high enough to carry out the transaction, including any sub-executions that occur as a result of that transaction, such as contract-to-contract messages. If, in the chain of transactions and messages, a particular message execution runs out of gas, then that message’s execution will revert, along with any subsequent messages triggered by the execution. However, the parent execution does not need to revert.

### <span class="ez-toc-section" id="Blocks"></span>Blocks<span class="ez-toc-section-end"></span> {#c4de.graf.graf--h3.graf-after--p}

<p id="1cf9" class="graf graf--p graf-after--h3">
  All transactions are grouped together into “blocks.” A blockchain contains a series of such blocks that are chained together.
</p>

<p id="1376" class="graf graf--p graf-after--p">
  In Ethereum, a block consists of:
</p>

<ul class="postList">
  <li id="47e1" class="graf graf--li graf-after--p">
    the <strong class="markup--strong markup--li-strong">block header</strong>
  </li>
  <li id="6995" class="graf graf--li graf-after--li">
    information about the <strong class="markup--strong markup--li-strong">set of transactions</strong> included in that block
  </li>
  <li id="28e9" class="graf graf--li graf-after--li">
    a <strong class="markup--strong markup--li-strong">set of other block headers for the current block’s ommers</strong>.
  </li>
</ul>

#### <span class="ez-toc-section" id="Ommers_explained"></span>Ommers explained<span class="ez-toc-section-end"></span> {#deea.graf.graf--h4.graf-after--li}

<p id="d78a" class="graf graf--p graf-after--h4">
  What the heck is an “ommer?” An ommer is a block whose parent is equal to the current block’s parent’s parent. Let’s take a quick dive into what ommers are used for and why a block contains the block headers for ommers.
</p>

<p id="7f51" class="graf graf--p graf-after--p">
  Because of the way Ethereum is built, block times are much lower (~15 seconds) than those of other blockchains, like Bitcoin (~10 minutes). This enables faster transaction processing. However, one of the downsides of shorter block times is that more competing block solutions are found by miners. These competing blocks are also referred to as “orphaned blocks” (i.e. mined blocks do not make it into the main chain).
</p>

<p id="0ec8" class="graf graf--p graf-after--p">
  The purpose of ommers is to help reward miners for including these orphaned blocks. The ommers that miners include must be “valid,” meaning within the sixth generation or smaller of the present block. After six children, stale orphaned blocks can no longer be referenced (because including older transactions would complicate things a bit).
</p>

<p id="9a98" class="graf graf--p graf-after--p">
  Ommer blocks receive a smaller reward than a full block. Nonetheless, there’s still some incentive for miners to include these orphaned blocks and reap a reward.
</p>

#### <span class="ez-toc-section" id="Block_header"></span>Block header<span class="ez-toc-section-end"></span> {#5f7a.graf.graf--h4.graf-after--p}

<p id="8911" class="graf graf--p graf-after--h4">
  Let’s get back to blocks for a moment. We mentioned previously that every block has a block “header,” but what exactly is this?
</p>

A block header is a portion of the block consisting of:

<ul class="postList">
  <li id="5ddf" class="graf graf--li graf-after--p">
    <strong class="markup--strong markup--li-strong">parentHash</strong>: a hash of the parent block’s header (this is what makes the block set a “chain”)
  </li>
  <li id="a394" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">ommersHash</strong>: a hash of the current block’s list of ommers
  </li>
  <li id="3c1d" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">beneficiary</strong>: the account address that receives the fees for mining this block
  </li>
  <li id="27b0" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">stateRoot</strong>: the hash of the root node of the state trie (recall how we learned that the state trie is stored in the header and makes it easy for light clients to verify anything about the state)
  </li>
  <li id="a581" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">transactionsRoot</strong>: the hash of the root node of the trie that contains all transactions listed in this block
  </li>
  <li id="fa00" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">receiptsRoot</strong>: the hash of the root node of the trie that contains the receipts of all transactions listed in this block
  </li>
  <li id="84c0" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">logsBloom</strong>: a <a class="markup--anchor markup--li-anchor" href="https://en.wikipedia.org/wiki/Bloom_filter" target="_blank" rel="noopener nofollow" data-href="https://en.wikipedia.org/wiki/Bloom_filter">Bloom filter</a> (data structure) that consists of log information
  </li>
  <li id="e2f0" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">difficulty</strong>: the difficulty level of this block
  </li>
  <li id="909f" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">number</strong>: the count of current block (the genesis block has a block number of zero; the block number increases by 1 for each each subsequent block)
  </li>
  <li id="b8e7" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">gasLimit</strong>: the current gas limit per block
  </li>
  <li id="7cbe" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">gasUsed</strong>: the sum of the total gas used by transactions in this block
  </li>
  <li id="866f" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">timestamp</strong>: the unix timestamp of this block’s inception
  </li>
  <li id="a527" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">extraData</strong>: extra data related to this block
  </li>
  <li id="4b10" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">mixHash</strong>: a hash that, when combined with the nonce, proves that this block has carried out enough computation
  </li>
  <li id="f58d" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">nonce</strong>: a hash that, when combined with the mixHash, proves that this block has carried out enough computation
  </li>
</ul>

<img decoding="async" loading="lazy" class="aligncenter wp-image-142 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-4EQFjXD2-dbiVgVv-8Si8g.png" alt="" width="800" height="553" /> 

<p id="b1b7" class="graf graf--p graf-after--figure">
  Notice how every block header contains three trie structures for:
</p>

<ul class="postList">
  <li id="4cdf" class="graf graf--li graf-after--p">
    state (<strong class="markup--strong markup--li-strong">stateRoot</strong>)
  </li>
  <li id="3212" class="graf graf--li graf-after--li">
    transactions (<strong class="markup--strong markup--li-strong">transactionsRoot</strong>)
  </li>
  <li id="e2ca" class="graf graf--li graf-after--li">
    receipts (<strong class="markup--strong markup--li-strong">receiptsRoot</strong>)
  </li>
</ul>

<p id="fce1" class="graf graf--p graf-after--li">
  These trie structures are nothing but the Merkle Patricia tries we discussed earlier.
</p>

<p id="9ca5" class="graf graf--p graf-after--p">
  Additionally, there are a few terms from the above description that are worth clarifying. Let’s take a look.
</p>

#### <span class="ez-toc-section" id="Logs"></span>Logs<span class="ez-toc-section-end"></span> {#e373.graf.graf--h4.graf-after--p}

<p id="2d13" class="graf graf--p graf-after--h4">
  Ethereum allows for logs to make it possible to track various transactions and messages. A contract can explicitly generate a log by defining “events” that it wants to log.
</p>

<p id="177e" class="graf graf--p graf-after--p">
  A log entry contains:
</p>

<ul class="postList">
  <li id="2996" class="graf graf--li graf-after--p">
    the logger’s account address,
  </li>
  <li id="b8c4" class="graf graf--li graf-after--li">
    a series of topics that represent various events carried out by this transaction, and
  </li>
  <li id="72bd" class="graf graf--li graf-after--li">
    any data associated with these events.
  </li>
</ul>

<p id="788e" class="graf graf--p graf-after--li">
  Logs are stored in a <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Bloom_filter" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Bloom_filter">bloom filter</a>, which stores the endless log data in an efficient manner.
</p>

#### <span class="ez-toc-section" id="Transaction_receipt"></span>Transaction receipt<span class="ez-toc-section-end"></span> {#3919.graf.graf--h4.graf-after--p}

<p id="322a" class="graf graf--p graf-after--h4">
  Logs stored in the header come from the log information contained in the transaction receipt. Just as you receive a receipt when you buy something at a store, Ethereum generates a receipt for every transaction. Like you’d expect, each receipt contains certain information about the transaction. This receipt includes items like:
</p>

<ul class="postList">
  <li id="be1d" class="graf graf--li graf-after--p">
    the block number
  </li>
  <li id="4a4b" class="graf graf--li graf-after--li">
    block hash
  </li>
  <li id="bcdb" class="graf graf--li graf-after--li">
    transaction hash
  </li>
  <li id="11cb" class="graf graf--li graf-after--li">
    gas used by the current transaction
  </li>
  <li id="3998" class="graf graf--li graf-after--li">
    cumulative gas used in the current block after the current transaction has executed
  </li>
  <li id="783b" class="graf graf--li graf-after--li">
    logs created when executing the current transaction
  </li>
  <li id="f4c0" class="graf graf--li graf-after--li">
    ..and so on
  </li>
</ul>

#### <span class="ez-toc-section" id="Block_difficulty"></span>Block difficulty<span class="ez-toc-section-end"></span> {#4363.graf.graf--h4.graf-after--li}

<p id="99d0" class="graf graf--p graf-after--h4">
  The “difficulty” of a block is used to enforce consistency in the time it takes to validate blocks. The genesis block has a difficulty of 131,072, and a special formula is used to calculate the difficulty of every block thereafter. If a certain block is validated more quickly than the previous block, the Ethereum protocol increases that block’s difficulty.
</p>

<p id="7762" class="graf graf--p graf-after--p">
  The difficulty of the block affects the <strong class="markup--strong markup--p-strong">nonce, </strong>which is a hash that must be calculated when mining a block, using the proof-of-work algorithm.
</p>

<p id="8e52" class="graf graf--p graf-after--p">
  The relationship between the block’s <strong class="markup--strong markup--p-strong">difficulty</strong> and <strong class="markup--strong markup--p-strong">nonce</strong> is mathematically formalized as:
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-143 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-5sQvq61s8QBk1xXPKmWu2w.png" alt="" width="162" height="114" /> 

<p id="f043" class="graf graf--p graf-after--figure">
  <em class="markup--em markup--p-em">where </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">Hd</em></strong><em class="markup--em markup--p-em"> is the difficulty.</em>
</p>

<p id="29ac" class="graf graf--p graf-after--p">
  The only way to find a nonce that meets a difficulty threshold is to use the proof-of-work algorithm to enumerate all of the possibilities. The expected time to find a solution is proportional to the difficulty — the higher the difficulty, the harder it becomes to find the nonce, and so the harder it is to validate the block, which in turn increases the time it takes to validate a new block. <strong class="markup--strong markup--p-strong">So, by adjusting the difficulty of a block, the protocol can adjust how long it takes to validate a block.</strong>
</p>

<p id="c87d" class="graf graf--p graf-after--p">
  If, on the other hand, validation time is getting slower, the protocol decreases the difficulty. In this way, the validation time self-adjusts to maintain a constant rate — on average, one block every 15 seconds.
</p>

### <span class="ez-toc-section" id="Transaction_Execution"></span>Transaction Execution<span class="ez-toc-section-end"></span> {#aa78.graf.graf--h3.graf-after--p}

<p id="8390" class="graf graf--p graf-after--h3">
  We’ve come to one of the most complex parts of the Ethereum protocol: the execution of a transaction. Say you send a transaction off into the Ethereum network to be processed. What happens to transition the state of Ethereum to include your transaction?
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-153 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-IxHl8-O1EYt48FpKQB_Sww.png" alt="" width="800" height="432" /> 

<p id="46b0" class="graf graf--p graf-after--figure">
  First, all transactions must meet an initial set of requirements in order to be executed. These include:
</p>

<ul class="postList">
  <li id="1030" class="graf graf--li graf-after--p">
    The transaction must be a properly formatted <strong class="markup--strong markup--li-strong">RLP</strong>. “RLP” stands for “Recursive Length Prefix” and is a data format used to encode nested arrays of binary data. RLP is the format Ethereum uses to serialize objects.
  </li>
  <li id="db7a" class="graf graf--li graf-after--li">
    Valid transaction signature.
  </li>
  <li id="5993" class="graf graf--li graf-after--li">
    Valid transaction nonce. Recall that the nonce of an account is the count of transactions sent from that account. To be valid, a transaction nonce must be equal to the sender account’s nonce.
  </li>
  <li id="c2c6" class="graf graf--li graf-after--li">
    The transaction’s gas limit must be equal to or greater than the <strong class="markup--strong markup--li-strong">intrinsic gas</strong> used by the transaction. The intrinsic gas includes:
  </li>
</ul>

<ol class="postList">
  <li id="1307" class="graf graf--li graf-after--li">
    a predefined cost of 21,000 gas for executing the transaction
  </li>
  <li id="cab4" class="graf graf--li graf-after--li">
    a gas fee for data sent with the transaction (4 gas for every byte of data or code that equals zero, and 68 gas for every non-zero byte of data or code)
  </li>
  <li id="f0ab" class="graf graf--li graf-after--li">
    if the transaction is a contract-creating transaction, an additional 32,000 gas
  </li>
</ol>

<img decoding="async" loading="lazy" class="aligncenter wp-image-168 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-vO9aZkm5gTeEJduOXVQuRg.png" alt="" width="800" height="206" /> 

The sender’s account balance must have enough Ether to cover the <strong class="markup--strong markup--li-strong">“upfront” gas costs</strong> that the sender must pay. The calculation for the upfront gas cost is simple: First, the transaction’s <strong class="markup--strong markup--li-strong">gas limit</strong> is multiplied by the transaction’s <strong class="markup--strong markup--li-strong">gas price </strong>to determine the maximum gas cost. Then, this maximum cost is added to the total value being transferred from the sender to the recipient.

<img decoding="async" loading="lazy" class="aligncenter wp-image-157 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-lX4pdY7owk7LHbC6rz3cow.png" alt="" width="800" height="172" /> 

<p id="ba7a" class="graf graf--p graf-after--figure">
  If the transaction meets all of the above requirements for validity, then we move onto the next step.
</p>

<p id="9e4a" class="graf graf--p graf-after--p">
  First, we deduct the upfront cost of execution from the sender’s balance, and increase the nonce of the sender’s account by 1 to account for the current transaction. At this point, we can calculate the gas remaining as the <strong class="markup--strong markup--p-strong">total gas limit for the transaction minus the intrinsic gas used.</strong>
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-169 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-XKKFOIXYtxqrUHpGRj-gNA.png" alt="" width="800" height="207" /> 

<p id="084e" class="graf graf--p graf-after--figure">
  Next, the transaction starts executing. Throughout the execution of a transaction, Ethereum keeps track of the “substate.” This substate is a way to record information accrued during the transaction that will be needed immediately after the transaction completes. Specifically, it contains:
</p>

<ul class="postList">
  <li id="e995" class="graf graf--li graf-after--p">
    <strong class="markup--strong markup--li-strong">Self-destruct set</strong>: a set of accounts (if any) that will be discarded after the transaction completes.
  </li>
  <li id="d75b" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">Log series</strong>: archived and indexable checkpoints of the virtual machine’s code execution.
  </li>
  <li id="f434" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">Refund balance</strong>: the amount to be refunded to the sender account after the transaction. Remember how we mentioned that storage in Ethereum costs money, and that a sender is refunded for clearing up storage? Ethereum keeps track of this using a refund counter. The refund counter starts at zero and increments every time the contract deletes something in storage.
  </li>
</ul>

<p id="09a7" class="graf graf--p graf-after--li">
  Next, the various computations required by the transaction are processed.
</p>

<p id="dece" class="graf graf--p graf-after--p">
  Once all the steps required by the transaction have been processed, and assuming there is no invalid state, the state is finalized by determining the amount of unused gas to be refunded to the sender. In addition to the unused gas, the sender is also refunded some allowance from the “refund balance” that we described above.
</p>

<p id="7f38" class="graf graf--p graf-after--p">
  Once the sender is refunded:
</p>

<ul class="postList">
  <li id="abe0" class="graf graf--li graf-after--p">
    the Ether for the gas is given to the miner
  </li>
  <li id="e2a5" class="graf graf--li graf-after--li">
    the gas used by the transaction is added to the block gas counter (which keeps track of the total gas used by all transactions in the block, and is useful when validating a block)
  </li>
  <li id="a7b0" class="graf graf--li graf-after--li">
    all accounts in the self-destruct set (if any) are deleted
  </li>
</ul>

<p id="8e77" class="graf graf--p graf-after--li">
  Finally, we’re left with the new state and a set of the logs created by the transaction.
</p>

<p id="c47b" class="graf graf--p graf-after--p">
  Now that we’ve covered the basics of transaction execution, let’s look at some of the differences between contract-creating transactions and message calls.
</p>

#### <span class="ez-toc-section" id="Contract_creation"></span>Contract creation<span class="ez-toc-section-end"></span> {#9352.graf.graf--h4.graf-after--p}

<p id="7b33" class="graf graf--p graf-after--h4">
  Recall that in Ethereum, there are two types of accounts: contract accounts and externally owned accounts. When we say a transaction is “contract-creating,” we mean that the purpose of the transaction is to create a new contract account.
</p>

<p id="713f" class="graf graf--p graf-after--p">
  In order to create a new contract account, we first declare the address of the new account using a special formula. Then we initialize the new account by:
</p>

<ul class="postList">
  <li id="026b" class="graf graf--li graf-after--p">
    Setting the nonce to zero
  </li>
  <li id="0781" class="graf graf--li graf-after--li">
    If the sender sent some amount of Ether as <em class="markup--em markup--li-em">value</em> with the transaction, setting the account balance to that value
  </li>
  <li id="bef5" class="graf graf--li graf-after--li">
    Deducting the value added to this new account’s balance from the sender’s balance
  </li>
  <li id="790e" class="graf graf--li graf-after--li">
    Setting the storage as empty
  </li>
  <li id="81a6" class="graf graf--li graf-after--li">
    Setting the contract’s codeHash as the hash of an empty string
  </li>
</ul>

<p id="f203" class="graf graf--p graf-after--li">
  Once we initialize the account, we can actually create the account, using the <strong class="markup--strong markup--p-strong">init code</strong> sent with the transaction (see the “Transaction and messages” section for a refresher on the init code). What happens during the execution of this init code is varied. Depending on the constructor of the contract, it might update the account’s storage, create other contract accounts, make other message calls, etc.
</p>

<p id="01db" class="graf graf--p graf-after--p">
  As the code to initialize a contract is executed, it uses gas. <strong class="markup--strong markup--p-strong">The transaction is not allowed to use up more gas than the remaining gas. If it does, the execution will hit an out-of-gas (OOG) exception and exit. If the transaction exits due to an out-of-gas exception, then the state is reverted to the point immediately prior to transaction. The sender is <em class="markup--em markup--p-em">not </em>refunded the gas that was spent before running out.</strong>
</p>

<p id="7eb0" class="graf graf--p graf-after--p">
  Boo hoo.
</p>

<p id="f0b9" class="graf graf--p graf-after--p">
  However, if the sender sent any Ether value with the transaction, the Ether value will be refunded even if the contract creation fails. Phew!
</p>

<p id="7ea4" class="graf graf--p graf-after--p">
  If the initialization code executes successfully, a final contract-creation cost is paid. This is a storage cost, and is proportional to the size of the created contract’s code (again, no free lunch!) If there’s not enough gas remaining to pay this final cost, then the transaction again declares an out-of-gas exception and aborts.
</p>

<p id="43f8" class="graf graf--p graf-after--p">
  If all goes well and we make it this far without exceptions, then any remaining unused gas is refunded to the original sender of the transaction, and the altered state is now allowed to persist!
</p>

<p id="9d04" class="graf graf--p graf-after--p">
  Hooray!
</p>

#### <span class="ez-toc-section" id="Message_calls"></span>Message calls<span class="ez-toc-section-end"></span> {#842d.graf.graf--h4.graf-after--p}

<p id="9b02" class="graf graf--p graf-after--h4">
  The execution of a message call is similar to that of a contract creation, with a few differences.
</p>

<p id="10e7" class="graf graf--p graf-after--p">
  A message call execution does not include any init code, since no new accounts are being created. However, it can contain input data, if this data was provided by the transaction sender. Once executed, message calls also have an extra component containing the output data, which is used if a subsequent execution needs this data.
</p>

<p id="c337" class="graf graf--p graf-after--p">
  As is true with contract creation, if a message call execution exits because it runs out of gas or because the transaction is invalid (e.g. stack overflow, invalid jump destination, or invalid instruction), none of the gas used is refunded to the original caller. Instead, all of the remaining unused gas is consumed, and the state is reset to the point immediately prior to balance transfer.
</p>

<p id="a05c" class="graf graf--p graf-after--p">
  Until the most recent update of Ethereum, there was no way to stop or revert the execution of a transaction without having the system consume all the gas you provided. For example, say you authored a contract that threw an error when a caller was not authorized to perform some transaction. In previous versions of Ethereum, the remaining gas would still be consumed, and no gas would be refunded to the sender. <strong class="markup--strong markup--p-strong">But the Byzantium update includes a new “revert” code that allows a contract to stop execution and revert state changes, without consuming the remaining gas, and with the ability to return a reason for the failed transaction.</strong> If a transaction exits due to a revert, then the unused gas is returned to the sender.
</p>

### <span class="ez-toc-section" id="Execution_model"></span>Execution model<span class="ez-toc-section-end"></span> {#9984.graf.graf--h3.graf-after--p}

<p id="2be3" class="graf graf--p graf-after--h3">
  So far, we’ve learned about the series of steps that have to happen for a transaction to execute from start to finish. Now, we’ll look at how the transaction actually executes within the VM.
</p>

<p id="38a2" class="graf graf--p graf-after--p">
  <strong class="markup--strong markup--p-strong">The part of the protocol that actually handles processing the transactions is Ethereum’s own virtual machine, known as the Ethereum Virtual Machine (EVM).</strong>
</p>

<p id="b5c2" class="graf graf--p graf-after--p">
  The EVM is a Turing complete virtual machine, as defined earlier. The only limitation the EVM has that a typical Turing complete machine does not is that the EVM is intrinsically bound by gas. Thus, the total amount of computation that can be done is intrinsically limited by the amount of gas provided.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-164 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-q9jrIqmLZ-wXlXg9tlpu9g.png" alt="" width="324" height="368" /> 

<p id="1c6c" class="graf graf--p graf-after--figure">
  Moreover, the EVM has a stack-based architecture. A <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Stack_machine" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Stack_machine">stack machine</a> is a computer that uses a last-in, first-out stack to hold temporary values.
</p>

<p id="5e18" class="graf graf--p graf-after--p">
  The size of each stack item in the EVM is 256-bit, and the stack has a maximum size of 1024.
</p>

<p id="4fb5" class="graf graf--p graf-after--p">
  The EVM has memory, where items are stored as word-addressed byte arrays. Memory is volatile, meaning it is not permanent.
</p>

<p id="e336" class="graf graf--p graf-after--p">
  The EVM also has storage. Unlike memory, storage is non-volatile and is maintained as part of the system state. The EVM stores program code separately, in a virtual <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Read-only_memory" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Read-only_memory">ROM</a> that can only be accessed via special instructions. In this way, the EVM differs from the typical <a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Von_Neumann_architecture" target="_blank" rel="nofollow noopener" data-href="https://en.wikipedia.org/wiki/Von_Neumann_architecture">von Neumann architecture</a>, in which program code is stored in memory or storage.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-163 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-OH3Yr_UHRRimsKKNpPzPw.png" alt="" width="800" height="472" /> 

<p id="8ffc" class="graf graf--p graf-after--figure">
  The EVM also has its own language: “EVM bytecode.” When a programmer like you or me writes smart contracts that operate on Ethereum, we typically write code in a higher-level language such as Solidity. We can then compile that down to EVM bytecode that the EVM can understand.
</p>

<p id="b962" class="graf graf--p graf-after--p">
  Okay, now on to execution.
</p>

<p id="8270" class="graf graf--p graf-after--p">
  Before executing a particular computation, the processor makes sure that the following information is available and valid:
</p>

<ul class="postList">
  <li id="7751" class="graf graf--li graf-after--p">
    System state
  </li>
  <li id="589a" class="graf graf--li graf-after--li">
    Remaining gas for computation
  </li>
  <li id="6ea0" class="graf graf--li graf-after--li">
    Address of the account that owns the code that is executing
  </li>
  <li id="fa8e" class="graf graf--li graf-after--li">
    Address of the sender of the transaction that originated this execution
  </li>
  <li id="4d18" class="graf graf--li graf-after--li">
    Address of the account that caused the code to execute (could be different from the original sender)
  </li>
  <li id="96ef" class="graf graf--li graf-after--li">
    Gas price of the transaction that originated this execution
  </li>
  <li id="1eac" class="graf graf--li graf-after--li">
    Input data for this execution
  </li>
  <li id="789e" class="graf graf--li graf-after--li">
    Value (in Wei) passed to this account as part of the current execution
  </li>
  <li id="240f" class="graf graf--li graf-after--li">
    Machine code to be executed
  </li>
  <li id="50ee" class="graf graf--li graf-after--li">
    Block header of the current block
  </li>
  <li id="a776" class="graf graf--li graf-after--li">
    Depth of the present message call or contract creation stack
  </li>
</ul>

<p id="ecd6" class="graf graf--p graf-after--li">
  At the start of execution, memory and stack are empty and the program counter is zero.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-201 size-full" src="http://roliu.work/wp-content/uploads/2018/03/07d3bd1498a0453180c4c18b64c0345e.png" alt="" width="1280" height="143" /> 

<p class="graf graf--p graf-after--li">
  The EVM then executes the transaction recursively, computing the <strong class="markup--strong markup--p-strong">system state</strong> and the <strong class="markup--strong markup--p-strong">machine state </strong>for each loop. The system state is simply Ethereum’s global state. The machine state is comprised of:
</p>

<ul class="postList">
  <li id="6fdc" class="graf graf--li graf-after--p">
    gas available
  </li>
  <li id="5e72" class="graf graf--li graf-after--li">
    program counter
  </li>
  <li id="9f9e" class="graf graf--li graf-after--li">
    memory contents
  </li>
  <li id="98b1" class="graf graf--li graf-after--li">
    active number of words in memory
  </li>
  <li id="eddd" class="graf graf--li graf-after--li">
    stack contents.
  </li>
</ul>

<p id="ccbe" class="graf graf--p graf-after--li">
  Stack items are added or removed from the leftmost portion of the series.
</p>

<p id="d399" class="graf graf--p graf-after--p">
  On each cycle, the appropriate gas amount is reduced from the remaining gas, and the program counter increments.
</p>

<p id="b070" class="graf graf--p graf-after--p">
  At the end of each loop, there are three possibilities:
</p>

<ol class="postList">
  <li id="0e1f" class="graf graf--li graf-after--p">
    The machine reaches an exceptional state (e.g. insufficient gas, invalid instructions, insufficient stack items, stack items would overflow above 1024, invalid JUMP/JUMPI destination, etc.) and so must be halted, with any changes discarded
  </li>
  <li id="7a1b" class="graf graf--li graf-after--li">
    The sequence continues to process into the next loop
  </li>
  <li id="28e0" class="graf graf--li graf-after--li">
    The machine reaches a controlled halt (the end of the execution process)
  </li>
</ol>

<p id="b7cc" class="graf graf--p graf-after--li">
  Assuming the execution doesn’t hit an exceptional state and reaches a “controlled” or normal halt, the machine generates the resultant state, the remaining gas after this execution, the accrued substate, and the resultant output.
</p>

<p id="c689" class="graf graf--p graf-after--p">
  Phew. We got through one of the most complex parts of Ethereum. Even if you didn’t fully comprehend this part, that’s okay. You don’t <em class="markup--em markup--p-em">really</em> need to understand the nitty gritty execution details unless you’re working at a very deep level.
</p>

#### <span class="ez-toc-section" id="How_a_block_gets_finalized"></span>How a block gets finalized<span class="ez-toc-section-end"></span> {#7ca6.graf.graf--h4.graf-after--p}

<p id="5940" class="graf graf--p graf-after--h4">
  Finally, let’s look at how a block of many transactions gets finalized.
</p>

<p id="dfcc" class="graf graf--p graf-after--p">
  When we say “finalized,” it can mean two different things, depending on whether the block is new or existing. If it’s a new block, we’re referring to the process required for mining this block. If it’s an existing block, then we’re talking about the process of validating the block. In either case, there are four requirements for a block to be “finalized”:
</p>

<strong class="markup--strong markup--p-strong">1) Validate (or, if mining, determine) ommers</strong>  
Each ommer block within the block header must be a valid header and be within the sixth generation of the present block.

<strong class="markup--strong markup--p-strong">2) Validate (or, if mining, determine) transactions</strong>  
The <strong class="markup--strong markup--p-strong">gasUsed</strong> number on the block must be equal to the cumulative gas used by the transactions listed in the block. (Recall that when executing a transaction, we keep track of the block gas counter, which keeps track of the total gas used by all transactions in the block).

<strong class="markup--strong markup--p-strong">3) Apply rewards (only if mining)</strong>  
The beneficiary address is awarded 5 Ether for mining the block. (Under Ethereum proposal <a class="markup--anchor markup--p-anchor" href="https://github.com/ethereum/EIPs/pull/669" target="_blank" rel="nofollow noopener" data-href="https://github.com/ethereum/EIPs/pull/669">EIP-649</a>, this reward of 5 ETH will soon be reduced to 3 ETH). Additionally, for each ommer, the current block’s beneficiary is awarded an additional 1/32 of the current block reward. Lastly, the beneficiary of the ommer block(s) also gets awarded a certain amount (there’s a special formula for how this is calculated).

<strong class="markup--strong markup--p-strong">4) Verify (or, if mining, compute a valid) state and nonce</strong>  
Ensure that all transactions and resultant state changes are applied, and then define the new block as the state after the block reward has been applied to the final transaction’s resultant state. Verification occurs by checking this final state against the state trie stored in the header.

### <span class="ez-toc-section" id="Mining_proof_of_work"></span>Mining proof of&nbsp;work<span class="ez-toc-section-end"></span> {#2f4d.graf.graf--h3.graf-after--p}

<p id="7a84" class="graf graf--p graf-after--h3">
  The “<em class="markup--em markup--p-em">Blocks</em>” section briefly addressed the concept of block difficulty. The algorithm that gives meaning to block difficulty is called Proof of Work (PoW).
</p>

<p id="5c8c" class="graf graf--p graf-after--p">
  Ethereum’s proof-of-work algorithm is called “<a class="markup--anchor markup--p-anchor" href="https://github.com/ethereum/wiki/wiki/Ethash" target="_blank" rel="nofollow noopener" data-href="https://github.com/ethereum/wiki/wiki/Ethash">Ethash</a>” (previously known as Dagger-Hashimoto).
</p>

<p id="1f74" class="graf graf--p graf-after--p">
  The algorithm is formally defined as:
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-147 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-BJrvs-jGJTtpsOXdseZBLQ.png" alt="" width="374" height="94" /> 

<p id="be1a" class="graf graf--p graf-after--figure">
  <em class="markup--em markup--p-em">where </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">m</em></strong><em class="markup--em markup--p-em"> is the </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">mixHash</em></strong><em class="markup--em markup--p-em">, </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">n</em></strong><em class="markup--em markup--p-em"> is the </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">nonce</em></strong><em class="markup--em markup--p-em">, </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">Hn</em></strong><em class="markup--em markup--p-em"> is the new block’s header (excluding the </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">nonce</em></strong><em class="markup--em markup--p-em"> and </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">mixHash</em></strong><em class="markup--em markup--p-em"> components, which have to be computed), </em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em">Hn</em></strong><em class="markup--em markup--p-em"> is the nonce of the block header, and</em><strong class="markup--strong markup--p-strong"><em class="markup--em markup--p-em"> d</em></strong><em class="markup--em markup--p-em"> is the </em><a class="markup--anchor markup--p-anchor" href="https://en.wikipedia.org/wiki/Directed_acyclic_graph" target="_blank" rel="noopener nofollow" data-href="https://en.wikipedia.org/wiki/Directed_acyclic_graph"><em class="markup--em markup--p-em">DAG</em></a><em class="markup--em markup--p-em">, which is a large data set.</em>
</p>

<p id="4095" class="graf graf--p graf-after--p">
  In the “<em class="markup--em markup--p-em">Blocks</em>” section, we talked about the various items that exist in a block header. Two of those components were called the <strong class="markup--strong markup--p-strong">mixHash</strong> and the <strong class="markup--strong markup--p-strong">nonce</strong>. As you may recall:
</p>

<ul class="postList">
  <li id="9c47" class="graf graf--li graf-after--p">
    <strong class="markup--strong markup--li-strong">mixHash </strong>is a hash that, when combined with the nonce, proves that this block has carried out enough computation
  </li>
  <li id="1e6e" class="graf graf--li graf-after--li">
    <strong class="markup--strong markup--li-strong">nonce </strong>is a hash that, when combined with the mixHash, proves that this block has carried out enough computation
  </li>
</ul>

<p id="324d" class="graf graf--p graf-after--li">
  The PoW function is used to evaluate these two items.
</p>

<p id="c998" class="graf graf--p graf-after--p">
  How exactly the <strong class="markup--strong markup--p-strong">mixHash</strong> and <strong class="markup--strong markup--p-strong">nonce</strong> are calculated using the PoW function is somewhat complex, and something we can delve deeper into in a separate post. But at a high level, it works like this:
</p>

<p id="2718" class="graf graf--p graf-after--p">
  A “seed” is calculated for each block. This seed is different for every “epoch,” where each epoch is 30,000 blocks long. For the first epoch, the seed is the hash of a series of 32 bytes of zeros. For every subsequent epoch, it is the hash of the previous seed hash. Using this seed, a node can calculate a pseudo-random “cache.”
</p>

<p id="5ac4" class="graf graf--p graf-after--p">
  This cache is incredibly useful because it enables the concept of “light nodes,” which we discussed previously in this post. The purpose of light nodes is to afford certain nodes the ability to efficiently verify a transaction without the burden of storing the entire blockchain dataset. A light node can verify the validity of a transaction based solely on this cache, because the cache can regenerate the specific block it needs to verify.
</p>

<p id="30f3" class="graf graf--p graf-after--p">
  Using the cache, a node can generate the DAG “dataset,” where each item in the dataset depends on a small number of pseudo-randomly-selected items from the cache. In order to be a miner, you must generate this full dataset; all full clients and miners store this dataset, and the dataset grows linearly with time.
</p>

<p id="45d9" class="graf graf--p graf-after--p">
  Miners can then take random slices of the dataset and put them through a mathematical function to hash them together into a “<strong class="markup--strong markup--p-strong">mixHash.</strong>” A miner will repeatedly generate a <strong class="markup--strong markup--p-strong">mixHash</strong> until the output is below the desired target <strong class="markup--strong markup--p-strong">nonce</strong>. When the output meets this requirement, this nonce is considered valid and the block can be added to the chain.
</p>

<p id="23fa" class="graf graf--p graf-after--p">
  <strong class="markup--strong markup--p-strong">Mining as a security mechanism</strong><br /> Overall, the purpose of the PoW is to prove, in a cryptographically secure way, that a particular amount of computation has been expended to generate some output (i.e. the <strong class="markup--strong markup--p-strong">nonce</strong>). <strong class="markup--strong markup--p-strong">This is because there is no better way to find a nonce that is below the required threshold other than to enumerate all the possibilities.</strong> The outputs of repeatedly applying the hash function have a uniform distribution, and so we can be assured that, on average, <strong class="markup--strong markup--p-strong">the time needed to find such a nonce depends on the difficulty threshold. </strong>The higher the difficulty, the longer it takes to solve for the nonce. In this way,<strong class="markup--strong markup--p-strong"> the PoW algorithm gives meaning to the concept of difficulty, which is used to enforce blockchain security.</strong>
</p>

<p id="cfa9" class="graf graf--p graf-after--p">
  What do we mean by blockchain security? It’s simple: we want to create a blockchain that EVERYONE trusts. As we discussed previously in this post, if more than one chain existed, users would lose trust, because they would be unable to reasonably determine which chain was the “valid” chain. In order for a group of users to accept the underlying state that is stored on a blockchain, we need a single canonical blockchain that a group of people believes in.
</p>

<p id="4947" class="graf graf--p graf-after--p">
  <strong class="markup--strong markup--p-strong">This is exactly what the PoW algorithm does: it ensures that a particular blockchain will remain canonical into the future, making it incredibly difficult for an attacker to create new blocks that overwrite a certain part of history (e.g. by erasing transactions or creating fake transactions) or maintain a fork.</strong> To have their block validated first, an attacker would need to consistently solve for the nonce faster than anyone else in the network, such that the network believes their chain is the heaviest chain (based on the principles of the GHOST protocol we mentioned earlier). This would be impossible unless the attacker had more than half of the network mining power, a scenario known as the <a class="markup--anchor markup--p-anchor" href="https://en.bitcoin.it/wiki/Majority_attack" target="_blank" rel="nofollow noopener" data-href="https://en.bitcoin.it/wiki/Majority_attack">majority 51% attack</a>.
</p>

<img decoding="async" loading="lazy" class="aligncenter wp-image-166 size-full" src="http://roliu.work/wp-content/uploads/2018/03/1-ULPB9XpVL2Ny8B_Yt75p9w.png" alt="" width="800" height="351" /> 

#### <span class="ez-toc-section" id="Mining_as_a_wealth_distribution_mechanism"></span>Mining as a wealth distribution mechanism<span class="ez-toc-section-end"></span> {#d5d5.graf.graf--h4.graf-after--figure}

<p id="679d" class="graf graf--p graf-after--h4">
  Beyond providing a secure blockchain, PoW is also a way to distribute wealth to those who expend their computation for providing this security. Recall that a miner receives a reward for mining a block, including:
</p>

<ul class="postList">
  <li id="c747" class="graf graf--li graf-after--p">
    a <em class="markup--em markup--li-em">static block reward </em>of 5 ether for the “winning’” block (soon to be <a class="markup--anchor markup--li-anchor" href="https://github.com/ethereum/EIPs/pull/669" target="_blank" rel="noopener nofollow" data-href="https://github.com/ethereum/EIPs/pull/669">changed to 3 ether</a>)
  </li>
  <li id="e30d" class="graf graf--li graf-after--li">
    the cost of gas expended within the block by the transactions included in the block
  </li>
  <li id="3149" class="graf graf--li graf-after--li">
    an extra reward for including ommers as part of the block
  </li>
</ul>

<p id="9787" class="graf graf--p graf-after--li">
  In order to ensure that the use of the PoW consensus mechanism for security and wealth distribution is sustainable in the long run, Ethereum strives to instill these two properties:
</p>

<ul class="postList">
  <li id="1973" class="graf graf--li graf-after--p">
    Make it accessible to as many people as possible. In other words, people shouldn’t need specialized or uncommon hardware to run the algorithm. The purpose of this is to make the wealth distribution model as open as possible so that anyone can provide any amount of compute power in return for Ether.
  </li>
  <li id="44fb" class="graf graf--li graf-after--li">
    Reduce the possibility for any single node (or small set) to make a disproportionate amount of profit. Any node that can make a disproportionate amount of profit means that the node has a large influence on determining the canonical blockchain. This is troublesome because it reduces network security.
  </li>
</ul>

<p id="a01e" class="graf graf--p graf-after--li">
  In the Bitcoin blockchain network, one problem that arises in relation to the above two properties is that the PoW algorithm is a SHA256 hash function. The weakness with this type of function is that it can be solved much more efficiently using specialized hardware, also known as ASICs.
</p>

<p id="0899" class="graf graf--p graf-after--p">
  In order to mitigate this issue, Ethereum has chosen to make its PoW algorithm (<a class="markup--anchor markup--p-anchor" href="https://github.com/ethereum/wiki/wiki/Ethash" target="_blank" rel="nofollow noopener" data-href="https://github.com/ethereum/wiki/wiki/Ethash">Ethhash</a>) sequentially memory-hard. This means that the algorithm is engineered so that calculating the nonce requires a lot of memory AND bandwidth. The large memory requirements make it hard for a computer to use its memory in parallel to discover multiple nonces simultaneously, and the high bandwidth requirements make it difficult for even a super-fast computer to discover multiple nonce simultaneously. This reduces the risk of centralization and creates a more level playing field for the nodes that are doing the verification.
</p>

<p id="b77a" class="graf graf--p graf-after--p">
  One thing to note is that Ethereum is transitioning from a PoW consensus mechanism to something called “proof-of-stake”. This is a beastly topic of its own that we can hopefully explore in a future post.
</p>

### <span class="ez-toc-section" id="Conclusion"></span>Conclusion<span class="ez-toc-section-end"></span> {#af57.graf.graf--h3.graf-after--p}

<p id="a440" class="graf graf--p graf-after--h3">
  …Phew! You made it to the end. I hope?
</p>

<p id="78cf" class="graf graf--p graf-after--p">
  There’s a lot to digest in this post, I know. If it takes you multiple reads to fully understand what’s going on, that’s totally fine. I personally read the Ethereum yellow paper, white paper, and various parts of the code base many times before grokking what was going on.
</p>

<p id="463b" class="graf graf--p graf-after--p">
  Nonetheless, I hope you found this overview helpful. If you find any errors or mistakes, I’d love for you to write a private note or post it directly in the comments. I look at all of ’em, I promise.
</p>

<p id="01aa" class="graf graf--p graf-after--p graf--trailing">
  And remember, I’m human (yep, it’s true) and I make mistakes. I took the time to write this post for the benefit of the community, for free. So please be constructive in your feedback, without unnecessary bashing.
</p>

**[1]** <a class="markup--anchor markup--p-anchor" href="https://github.com/ethereum/yellowpaper" target="_blank" rel="noopener nofollow" data-href="https://github.com/ethereum/yellowpaper"><strong>https://github.com/ethereum/yellowpaper</strong></a>

**The original link：**[**https://medium.com/@preethikasireddy/how-does-ethereum-work-anyway-22d1df506369**][1]

 [1]: https://medium.com/@preethikasireddy/how-does-ethereum-work-anyway-22d1df506369