---
title: 'Building Blockchain in Go. Part 4: Transactions 1'
date: 2018-06-25T04:30:34+00:00
categories:
  - 区块链
series:
series_order: 4
---

## <span class="ez-toc-section" id="Introduction"></span>Introduction<span class="ez-toc-section-end"></span> {#introduction}

Transactions are the heart of Bitcoin and the only purpose of blockchain is to store transactions in a secure and reliable way, so no one could modify them after they are created. Today we’re starting implementing transactions. But because this is quite a big topic, I’ll split it into two parts: in this part, we’ll implement the general mechanism of transactions and in the second part we’ll work through details.

Also, since code changes are massive, it makes no sense describing all of them here. You can see all the changes [here][1].

## <span class="ez-toc-section" id="There_is_no_spoon"></span>There is no spoon<span class="ez-toc-section-end"></span> {#there-is-no-spoon}

If you’ve ever developed a web application, in order to implement payments you would likely to create these tables in a DB: `accounts` and `transactions`. An account would store information about a user, including their personal information and balance, and a transaction would store information about money transferring from one account to another. In Bitcoin, payments are realized in completely different way. There are:

  1. No accounts.
  2. No balances.
  3. No addresses.
  4. No coins.
  5. No senders and receivers.

Since blockchain is a public and open database, we don’t want to store sensitive information about wallet owners. Coins are not collected in accounts. Transactions do not transfer money from one address to another. There’s no field or attribute that holds account balance. There are only transactions. But what’s inside a transaction?

## <span class="ez-toc-section" id="Bitcoin_Transaction"></span>Bitcoin Transaction<span class="ez-toc-section-end"></span> {#bitcoin-transaction}

A transaction is a combination of inputs and outputs:

<pre><code class="language-go">type Transaction struct {
	ID   []byte
	Vin  []TXInput
	Vout []TXOutput
}
</code></pre>

Inputs of a new transaction reference outputs of a previous transaction (there’s an exception though, which we’ll discuss later). Outputs are where coins are actually stored. The following diagram demonstrates the interconnection of transactions:

<img decoding="async" loading="lazy" class="aligncenter wp-image-279 size-full" src="http://roliu.work/wp-content/uploads/2018/06/transactions-diagram.png" alt="" width="825" height="425" /> 

Notice that:

  1. There are outputs that are not linked to inputs.
  2. In one transaction, inputs can reference outputs from multiple transactions.
  3. An input must reference an output.

Throughout this article, we’ll use words like “money”, “coins”, “spend”, “send”, “account”, etc. But there are no such concepts in Bitcoin. Transactions just lock values with a script, which can be unlocked only by the one who locked them.

## <span class="ez-toc-section" id="Transaction_Outputs"></span>Transaction Outputs<span class="ez-toc-section-end"></span> {#transaction-outputs}

Let’s start with outputs first:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> TXOutput &lt;span class="hljs-keyword">struct&lt;/span> {
	Value        &lt;span class="hljs-keyword">int&lt;/span>
	ScriptPubKey &lt;span class="hljs-keyword">string&lt;/span>
}
</code></pre>

Actually, it’s outputs that store “coins” (notice the `Value` field above). And storing means locking them with a puzzle, which is stored in the `ScriptPubKey`. Internally, Bitcoin uses a scripting language called _Script_, that is used to define outputs locking and unlocking logic. The language is quite primitive (this is made intentionally, to avoid possible hacks and misuses), but we won’t discuss it in details. You can find a detailed explanation of it [here][2].

> In Bitcoin, the _value_ field stores the number of _satoshis_, not the number of BTC. A _satoshi_ is a hundred millionth of a bitcoin (0.00000001 BTC), thus this is the smallest unit of currency in Bitcoin (like a cent).

Since we don’t have addresses implemented, we’ll avoid the whole scripting related logic for now. `ScriptPubKey` will store an arbitrary string (user defined wallet address).

> By the way, having such scripting language means that Bitcoin can be used as a smart-contract platform as well.

One important thing about outputs is that they are **indivisible**, which means that you cannot reference a part of its value. When an output is referenced in a new transaction, it’s spent as a whole. And if its value is greater than required, a change is generated and sent back to the sender. This is similar to a real world situation when you pay, say, a $5 banknote for something that costs $1 and get a change of $4.

