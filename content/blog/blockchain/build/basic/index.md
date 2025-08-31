---
title: 'Building Blockchain in Go. Part 1: Basic Prototype'
date: 2018-06-07T07:03:36+00:00
categories:
  - 区块链
series:
series_order: 1
---

## <span class="ez-toc-section" id="Introduction"></span>Introduction<span class="ez-toc-section-end"></span> {#introduction}

Blockchain is one of the most revolutionary technologies of the 21st century, which is still maturing and which potential is not fully realized yet. In its essence, blockchain is just a distributed database of records. But what makes it unique is that it’s not a private database, but a public one, i.e. everyone who uses it has a full or partial copy of it. And a new record can be added only with a consent of other keepers of the database. Also, it’s blockchain that made cryptocurrencies and smart contracts possible.

In this series of articles we’ll build a simplified cryptocurrency that’s based on a simple blockchain implementation.

## <span class="ez-toc-section" id="Block"></span>Block<span class="ez-toc-section-end"></span> {#block}

Let’s start with the “block” part of “blockchain”. In blockchain it’s blocks that store valuable information. For example, bitcoin blocks store transactions, the essence of any cryptocurrency. Besides this, a block contains some technical information, like its version, current timestamp and the hash of the previous block.  
In this article we’re not going to implement the block as it’s described in blockchain or Bitcoin specifications, instead we’ll use a simplified version of it, which contains only significant information. Here’s what it looks like:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> Block &lt;span class="hljs-keyword">struct&lt;/span> {
	Timestamp     &lt;span class="hljs-keyword">int64&lt;/span>
	Data          []&lt;span class="hljs-keyword">byte&lt;/span>
	PrevBlockHash []&lt;span class="hljs-keyword">byte&lt;/span>
	Hash          []&lt;span class="hljs-keyword">byte&lt;/span>
}
</code></pre>

`Timestamp` is the current timestamp (when the block is created), `Data` is the actual valuable information containing in the block, `PrevBlockHash` stores the hash of the previous block, and `Hash` is the hash of the block. In Bitcoint specification `Timestamp`, `PrevBlockHash`, and `Hash` are block headers, which form a separate data structure, and transactions (`Data` in our case) is a separate data structure. So we’re mixing them here for simplicity.

So how do we calculate the hashes? The way hashes are calculates is very important feature of blockchain, and it’s this feature that makes blockchain secure. The thing is that calculating a hash is a computationally difficult operation, it takes some time even on fast computers (that’s why people buy powerful GPUs to mine Bitcoin). This is an intentional architectural design, which makes adding new blocks difficult, thus preventing their modification after they’re added. We’ll discuss and implement this mechanism in a future article.

For now, we’ll just take block fields, concatenate them, and calculate a SHA-256 hash on the concatenated combination. Let’s do this in `SetHash` method:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(b *Block)&lt;/span> &lt;span class="hljs-title">SetHash&lt;/span>&lt;span class="hljs-params">()&lt;/span>&lt;/span> {
	timestamp := []&lt;span class="hljs-keyword">byte&lt;/span>(strconv.FormatInt(b.Timestamp, &lt;span class="hljs-number">10&lt;/span>))
	headers := bytes.Join([][]&lt;span class="hljs-keyword">byte&lt;/span>{b.PrevBlockHash, b.Data, timestamp}, []&lt;span class="hljs-keyword">byte&lt;/span>{})
	hash := sha256.Sum256(headers)

	b.Hash = hash[:]
}
</code></pre>

Next, following a Golang convention, we’ll implement a function that’ll simplify the creation of a block:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewBlock&lt;/span>&lt;span class="hljs-params">(data &lt;span class="hljs-keyword">string&lt;/span>, prevBlockHash []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	block := &Block{time.Now().Unix(), []&lt;span class="hljs-keyword">byte&lt;/span>(data), prevBlockHash, []&lt;span class="hljs-keyword">byte&lt;/span>{}}
	block.SetHash()
	&lt;span class="hljs-keyword">return&lt;/span> block
}
</code></pre>

And that’s it for the block!

## <span class="ez-toc-section" id="Blockchain"></span>Blockchain<span class="ez-toc-section-end"></span> {#blockchain}

Now let’s implement a blockchain. In its essence blockchain is just a database with certain structure: it’s an ordered, back-linked list. Which means that blocks are stored in the insertion order and that each block is linked to the previous one. This structure allows to quickly get the latest block in a chain and to (efficiently) get a block by its hash.

