---
title: 'Building Blockchain in Go. Part 3: Persistence and CLI'
date: 2018-06-09T10:05:37+00:00
categories:
  - 区块链
series:
series_order: 3
summary: 使用Go语言构建区块链系列教程第三部分，实现区块链数据的持久化存储和命令行界面，让区块链数据能够保存到本地数据库。
---

## <span class="ez-toc-section" id="Introduction"></span>Introduction<span class="ez-toc-section-end"></span> {#introduction}

[So][1] [far][2], we’ve built a blockchain with a proof-of-work system, which makes mining possible. Our implementation is getting closer to a fully functional blockchain, but it still lacks some important features. Today will start storing a blockchain in a database, and after that we’ll make a simple command-line interface to perform operations with the blockchain. In its essence, blockchain is a distributed database. We’re going to omit the “distributed” part for now and focus on the “database” part.

## <span class="ez-toc-section" id="Database_Choice"></span>Database Choice<span class="ez-toc-section-end"></span> {#database-choice}

Currently, there’s no database in our implementation; instead, we create blocks every time we run the program and store them in memory. We cannot reuse a blockchain, we cannot share it with others, thus we need to store it on the disk.

Which database do we need? Actually, any of them. In [the original Bitcoin paper][3], nothing is said about using a certain database, so it’s up to a developer what DB to use. [Bitcoin Core][4], which was initially published by Satoshi Nakamoto and which is currently a reference implementation of Bitcoin, uses [LevelDB][5] (although it was introduced to the client only in 2012). And we’ll use…

## <span class="ez-toc-section" id="BoltDB"></span>BoltDB<span class="ez-toc-section-end"></span> {#boltdb}

Because:

  1. It’s simple and minimalistic.
  2. It’s implemented in Go.
  3. It doesn’t require to run a server.
  4. It allows to build the data structure we want.

From the BoltDB’s [README on Github][6]:

Bolt is a pure Go key/value store inspired by Howard Chu’s LMDB project. The goal of the project is to provide a simple, fast, and reliable database for projects that don’t require a full database server such as Postgres or MySQL.

Since Bolt is meant to be used as such a low-level piece of functionality, simplicity is key. The API will be small and only focus on getting values and setting values. That’s it.

Sounds perfect for our needs! Let’s spend a minute reviewing it.

BoltDB is a key/value storage, which means there’re no tables like in SQL RDBMS (MySQL, PostgreSQL, etc.), no rows, no columns. Instead, data is stored as key-value pairs (like in Golang maps). Key-value pairs are stored in buckets, which are intended to group similar pairs (this is similar to tables in RDBMS). Thus, in order to get a value, you need to know a bucket and a key.

One important thing about BoltDB is that there are no data types: keys and values are byte arrays. Since we’ll store Go structs (`Block`, in particular) in it, we’ll need to serialize them, i.e. implement a mechanism of converting a Go struct into a byte array and restoring it back from a byte array. We’ll use [encoding/gob][7] for this, but `JSON`, `XML`, `Protocol Buffers`, etc. can be used as well. We’re using `encoding/gob` because it’s simple and is a part of the standard Go library.

## <span class="ez-toc-section" id="Database_Structure"></span>Database Structure<span class="ez-toc-section-end"></span> {#database-structure}

Before starting implementing persistence logic, we first need to decide how we’ll store data in the DB. And for this, we’ll refer to the way Bitcoin Core does that.

In simple words, Bitcoin Core uses two “buckets” to store data:

  1. `blocks` stores metadata describing all the blocks in a chain.
  2. `chainstate` stores the state of a chain, which is all currently unspent transaction outputs and some metadata.

Also, blocks are stored as separate files on the disk. This is done for a performance purpose: reading a single block won’t require loading all (or some) of them into memory. We won’t implement this.

In `blocks`, the `key -> value` pairs are:

  1. `'b' + 32-byte block hash -> block index record`
  2. `'f' + 4-byte file number -> file information record`
  3. `'l' -> 4-byte file number: the last block file number used`
  4. `'R' -> 1-byte boolean: whether we're in the process of reindexing`
  5. `'F' + 1-byte flag name length + flag name string -> 1 byte boolean: various flags that can be on or off`
  6. `'t' + 32-byte transaction hash -> transaction index record`