## <span class="ez-toc-section" id="Transaction_Inputs"></span>Transaction Inputs<span class="ez-toc-section-end"></span> {#transaction-inputs}

And here’s the input:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> TXInput &lt;span class="hljs-keyword">struct&lt;/span> {
	Txid      []&lt;span class="hljs-keyword">byte&lt;/span>
	Vout      &lt;span class="hljs-keyword">int&lt;/span>
	ScriptSig &lt;span class="hljs-keyword">string&lt;/span>
}
</code></pre>

As mentioned earlier, an input references a previous output: `Txid` stores the ID of such transaction, and `Vout` stores an index of an output in the transaction. `ScriptSig` is a script which provides data to be used in an output’s `ScriptPubKey`. If the data is correct, the output can be unlocked, and its value can be used to generate new outputs; if it’s not correct, the output cannot be referenced in the input. This is the mechanism that guarantees that users cannot spend coins belonging to other people.

Again, since we don’t have addresses implemented yet, `ScriptSig` will store just an arbitrary user defined wallet address. We’ll implement public keys and signatures checking in the next article.

Let’s sum it up. Outputs are where “coins” are stored. Each output comes with an unlocking script, which determines the logic of unlocking the output. Every new transaction must have at least one input and output. An input references an output from a previous transaction and provides data (the `ScriptSig` field) that is used in the output’s unlocking script to unlock it and use its value to create new outputs.

But what came first: inputs or outputs?

## <span class="ez-toc-section" id="The_egg"></span>The egg<span class="ez-toc-section-end"></span> {#the-egg}

In Bitcoin, it’s the egg that came before the chicken. The inputs-referencing-outputs logic is the classical “chicken or the egg” situation: inputs produce outputs and outputs make inputs possible. And in Bitcoin, outputs come before inputs.

When a miner starts mining a block, it adds a **coinbase transaction** to it. A coinbase transaction is a special type of transactions, which doesn’t require previously existing outputs. It creates outputs (i.e., “coins”) out of nowhere. The egg without a chicken. This is the reward miners get for mining new blocks.

As you know, there’s the genesis block in the beginning of a blockchain. It’s this block that generates the very first output in the blockchain. And no previous outputs are required since there are no previous transactions and no such outputs.

Let’s create a coinbase transaction:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewCoinbaseTX&lt;/span>&lt;span class="hljs-params">(to, data &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> *&lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">if&lt;/span> data == &lt;span class="hljs-string">""&lt;/span> {
		data = fmt.Sprintf(&lt;span class="hljs-string">"Reward to '%s'"&lt;/span>, to)
	}

	txin := TXInput{[]&lt;span class="hljs-keyword">byte&lt;/span>{}, &lt;span class="hljs-number">-1&lt;/span>, data}
	txout := TXOutput{subsidy, to}
	tx := Transaction{&lt;span class="hljs-literal">nil&lt;/span>, []TXInput{txin}, []TXOutput{txout}}
	tx.SetID()

	&lt;span class="hljs-keyword">return&lt;/span> &tx
}
</code></pre>

A coinbase transaction has only one input. In our implementation its `Txid` is empty and `Vout` equals to -1. Also, a coinbase transaction doesn’t store a script in `ScriptSig`. Instead, arbitrary data is stored there.

> In Bitcoin, the very first coinbase transaction contains the following message: “The Times 03/Jan/2009 Chancellor on brink of second bailout for banks”. [You can see it yourself][3].

`subsidy` is the amount of reward. In Bitcoin, this number is not stored anywhere and calculated based only on the total number of blocks: the number of blocks is divided by `210000`. Mining the genesis block produced 50 BTC, and every `210000` blocks the reward is halved. In our implementation, we’ll store the reward as a constant (at least for now).

## <span class="ez-toc-section" id="Storing_Transactions_in_Blockchain"></span>Storing Transactions in Blockchain<span class="ez-toc-section-end"></span> {#storing-transactions-in-blockchain}

From now on, every block must store at least one transaction and it’s no more possible to mine blocks without transactions. This means that we should remove the `Data` field of `Block` and store transactions instead:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> Block &lt;span class="hljs-keyword">struct&lt;/span> {
	Timestamp     &lt;span class="hljs-keyword">int64&lt;/span>
	Transactions  []*Transaction
	PrevBlockHash []&lt;span class="hljs-keyword">byte&lt;/span>
	Hash          []&lt;span class="hljs-keyword">byte&lt;/span>
	Nonce         &lt;span class="hljs-keyword">int&lt;/span>
}
</code></pre>