In Golang this structure can be implemented by using an array and a map: the array would keep ordered hashes (arrays are ordered in Go), and the map would keep `hash → block` pairs (maps are unordered). But for our blockchain prototype we’ll just use an array, because we don’t need to get blocks by their hash for now.

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> Blockchain &lt;span class="hljs-keyword">struct&lt;/span> {
	blocks []*Block
}
</code></pre>

This is our first blockchain! I’ve never thought it would be so easy.

Now let’s make it possible to add blocks to it:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">AddBlock&lt;/span>&lt;span class="hljs-params">(data &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span>&lt;/span> {
	prevBlock := bc.blocks[&lt;span class="hljs-built_in">len&lt;/span>(bc.blocks)&lt;span class="hljs-number">-1&lt;/span>]
	newBlock := NewBlock(data, prevBlock.Hash)
	bc.blocks = &lt;span class="hljs-built_in">append&lt;/span>(bc.blocks, newBlock)
}
</code></pre>

That’s it! Or not?..

To add a new block we need an existing block, but there’re not blocks in our blockchain! So, in any blockchain, there must be at least one block, and such block, the first in the chain, is called **genesis block**. Let’s implement a method that creates such a block:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewGenesisBlock&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> NewBlock(&lt;span class="hljs-string">"Genesis Block"&lt;/span>, []&lt;span class="hljs-keyword">byte&lt;/span>{})
}
</code></pre>

Now, we can implement a function that creates a blockchain with the genesis block:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewBlockchain&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">Blockchain&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> &Blockchain{[]*Block{NewGenesisBlock()}}
}
</code></pre>

Let’s check that the blockchain works correctly:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">main&lt;/span>&lt;span class="hljs-params">()&lt;/span>&lt;/span> {
	bc := NewBlockchain()

	bc.AddBlock(&lt;span class="hljs-string">"Send 1 BTC to Ivan"&lt;/span>)
	bc.AddBlock(&lt;span class="hljs-string">"Send 2 more BTC to Ivan"&lt;/span>)

	&lt;span class="hljs-keyword">for&lt;/span> _, block := &lt;span class="hljs-keyword">range&lt;/span> bc.blocks {
		fmt.Printf(&lt;span class="hljs-string">"Prev. hash: %x\n"&lt;/span>, block.PrevBlockHash)
		fmt.Printf(&lt;span class="hljs-string">"Data: %s\n"&lt;/span>, block.Data)
		fmt.Printf(&lt;span class="hljs-string">"Hash: %x\n"&lt;/span>, block.Hash)
		fmt.Println()
	}
}
</code></pre>

Output:

<pre><code class="hljs bash">Prev. &lt;span class="hljs-built_in">hash&lt;/span>:
Data: Genesis Block
Hash: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168

Prev. &lt;span class="hljs-built_in">hash&lt;/span>: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168
Data: Send 1 BTC to Ivan
Hash: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1

Prev. &lt;span class="hljs-built_in">hash&lt;/span>: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1
Data: Send 2 more BTC to Ivan
Hash: 561237522bb7fcfbccbc6fe0e98bbbde7427ffe01c6fb223f7562288ca2295d1
</code></pre>

That’s it!

## <span class="ez-toc-section" id="Conclusion"></span>Conclusion<span class="ez-toc-section-end"></span> {#conclusion}

We built a very simple blockchain prototype: it’s just an array of blocks, with each block having a connection to the previous one. The actual blockchain is much more complex though. In our blockchain adding new blocks is easy and fast, but in real blockchain adding new blocks requires some work: one has to perform some heavy computations before getting a permission to add block (this mechanism is called Proof-of-Work). Also, blockchain is a distributed database that has no single decision maker. Thus, a new block must be confirmed and approved by other participants of the network (this mechanism is called consensus). And there’re no transactions in our blockchain yet!

In future articles we’ll cover each of these features.

Links:

  1. Full source codes: [https://github.com/Jeiwan/blockchain\_go/tree/part\_1][1]
  2. Block hashing algorithm: [https://en.bitcoin.it/wiki/Block\_hashing\_algorithm][2]

The original link：<https://jeiwan.cc/posts/building-blockchain-in-go-part-1/>

Chinese translations: [by liuchengxu][3], [by zhangli1][4].

 [1]: https://github.com/Jeiwan/blockchain_go/tree/part_1
 [2]: https://en.bitcoin.it/wiki/Block_hashing_algorithm
 [3]: https://github.com/liuchengxu/blockchain-tutorial/blob/master/content/part-1/basic-prototype.md
 [4]: https://zhangli1.gitbooks.io/dummies-for-blockchain/content/