In `chainstate`, the `key -> value` pairs are:

  1. `'c' + 32-byte transaction hash -> unspent transaction output record for that transaction`
  2. `'B' -> 32-byte block hash: the block hash up to which the database represents the unspent transaction outputs`

_(Detailed explanation can be found [here][8])_

Since we don’t have transactions yet, we’re going to have only `blocks` bucket. Also, as said above, we will store the whole DB as a single file, without storing blocks in separate files. So we won’t need anything related to file numbers. So these are `key -> value` pairs we’ll use:

  1. `32-byte block-hash -> Block structure (serialized)`
  2. `'l' -> the hash of the last block in a chain`

That’s all we need to know to start implementing the persistence mechanism.

## <span class="ez-toc-section" id="Serialization"></span>Serialization<span class="ez-toc-section-end"></span> {#serialization}

As said before, in BoltDB values can be only of `[]byte` type, and we want to store `Block` structs in the DB. We’ll use [encoding/gob][7] to serialize the structs.

Let’s implement `Serialize` method of `Block` (errors processing is omitted for brevity):

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(b *Block)&lt;/span> &lt;span class="hljs-title">Serialize&lt;/span>&lt;span class="hljs-params">()&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> result bytes.Buffer
	encoder := gob.NewEncoder(&result)

	err := encoder.Encode(b)

	&lt;span class="hljs-keyword">return&lt;/span> result.Bytes()
}
</code></pre>

The piece is straightforward: at first, we declare a buffer that will store serialized data; then we initialize a `gob` encoder and encode the block; the result is returned as a byte array.

Next, we need a deserializing function that will receive a byte array as input and return a `Block`. This won’t be a method but an independent function:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">DeserializeBlock&lt;/span>&lt;span class="hljs-params">(d []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> block Block

	decoder := gob.NewDecoder(bytes.NewReader(d))
	err := decoder.Decode(&block)

	&lt;span class="hljs-keyword">return&lt;/span> &block
}
</code></pre>

And that’s it for the serialization!

## <span class="ez-toc-section" id="Persistence"></span>Persistence<span class="ez-toc-section-end"></span> {#persistence}

Let’s start with the `NewBlockchain` function. Currently, it creates a new instance of `Blockchain` and adds the genesis block to it. What we want it to do is to:

  1. Open a DB file.
  2. Check if there’s a blockchain stored in it.
  3. If there’s a blockchain: 
      1. Create a new `Blockchain` instance.
      2. Set the tip of the `Blockchain` instance to the last block hash stored in the DB.
  4. If there’s no existing blockchain: 
      1. Create the genesis block.
      2. Store in the DB.
      3. Save the genesis block’s hash as the last block hash.
      4. Create a new `Blockchain` instance with its tip pointing at the genesis block.

In code, it looks like this:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewBlockchain&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">Blockchain&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> tip []&lt;span class="hljs-keyword">byte&lt;/span>
	db, err := bolt.Open(dbFile, &lt;span class="hljs-number">0600&lt;/span>, &lt;span class="hljs-literal">nil&lt;/span>)

	err = db.Update(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
		b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))

		&lt;span class="hljs-keyword">if&lt;/span> b == &lt;span class="hljs-literal">nil&lt;/span> {
			genesis := NewGenesisBlock()
			b, err := tx.CreateBucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
			err = b.Put(genesis.Hash, genesis.Serialize())
			err = b.Put([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>), genesis.Hash)
			tip = genesis.Hash
		} &lt;span class="hljs-keyword">else&lt;/span> {
			tip = b.Get([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>))
		}

		&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">nil&lt;/span>
	})

	bc := Blockchain{tip, db}

	&lt;span class="hljs-keyword">return&lt;/span> &bc
}
</code></pre>

Let’s review this piece by piece.

<pre><code class="language-go hljs">db, err := bolt.Open(dbFile, &lt;span class="hljs-number">0600&lt;/span>, &lt;span class="hljs-literal">nil&lt;/span>)
</code></pre>

This is a standard way of opening a BoltDB file. Notice that it won’t return an error if there’s no such file.