`NewBlock` and `NewGenesisBlock` also must be changed accordingly:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewBlock&lt;/span>&lt;span class="hljs-params">(transactions []*Transaction, prevBlockHash []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	block := &Block{time.Now().Unix(), transactions, prevBlockHash, []&lt;span class="hljs-keyword">byte&lt;/span>{}, &lt;span class="hljs-number">0&lt;/span>}
	...
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewGenesisBlock&lt;/span>&lt;span class="hljs-params">(coinbase *Transaction)&lt;/span> *&lt;span class="hljs-title">Block&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> NewBlock([]*Transaction{coinbase}, []&lt;span class="hljs-keyword">byte&lt;/span>{})
}
</code></pre>

Next thing to change is the creation of a new blockchain:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">CreateBlockchain&lt;/span>&lt;span class="hljs-params">(address &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> *&lt;span class="hljs-title">Blockchain&lt;/span>&lt;/span> {
	...
	err = db.Update(&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span>&lt;span class="hljs-params">(tx *bolt.Tx)&lt;/span> &lt;span class="hljs-title">error&lt;/span>&lt;/span> {
		cbtx := NewCoinbaseTX(address, genesisCoinbaseData)
		genesis := NewGenesisBlock(cbtx)

		b, err := tx.CreateBucket([]&lt;span class="hljs-keyword">byte&lt;/span>(blocksBucket))
		err = b.Put(genesis.Hash, genesis.Serialize())
		...
	})
	...
}
</code></pre>

Now, the function takes an address which will receive the reward for mining the genesis block.

## <span class="ez-toc-section" id="Proof-of-Work"></span>Proof-of-Work<span class="ez-toc-section-end"></span> {#proof-of-work}

The Proof-of-Work algorithm must consider transactions stored in a block, to guarantee the consistency and reliability of blockchain as a storage of transaction. So now we must modify the `ProofOfWork.prepareData` method:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(pow *ProofOfWork)&lt;/span> &lt;span class="hljs-title">prepareData&lt;/span>&lt;span class="hljs-params">(nonce &lt;span class="hljs-keyword">int&lt;/span>)&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	data := bytes.Join(
		[][]&lt;span class="hljs-keyword">byte&lt;/span>{
			pow.block.PrevBlockHash,
			pow.block.HashTransactions(), &lt;span class="hljs-comment">// This line was changed&lt;/span>
			IntToHex(pow.block.Timestamp),
			IntToHex(&lt;span class="hljs-keyword">int64&lt;/span>(targetBits)),
			IntToHex(&lt;span class="hljs-keyword">int64&lt;/span>(nonce)),
		},
		[]&lt;span class="hljs-keyword">byte&lt;/span>{},
	)

	&lt;span class="hljs-keyword">return&lt;/span> data
}
</code></pre>

Instead of `pow.block.Data` we now use `pow.block.HashTransactions()` which is:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(b *Block)&lt;/span> &lt;span class="hljs-title">HashTransactions&lt;/span>&lt;span class="hljs-params">()&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> txHashes [][]&lt;span class="hljs-keyword">byte&lt;/span>
	&lt;span class="hljs-keyword">var&lt;/span> txHash [&lt;span class="hljs-number">32&lt;/span>]&lt;span class="hljs-keyword">byte&lt;/span>

	&lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> b.Transactions {
		txHashes = &lt;span class="hljs-built_in">append&lt;/span>(txHashes, tx.ID)
	}
	txHash = sha256.Sum256(bytes.Join(txHashes, []&lt;span class="hljs-keyword">byte&lt;/span>{}))

	&lt;span class="hljs-keyword">return&lt;/span> txHash[:]
}
</code></pre>

Again, we’re using hashing as a mechanism of providing unique representation of data. We want all transactions in a block to be uniquely identified by a single hash. To achieve this, we get hashes of each transaction, concatenate them, and get a hash of the concatenated combination.

> Bitcoin uses a more elaborate technique: it represents all transactions containing in a block as a [Merkle tree][4] and uses the root hash of the tree in the Proof-of-Work system. This approach allows to quickly check if a block contains certain transaction, having only just the root hash and without downloading all the transactions.

Let’s check that everything is correct so far:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go createblockchain -address Ivan&lt;/span>
00000093450837f8b52b78c25f8163bb6137caf43ff4d9a01d1b731fa8ddcc8a

Done!
</code></pre>

Good! We received out first mining reward. But how do we check the balance?

## <span class="ez-toc-section" id="Unspent_Transaction_Outputs"></span>Unspent Transaction Outputs<span class="ez-toc-section-end"></span> {#unspent-transaction-outputs}

We need to find all unspent transaction outputs (UTXO). _Unspent_ means that these outputs weren’t referenced in any inputs. On the diagram above, these are:

  1. tx0, output 1;
  2. tx1, output 0;
  3. tx3, output 0;
  4. tx4, output 0.

Of course, when we check balance, we don’t need all of them, but only those that can be unlocked by the key we own (currently we don’t have keys implemented and will use user defined addresses instead). First, let’s define locking-unlocking methods on inputs and outputs:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(in *TXInput)&lt;/span> &lt;span class="hljs-title">CanUnlockOutputWith&lt;/span>&lt;span class="hljs-params">(unlockingData &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> in.ScriptSig == unlockingData
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(out *TXOutput)&lt;/span> &lt;span class="hljs-title">CanBeUnlockedWith&lt;/span>&lt;span class="hljs-params">(unlockingData &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> out.ScriptPubKey == unlockingData
}
</code></pre>

Here we just compare the script fields with `unlockingData`. These pieces will be improved in a future article, after we implement addresses based on private keys.

The next step &#8211; finding transactions containing unspent outputs &#8211; is quite difficult:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">FindUnspentTransactions&lt;/span>&lt;span class="hljs-params">(address &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> []&lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
  &lt;span class="hljs-keyword">var&lt;/span> unspentTXs []Transaction
  spentTXOs := &lt;span class="hljs-built_in">make&lt;/span>(&lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>][]&lt;span class="hljs-keyword">int&lt;/span>)
  bci := bc.Iterator()

  &lt;span class="hljs-keyword">for&lt;/span> {
    block := bci.Next()

    &lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> block.Transactions {
      txID := hex.EncodeToString(tx.ID)

    Outputs:
      &lt;span class="hljs-keyword">for&lt;/span> outIdx, out := &lt;span class="hljs-keyword">range&lt;/span> tx.Vout {
        &lt;span class="hljs-comment">// Was the output spent?&lt;/span>
        &lt;span class="hljs-keyword">if&lt;/span> spentTXOs[txID] != &lt;span class="hljs-literal">nil&lt;/span> {
          &lt;span class="hljs-keyword">for&lt;/span> _, spentOut := &lt;span class="hljs-keyword">range&lt;/span> spentTXOs[txID] {
            &lt;span class="hljs-keyword">if&lt;/span> spentOut == outIdx {
              &lt;span class="hljs-keyword">continue&lt;/span> Outputs
            }
          }
        }

        &lt;span class="hljs-keyword">if&lt;/span> out.CanBeUnlockedWith(address) {
          unspentTXs = &lt;span class="hljs-built_in">append&lt;/span>(unspentTXs, *tx)
        }
      }

      &lt;span class="hljs-keyword">if&lt;/span> tx.IsCoinbase() == &lt;span class="hljs-literal">false&lt;/span> {
        &lt;span class="hljs-keyword">for&lt;/span> _, in := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
          &lt;span class="hljs-keyword">if&lt;/span> in.CanUnlockOutputWith(address) {
            inTxID := hex.EncodeToString(in.Txid)
            spentTXOs[inTxID] = &lt;span class="hljs-built_in">append&lt;/span>(spentTXOs[inTxID], in.Vout)
          }
        }
      }
    }

    &lt;span class="hljs-keyword">if&lt;/span> &lt;span class="hljs-built_in">len&lt;/span>(block.PrevBlockHash) == &lt;span class="hljs-number">0&lt;/span> {
      &lt;span class="hljs-keyword">break&lt;/span>
    }
  }

  &lt;span class="hljs-keyword">return&lt;/span> unspentTXs
}
</code></pre>

Since transactions are stored in blocks, we have to check every block in a blockchain. We start with outputs:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">if&lt;/span> out.CanBeUnlockedWith(address) {
	unspentTXs = &lt;span class="hljs-built_in">append&lt;/span>(unspentTXs, tx)
}
</code></pre>