<pre><code class="language-go hljs">err = db.Update(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
...
})
</code></pre>

In BoltDB, operations with a database are run within a transaction. And there are two types of transactions: read-only and read-write. Here, we open a read-write transaction (`db.Update(...)`), because we expect to put the genesis block in the DB.

<pre><code class="language-go hljs">b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))

&lt;span class="hljs-keyword">if&lt;/span> b == &lt;span class="hljs-literal">nil&lt;/span> {
	genesis := NewGenesisBlock()
	b, err := tx.CreateBucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
	err = b.Put(genesis.Hash, genesis.Serialize())
	err = b.Put([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>), genesis.Hash)
	tip = genesis.Hash
} &lt;span class="hljs-keyword">else&lt;/span> {
	tip = b.Get([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>))
}
</code></pre>

This is the core of the function. Here, we obtain the bucket storing our blocks: if it exists, we read the `l` key from it; if it doesn’t exist, we generate the genesis block, create the bucket, save the block into it, and update the `l` key storing the last block hash of the chain.

Also, notice the new way of creating a `Blockchain`:

<pre><code class="language-go hljs">bc := Blockchain{tip, db}
</code></pre>

We don’t store all the blocks in it anymore, instead only the tip of the chain is stored. Also, we store a DB connection, because we want to open it once and keep it open while the program is running. Thus, the `Blockchain` structure now looks like this:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> Blockchain &lt;span class="hljs-keyword">struct&lt;/span> {
	tip []&lt;span class="hljs-keyword">byte&lt;/span>
	db  *bolt.DB
}
</code></pre>

Next thing we want to update is the `AddBlock` method: adding blocks to a chain now is not as easy as adding an element to an array. From now on we’ll store blocks in the DB:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">AddBlock&lt;/span>&lt;span class="hljs-params">(data &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> lastHash []&lt;span class="hljs-keyword">byte&lt;/span>

	err := bc.db.View(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
		b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
		lastHash = b.Get([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>))

		&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">nil&lt;/span>
	})

	newBlock := NewBlock(data, lastHash)

	err = bc.db.Update(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
		b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
		err := b.Put(newBlock.Hash, newBlock.Serialize())
		err = b.Put([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>), newBlock.Hash)
		bc.tip = newBlock.Hash

		&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">nil&lt;/span>
	})
}
</code></pre>

Let’s review this piece by piece:

<pre><code class="language-go hljs">err := bc.db.View(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
	b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
	lastHash = b.Get([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>))

	&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">nil&lt;/span>
})
</code></pre>

This is the other (read-only) type of BoltDB transactions. Here we get the last block hash from the DB to use it to mine a new block hash.

<pre><code class="language-go hljs">newBlock := NewBlock(data, lastHash)
b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
err := b.Put(newBlock.Hash, newBlock.Serialize())
err = b.Put([]&lt;span class="hljs-keyword">byte&lt;/span>(&lt;span class="hljs-string">"l"&lt;/span>), newBlock.Hash)
bc.tip = newBlock.Hash
</code></pre>

After mining a new block, we save its serialized representation into the DB and update the `l` key, which now stores the new block’s hash.

Done! It wasn’t hard, was it?

## <span class="ez-toc-section" id="Inspecting_Blockchain"></span>Inspecting Blockchain<span class="ez-toc-section-end"></span> {#inspecting-blockchain}

All new blocks are now saved in a database, so we can reopen a blockchain and add a new block to it. But after implementing this, we lost a nice feature: we cannot print out blockchain blocks anymore because we don’t store blocks in an array any longer. Let’s fix this flaw!

BoltDB allows to iterate over all the keys in a bucket, but the keys are stored in byte-sorted order, and we want blocks to be printed in the order they take in a blockchain. Also, because we don’t want to load all the blocks into memory (our blockchain DB could be huge!.. or let’s just pretend it could), we’ll read them one by one. For this purpose, we’ll need a blockchain iterator:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> BlockchainIterator &lt;span class="hljs-keyword">struct&lt;/span> {
	currentHash []&lt;span class="hljs-keyword">byte&lt;/span>
	db          *bolt.DB
}
</code></pre>