If an output was locked by the same address we’re searching unspent transaction outputs for, then this is the output we want. But before taking it, we need to check if an output was already referenced in an input:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">if&lt;/span> spentTXOs[txID] != &lt;span class="hljs-literal">nil&lt;/span> {
	&lt;span class="hljs-keyword">for&lt;/span> _, spentOut := &lt;span class="hljs-keyword">range&lt;/span> spentTXOs[txID] {
		&lt;span class="hljs-keyword">if&lt;/span> spentOut == outIdx {
			&lt;span class="hljs-keyword">continue&lt;/span> Outputs
		}
	}
}
</code></pre>

We skip those that were referenced in inputs (their values were moved to other outputs, thus we cannot count them). After checking outputs we gather all inputs that could unlock outputs locked with the provided address (this doesn’t apply to coinbase transactions, since they don’t unlock outputs):

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">if&lt;/span> tx.IsCoinbase() == &lt;span class="hljs-literal">false&lt;/span> {
    &lt;span class="hljs-keyword">for&lt;/span> _, in := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
        &lt;span class="hljs-keyword">if&lt;/span> in.CanUnlockOutputWith(address) {
            inTxID := hex.EncodeToString(in.Txid)
            spentTXOs[inTxID] = &lt;span class="hljs-built_in">append&lt;/span>(spentTXOs[inTxID], in.Vout)
        }
    }
}
</code></pre>

The function returns a list of transactions containing unspent outputs. To calculate balance we need one more function that takes the transactions and returns only outputs:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">FindUTXO&lt;/span>&lt;span class="hljs-params">(address &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span> []&lt;span class="hljs-title">TXOutput&lt;/span>&lt;/span> {
       &lt;span class="hljs-keyword">var&lt;/span> UTXOs []TXOutput
       unspentTransactions := bc.FindUnspentTransactions(address)

       &lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> unspentTransactions {
               &lt;span class="hljs-keyword">for&lt;/span> _, out := &lt;span class="hljs-keyword">range&lt;/span> tx.Vout {
                       &lt;span class="hljs-keyword">if&lt;/span> out.CanBeUnlockedWith(address) {
                               UTXOs = &lt;span class="hljs-built_in">append&lt;/span>(UTXOs, out)
                       }
               }
       }

       &lt;span class="hljs-keyword">return&lt;/span> UTXOs
}
</code></pre>

That’s it! Now we can implement `getbalance` command:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(cli *CLI)&lt;/span> &lt;span class="hljs-title">getBalance&lt;/span>&lt;span class="hljs-params">(address &lt;span class="hljs-keyword">string&lt;/span>)&lt;/span>&lt;/span> {
	bc := NewBlockchain(address)
	&lt;span class="hljs-keyword">defer&lt;/span> bc.db.Close()

	balance := &lt;span class="hljs-number">0&lt;/span>
	UTXOs := bc.FindUTXO(address)

	&lt;span class="hljs-keyword">for&lt;/span> _, out := &lt;span class="hljs-keyword">range&lt;/span> UTXOs {
		balance += out.Value
	}

	fmt.Printf(&lt;span class="hljs-string">"Balance of '%s': %d\n"&lt;/span>, address, balance)
}

</code></pre>

The account balance is the sum of values of all unspent transaction outputs locked by the account address.

Let’s check our balance after mining the genesis block:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Ivan&lt;/span>
Balance of 'Ivan': 10
</code></pre>

This is our first money!

## <span class="ez-toc-section" id="Sending_Coins"></span>Sending Coins<span class="ez-toc-section-end"></span> {#sending-coins}