An iterator will be created each time we want to iterate over blocks in a blockchain and it’ll store the block hash of the current iteration and a connection to a DB. Because of the latter, an iterator is logically attached to a blockchain (it’s a `Blockchain` instance that stores a DB connection) and, thus, is created in a `Blockchain` method:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">Iterator&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">BlockchainIterator&lt;/span>&lt;/span> {
	bci := &BlockchainIterator{bc.tip, bc.db}

	&lt;span class="hljs-keyword">return&lt;/span> bci
}
</code></pre>

Notice that an iterator initially points at the tip of a blockchain, thus blocks will be obtained from top to bottom, from newest to oldest. In fact, **choosing a tip means “voting” for a blockchain**. A blockchain can have multiple branches, and it’s the longest of them that’s considered main. After getting a tip (it can be any block in the blockchain) we can reconstruct the whole blockchain and find its length and the work required to build it. This fact also means that a tip is a kind of an identifier of a blockchain.

`BlockchainIterator` will do only one thing: it’ll return the next block from a blockchain.

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(i *BlockchainIterator)&lt;/span> &lt;span class="hljs-title">Next&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> block *Block

	err := i.db.View(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
		b := tx.Bucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
		encodedBlock := b.Get(i.currentHash)
		block = DeserializeBlock(encodedBlock)

		&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">nil&lt;/span>
	})

	i.currentHash = block.PrevBlockHash

	&lt;span class="hljs-keyword">return&lt;/span> block
}
</code></pre>

That’s it for the DB part!

## <span class="ez-toc-section" id="CLI"></span>CLI<span class="ez-toc-section-end"></span> {#cli}

Until now our implementation hasn’t provided any interface to interact with the program: we’ve simply executed `NewBlockchain`, `bc.AddBlock` in the `main` function. Time to improve this! We want to have these commands:

<pre><code class="hljs css">&lt;span class="hljs-selector-tag">blockchain_go&lt;/span> &lt;span class="hljs-selector-tag">addblock&lt;/span> "&lt;span class="hljs-selector-tag">Pay&lt;/span> 0&lt;span class="hljs-selector-class">.031337&lt;/span> &lt;span class="hljs-selector-tag">for&lt;/span> &lt;span class="hljs-selector-tag">a&lt;/span> &lt;span class="hljs-selector-tag">coffee&lt;/span>"
&lt;span class="hljs-selector-tag">blockchain_go&lt;/span> &lt;span class="hljs-selector-tag">printchain&lt;/span>
</code></pre>

All command-line related operations will be processed by the `CLI` struct:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> CLI &lt;span class="hljs-keyword">struct&lt;/span> {
	bc *Blockchain
}
</code></pre>

Its “entrypoint” is the `Run` function:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(cli *CLI)&lt;/span> &lt;span class="hljs-title">Run&lt;/span>&lt;span class="hljs-params">()&lt;/span>&lt;/span> {
	cli.validateArgs()

	addBlockCmd := flag.NewFlagSet(&lt;span class="hljs-string">"addblock"&lt;/span>, flag.ExitOnError)
	printChainCmd := flag.NewFlagSet(&lt;span class="hljs-string">"printchain"&lt;/span>, flag.ExitOnError)

	addBlockData := addBlockCmd.String(&lt;span class="hljs-string">"data"&lt;/span>, &lt;span class="hljs-string">""&lt;/span>, &lt;span class="hljs-string">"Block data"&lt;/span>)

	&lt;span class="hljs-keyword">switch&lt;/span> os.Args[&lt;span class="hljs-number">1&lt;/span>] {
	&lt;span class="hljs-keyword">case&lt;/span> &lt;span class="hljs-string">"addblock"&lt;/span>:
		err := addBlockCmd.Parse(os.Args[&lt;span class="hljs-number">2&lt;/span>:])
	&lt;span class="hljs-keyword">case&lt;/span> &lt;span class="hljs-string">"printchain"&lt;/span>:
		err := printChainCmd.Parse(os.Args[&lt;span class="hljs-number">2&lt;/span>:])
	&lt;span class="hljs-keyword">default&lt;/span>:
		cli.printUsage()
		os.Exit(&lt;span class="hljs-number">1&lt;/span>)
	}

	&lt;span class="hljs-keyword">if&lt;/span> addBlockCmd.Parsed() {
		&lt;span class="hljs-keyword">if&lt;/span> *addBlockData == &lt;span class="hljs-string">""&lt;/span> {
			addBlockCmd.Usage()
			os.Exit(&lt;span class="hljs-number">1&lt;/span>)
		}
		cli.addBlock(*addBlockData)
	}

	&lt;span class="hljs-keyword">if&lt;/span> printChainCmd.Parsed() {
		cli.printChain()
	}
}
</code></pre>

We’re using the standard [flag][9] package to parse command-line arguments.

<pre><code class="language-go hljs">addBlockCmd := flag.NewFlagSet(&lt;span class="hljs-string">"addblock"&lt;/span>, flag.ExitOnError)
printChainCmd := flag.NewFlagSet(&lt;span class="hljs-string">"printchain"&lt;/span>, flag.ExitOnError)
addBlockData := addBlockCmd.String(&lt;span class="hljs-string">"data"&lt;/span>, &lt;span class="hljs-string">""&lt;/span>, &lt;span class="hljs-string">"Block data"&lt;/span>)
</code></pre>

First, we create two subcommands, `addblock` and `printchain`, then we add `-data` flag to the former. `printchain` won’t have any flags.

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">switch&lt;/span> os.Args[&lt;span class="hljs-number">1&lt;/span>] {
&lt;span class="hljs-keyword">case&lt;/span> &lt;span class="hljs-string">"addblock"&lt;/span>:
	err := addBlockCmd.Parse(os.Args[&lt;span class="hljs-number">2&lt;/span>:])
&lt;span class="hljs-keyword">case&lt;/span> &lt;span class="hljs-string">"printchain"&lt;/span>:
	err := printChainCmd.Parse(os.Args[&lt;span class="hljs-number">2&lt;/span>:])
&lt;span class="hljs-keyword">default&lt;/span>:
	cli.printUsage()
	os.Exit(&lt;span class="hljs-number">1&lt;/span>)
}
</code></pre>

Next we check the command provided by user and parse related `flag` subcommand.

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">if&lt;/span> addBlockCmd.Parsed() {
	&lt;span class="hljs-keyword">if&lt;/span> *addBlockData == &lt;span class="hljs-string">""&lt;/span> {
		addBlockCmd.Usage()
		os.Exit(&lt;span class="hljs-number">1&lt;/span>)
	}
	cli.addBlock(*addBlockData)
}

&lt;span class="hljs-keyword">if&lt;/span> printChainCmd.Parsed() {
	cli.printChain()
}
</code></pre>