Now, we want to send some coins to someone else. For this, we need to create a new transaction, put it in a block, and mine the block. So far, we implemented only the coinbase transaction (which is a special type of transactions), now we need a general transaction:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewUTXOTransaction&lt;/span>&lt;span class="hljs-params">(from, to &lt;span class="hljs-keyword">string&lt;/span>, amount &lt;span class="hljs-keyword">int&lt;/span>, bc *Blockchain)&lt;/span> *&lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> inputs []TXInput
	&lt;span class="hljs-keyword">var&lt;/span> outputs []TXOutput

	acc, validOutputs := bc.FindSpendableOutputs(from, amount)

	&lt;span class="hljs-keyword">if&lt;/span> acc &lt; amount {
		log.Panic(&lt;span class="hljs-string">"ERROR: Not enough funds"&lt;/span>)
	}

	&lt;span class="hljs-comment">// Build a list of inputs&lt;/span>
	&lt;span class="hljs-keyword">for&lt;/span> txid, outs := &lt;span class="hljs-keyword">range&lt;/span> validOutputs {
		txID, err := hex.DecodeString(txid)

		&lt;span class="hljs-keyword">for&lt;/span> _, out := &lt;span class="hljs-keyword">range&lt;/span> outs {
			input := TXInput{txID, out, from}
			inputs = &lt;span class="hljs-built_in">append&lt;/span>(inputs, input)
		}
	}

	&lt;span class="hljs-comment">// Build a list of outputs&lt;/span>
	outputs = &lt;span class="hljs-built_in">append&lt;/span>(outputs, TXOutput{amount, to})
	&lt;span class="hljs-keyword">if&lt;/span> acc &gt; amount {
		outputs = &lt;span class="hljs-built_in">append&lt;/span>(outputs, TXOutput{acc - amount, from}) &lt;span class="hljs-comment">// a change&lt;/span>
	}

	tx := Transaction{&lt;span class="hljs-literal">nil&lt;/span>, inputs, outputs}
	tx.SetID()

	&lt;span class="hljs-keyword">return&lt;/span> &tx
}
</code></pre>

Before creating new outputs, we first have to find all unspent outputs and ensure that they store enough value. This is what `FindSpendableOutputs` method does. After that, for each found output an input referencing it is created. Next, we create two outputs:

  1. One that’s locked with the receiver address. This is the actual transferring of coins to other address.
  2. One that’s locked with the sender address. This is a change. It’s only created when unspent outputs hold more value than required for the new transaction. Remember: outputs are **indivisible**.

`FindSpendableOutputs` method is based on the `FindUnspentTransactions` method we defined earlier:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">FindSpendableOutputs&lt;/span>&lt;span class="hljs-params">(address &lt;span class="hljs-keyword">string&lt;/span>, amount &lt;span class="hljs-keyword">int&lt;/span>)&lt;/span> &lt;span class="hljs-params">(&lt;span class="hljs-keyword">int&lt;/span>, &lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>][]&lt;span class="hljs-keyword">int&lt;/span>)&lt;/span>&lt;/span> {
	unspentOutputs := &lt;span class="hljs-built_in">make&lt;/span>(&lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>][]&lt;span class="hljs-keyword">int&lt;/span>)
	unspentTXs := bc.FindUnspentTransactions(address)
	accumulated := &lt;span class="hljs-number">0&lt;/span>

Work:
	&lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> unspentTXs {
		txID := hex.EncodeToString(tx.ID)

		&lt;span class="hljs-keyword">for&lt;/span> outIdx, out := &lt;span class="hljs-keyword">range&lt;/span> tx.Vout {
			&lt;span class="hljs-keyword">if&lt;/span> out.CanBeUnlockedWith(address) && accumulated &lt; amount {
				accumulated += out.Value
				unspentOutputs[txID] = &lt;span class="hljs-built_in">append&lt;/span>(unspentOutputs[txID], outIdx)

				&lt;span class="hljs-keyword">if&lt;/span> accumulated &gt;= amount {
					&lt;span class="hljs-keyword">break&lt;/span> Work
				}
			}
		}
	}

	&lt;span class="hljs-keyword">return&lt;/span> accumulated, unspentOutputs
}

</code></pre>

The method iterates over all unspent transactions and accumulates their values. When the accumulated value is more or equals to the amount we want to transfer, it stops and returns the accumulated value and output indices grouped by transaction IDs. We don’t want to take more than we’re going to spend.

Now we can modify the `Blockchain.MineBlock` method:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">MineBlock&lt;/span>&lt;span class="hljs-params">(transactions []*Transaction)&lt;/span>&lt;/span> {
	...
	newBlock := NewBlock(transactions, lastHash)
	...
}
</code></pre>

Finally, let’s implement `send` command:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(cli *CLI)&lt;/span> &lt;span class="hljs-title">send&lt;/span>&lt;span class="hljs-params">(from, to &lt;span class="hljs-keyword">string&lt;/span>, amount &lt;span class="hljs-keyword">int&lt;/span>)&lt;/span>&lt;/span> {
	bc := NewBlockchain(from)
	&lt;span class="hljs-keyword">defer&lt;/span> bc.db.Close()

	tx := NewUTXOTransaction(from, to, amount, bc)
	bc.MineBlock([]*Transaction{tx})
	fmt.Println(&lt;span class="hljs-string">"Success!"&lt;/span>)
}
</code></pre>