Next we check which of the subcommands were parsed and run related functions.

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(cli *CLI)&lt;/span> &lt;span class="hljs-title">addBlock&lt;/span>&lt;span class="hljs-params">(data &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span>&lt;/span> {
	cli.bc.AddBlock(data)
	fmt.Println(&lt;span class="hljs-string">"Success!"&lt;/span>)
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(cli *CLI)&lt;/span> &lt;span class="hljs-title">printChain&lt;/span>&lt;span class="hljs-params">()&lt;/span>&lt;/span> {
	bci := cli.bc.Iterator()

	&lt;span class="hljs-keyword">for&lt;/span> {
		block := bci.Next()

		fmt.Printf(&lt;span class="hljs-string">"Prev. hash: %x\n"&lt;/span>, block.PrevBlockHash)
		fmt.Printf(&lt;span class="hljs-string">"Data: %s\n"&lt;/span>, block.Data)
		fmt.Printf(&lt;span class="hljs-string">"Hash: %x\n"&lt;/span>, block.Hash)
		pow := NewProofOfWork(block)
		fmt.Printf(&lt;span class="hljs-string">"PoW: %s\n"&lt;/span>, strconv.FormatBool(pow.Validate()))
		fmt.Println()

		&lt;span class="hljs-keyword">if&lt;/span> &lt;span class="hljs-built_in">len&lt;/span>(block.PrevBlockHash) == &lt;span class="hljs-number">0&lt;/span> {
			&lt;span class="hljs-keyword">break&lt;/span>
		}
	}
}
</code></pre>

This piece is very similar to the one we had before. The only difference is that we’re now using a `BlockchainIterator` to iterate over blocks in a blockchain.

Also let’s not forget to modify the `main` function accordingly:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">main&lt;/span>&lt;span class="hljs-params">()&lt;/span>&lt;/span> {
	bc := NewBlockchain()
	&lt;span class="hljs-keyword">defer&lt;/span> bc.db.Close()

	cli := CLI{bc}
	cli.Run()
}
</code></pre>

Note that a new `Blockchain` is created no matter what command-line arguments are provided.

And that’s it! Let’s check that everything works as expected:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go printchain&lt;/span>
No existing blockchain found. Creating a new one...
Mining the block containing "Genesis Block"
000000edc4a82659cebf087adee1ea353bd57fcd59927662cd5ff1c4f618109b

Prev. hash:
Data: Genesis Block
Hash: 000000edc4a82659cebf087adee1ea353bd57fcd59927662cd5ff1c4f618109b
PoW: true
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go addblock -data &lt;span class="hljs-string">"Send 1 BTC to Ivan"&lt;/span>&lt;/span>
Mining the block containing "Send 1 BTC to Ivan"
000000d7b0c76e1001cdc1fc866b95a481d23f3027d86901eaeb77ae6d002b13

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go addblock -data &lt;span class="hljs-string">"Pay 0.31337 BTC for a coffee"&lt;/span>&lt;/span>
Mining the block containing "Pay 0.31337 BTC for a coffee"
000000aa0748da7367dec6b9de5027f4fae0963df89ff39d8f20fd7299307148

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go printchain&lt;/span>
Prev. hash: 000000d7b0c76e1001cdc1fc866b95a481d23f3027d86901eaeb77ae6d002b13
Data: Pay 0.31337 BTC for a coffee
Hash: 000000aa0748da7367dec6b9de5027f4fae0963df89ff39d8f20fd7299307148
PoW: true

Prev. hash: 000000edc4a82659cebf087adee1ea353bd57fcd59927662cd5ff1c4f618109b
Data: Send 1 BTC to Ivan
Hash: 000000d7b0c76e1001cdc1fc866b95a481d23f3027d86901eaeb77ae6d002b13
PoW: true

Prev. hash:
Data: Genesis Block
Hash: 000000edc4a82659cebf087adee1ea353bd57fcd59927662cd5ff1c4f618109b
PoW: true
</code></pre>

_(sound of a beer can opening)_

## <span class="ez-toc-section" id="Conclusion"></span>Conclusion<span class="ez-toc-section-end"></span> {#conclusion}

Next time we’ll implement addresses, wallets, and (probably) transactions. So stay tuned!

## <span class="ez-toc-section" id="Links"></span>Links<span class="ez-toc-section-end"></span> {#links}

  1. [Full source codes][10]
  2. [Bitcoin Core Data Storage][8]
  3. [boltdb][6]
  4. [encoding/gob][7]
  5. [flag][9]

The original link：<https://jeiwan.cc/posts/building-blockchain-in-go-part-3/>

Chinese translations: [by liuchengxu][11], [by zhangli1][12].

 [1]: https://jeiwan.cc/posts/building-blockchain-in-go-part-1/
 [2]: https://jeiwan.cc/posts/building-blockchain-in-go-part-2/
 [3]: https://bitcoin.org/bitcoin.pdf
 [4]: https://github.com/bitcoin/bitcoin
 [5]: https://github.com/google/leveldb
 [6]: https://github.com/boltdb/bolt
 [7]: https://golang.org/pkg/encoding/gob/
 [8]: https://en.bitcoin.it/wiki/Bitcoin_Core_0.11_(ch_2):_Data_Storage
 [9]: https://golang.org/pkg/flag/
 [10]: https://github.com/Jeiwan/blockchain_go/tree/part_3
 [11]: https://github.com/liuchengxu/blockchain-tutorial/blob/master/content/part-3/persistence-and-cli.md
 [12]: https://zhangli1.gitbooks.io/dummies-for-blockchain/content/