Sending coins means creating a transaction and adding it to the blockchain via mining a block. But Bitcoin doesn’t do this immediately (as we do). Instead, it puts all new transactions into memory pool (or mempool), and when a miner is ready to mine a block, it takes all transactions from the mempool and creates a candidate block. Transactions become confirmed only when a block containing them is mined and added to the blockchain.

Let’s check that sending coins works:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go send -from Ivan -to Pedro -amount 6&lt;/span>
00000001b56d60f86f72ab2a59fadb197d767b97d4873732be505e0a65cc1e37

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Ivan&lt;/span>
Balance of 'Ivan': 4
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Pedro&lt;/span>
Balance of 'Pedro': 6
</code></pre>

Nice! Now, let’s create more transactions and ensure that sending from multiple outputs works fine:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go send -from Pedro -to Helen -amount 2&lt;/span>
00000099938725eb2c7730844b3cd40209d46bce2c2af9d87c2b7611fe9d5bdf

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go send -from Ivan -to Helen -amount 2&lt;/span>
000000a2edf94334b1d94f98d22d7e4c973261660397dc7340464f7959a7a9aa

Success!
</code></pre>

Now, Helen’s coins are locked in two outputs: one from Pedro and one from Ivan. Let’s send them to someone else:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go send -from Helen -to Rachel -amount 3&lt;/span>
000000c58136cffa669e767b8f881d16e2ede3974d71df43058baaf8c069f1a0

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Ivan&lt;/span>
Balance of 'Ivan': 2
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Pedro&lt;/span>
Balance of 'Pedro': 4
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Helen&lt;/span>
Balance of 'Helen': 1
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Rachel&lt;/span>
Balance of 'Rachel': 3
</code></pre>

Looks fine! Now let’s test a failure:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go send -from Pedro -to Ivan -amount 5&lt;/span>
panic: ERROR: Not enough funds
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Pedro&lt;/span>
Balance of 'Pedro': 4
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address Ivan&lt;/span>
Balance of 'Ivan': 2
</code></pre>

## <span class="ez-toc-section" id="Conclusion"></span>Conclusion<span class="ez-toc-section-end"></span> {#conclusion}

Phew! It wasn’t easy, but we have transactions now! Although, some key features of a Bitcoin-like cryptocurrency are missing:

  1. Addresses. We don’t have real, private key based addresses yet.
  2. Rewards. Mining blocks is absolutely not profitable!
  3. UTXO set. Getting balance requires scanning the whole blockchain, which can take very long time when there are many and many blocks. Also, it can take a lot of time if we want to validate later transactions. UTXO set is intended to solve these problems and make operations with transactions fast.
  4. Mempool. This is where transactions are stored before being packed in a block. In our current implementation, a block contains only one transaction, and this is quite inefficient.

## <span class="ez-toc-section" id="Links"></span>Links:<span class="ez-toc-section-end"></span>

  1. [Full source codes][5]
  2. [Transaction][6]
  3. [Merkle tree][7]
  4. [Coinbase][8]

The original link：<https://jeiwan.cc/posts/building-blockchain-in-go-part-3/>

Chinese translations: [by liuchengxu][9], [by zhangli1][10]

 [1]: https://github.com/Jeiwan/blockchain_go/compare/part_3...part_4#files_bucket
 [2]: https://en.bitcoin.it/wiki/Script
 [3]: https://blockchain.info/tx/4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b?show_adv=true
 [4]: https://en.wikipedia.org/wiki/Merkle_tree
 [5]: https://github.com/Jeiwan/blockchain_go/tree/part_4
 [6]: https://en.bitcoin.it/wiki/Transaction
 [7]: https://en.bitcoin.it/wiki/Protocol_documentation#Merkle_Trees
 [8]: https://en.bitcoin.it/wiki/Coinbase
 [9]: https://github.com/liuchengxu/blockchain-tutorial/blob/master/content/part-4/transactions-1.md
 [10]: https://zhangli1.gitbooks.io/dummies-for-blockchain/content/