---
title: 'Building Blockchain in Go. Part 5: Addresses'
date: 2018-07-27T16:10:03+00:00
categories:
 - 区块链
tags:
 - 区块链
series:
 - 构建自己的区块链
series_order: 5
---

## <span class="ez-toc-section" id="Introduction"></span>Introduction<span class="ez-toc-section-end"></span> {#introduction}

In [the previous article][1], we started implementing transactions. You were also introduced to the impersonal nature of transactions: there are no user accounts, your personal data (e.g., name, passport number or SSN) is not required and not stored anywhere in Bitcoin. But there still must be something that identifies you as the owner of transaction outputs (i.e. the owner of coins locked on these outputs). And this is what Bitcoin addresses are needed for. So far we’ve used arbitrary user defined strings as addresses, and the time has come to implement real addresses, as they’re implemented in Bitcoin.

This part introduces significant code changes, so it makes no sense explaining all of them here. Please refer to [this page][2] to see all the changes since the last article.

## <span class="ez-toc-section" id="Bitcoin_Address"></span>Bitcoin Address<span class="ez-toc-section-end"></span> {#bitcoin-address}

Here’s an example of a Bitcoin address: [1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa][3]. This is the very first Bitcoin address, which allegedly belongs to Satoshi Nakamoto. Bitcoin addresses are public. If you want to send coins to someone, you need to know their address. But addresses (despite being unique) are not something that identifies you as the owner of a “wallet”. In fact, such addresses are a human readable representation of public keys. In Bitcoin, your identity is a pair (or pairs) of private and public keys stored on your computer (or stored in some other place you have access to). Bitcoin relies on a combination of cryptography algorithms to create these keys, and guarantee that no one else in the world can access your coins without getting physical access to your keys. Let’s discuss what these algorithms are.

## <span class="ez-toc-section" id="Public-key_Cryptography"></span>Public-key Cryptography<span class="ez-toc-section-end"></span> {#public-key-cryptography}

Public-key cryptography algorithms use pairs of keys: public keys and private keys. Public keys are not sensitive and can be disclosed to anyone. In contrast, private keys shouldn’t be disclosed: no one but the owner should have access to them because it’s private keys that serve as the identifier of the owner. You are your private keys (in the world of cryptocurrencies, of course).

In essence, a Bitcoin wallet is just a pair of such keys. When you install a wallet application or use a Bitcoin client to generate a new address, a pair of keys is generated for you. The one who controls the private key controls all the coins sent to this key in Bitcoin.

Private and public keys are just random sequences of bytes, thus they cannot be printed on the screen and read by a human. That’s why Bitcoin uses an algorithm to convert public keys into a human readable string.

If you’ve ever used a Bitcoin wallet application, it’s likely that a mnemonic pass phrase was generated for you. Such phrases are used instead of private keys and can be used to generate them. This mechanism is implemented in [BIP-039][4].

Ok, we now know what identifies users in Bitcoin. But how does Bitcoin check the ownership of transaction outputs (and coins stored on them)?

## <span class="ez-toc-section" id="Digital_Signatures"></span>Digital Signatures<span class="ez-toc-section-end"></span> {#digital-signatures}

In mathematics and cryptography, there’s a concept of digital signature – algorithms that guarantee:

  1. that data wasn’t modified while being transferred from a sender to a recipient;
  2. that data was created by a certain sender;
  3. that the sender cannot deny sending the data.

By applying a signing algorithm to data (i.e., signing the data), one gets a signature, which can later be verified. Digital signing happens with the usage of a private key, and verification requires a public key.

In order to sign data we need the following things:

  1. data to sign;
  2. private key.

The operation of signing produces a signature, which is stored in transaction inputs. In order to verify a signature, the following is required:

  1. data that was signed;
  2. the signature;
  3. public key.

In simple terms, the verification process can be described as: check that this signature was obtained from this data with a private key used to generate the public key.

Digital signatures are not encryption, you cannot reconstruct the data from a signature. This is similar to hashing: you run data through a hashing algorithm and get a unique representation of the data. The difference between signatures and hashes is key pairs: they make signature verification possible.  
But key pairs can also be used to encrypt data: a private key is used to encrypt, and a public key is used to decrypt the data. Bitcoin doesn’t use encryption algorithms though.

Every transaction input in Bitcoin is signed by the one who created the transaction. Every transaction in Bitcoin must be verified before being put in a block. Verification means (besides other procedures):

  1. Checking that inputs have permission to use outputs from previous transactions.
  2. Checking that the transaction signature is correct.

Schematically, the process of signing data and verifying signature looks likes this:

<img decoding="async" loading="lazy" class="alignnone size-medium wp-image-323" src="http://roliu.work/wp-content/uploads/2018/07/signing-scheme-300x129.png" alt="" width="300" height="129" /> 

<img decoding="async" loading="lazy" class="aligncenter wp-image-323 size-full" src="http://roliu.work/wp-content/uploads/2018/07/signing-scheme.png" alt="" width="1014" height="435" /> 

Let’s now review the full lifecycle of a transaction:

  1. In the beginning, there’s the genesis block that contains a coinbase transaction. There are no real inputs in coinbase transactions, so signing is not necessary. The output of the coinbase transaction contains a hashed public key (`RIPEMD16(SHA256(PubKey))` algorithms are used).
  2. When one sends coins, a transaction is created. Inputs of the transaction will reference outputs from previous transaction(s). Every input will store a public key (not hashed) and a signature of the whole transaction.
  3. Other nodes in the Bitcoin network that receive the transaction will verify it. Besides other things, they will check that: the hash of the public key in an input matches the hash of the referenced output (this ensures that the sender spends only coins belonging to them); the signature is correct (this ensures that the transaction is created by the real owner of the coins).
  4. When a miner node is ready to mine a new block, it’ll put the transaction in a block and start mining it.
  5. When the blocked is mined, every other node in the network receives a message saying the block is mined and adds the block to the blockchain.
  6. After a block is added to the blockchain, the transaction is completed, its outputs can be referenced in new transactions.

## <span class="ez-toc-section" id="Elliptic_Curve_Cryptography"></span>Elliptic Curve Cryptography<span class="ez-toc-section-end"></span> {#elliptic-curve-cryptography}

As described above, public and private keys are sequences of random bytes. Since it’s private keys that are used to identify owners of coins, there’s a required condition: the randomness algorithm must produce truly random bytes. We don’t want to accidentally generate a private key that’s owned by someone else.

Bitcoin uses elliptic curves to generate private keys. Elliptic curves is a complex mathematical concept, which we’re not going to explain in details here (if you’re curious, check out [this gentle introduction to elliptic curves][5] WARNING: Math formulas!). What we need to know is that these curves can be used to generate really big and random numbers. The curve used by Bitcoin can randomly pick a number between 0 and 2²⁵⁶ (which is approximately 10⁷⁷, when there are between 10⁷⁸ and 10⁸² atoms in the visible universe). Such a huge upper limit means that it’s almost impossible to generate the same private key twice.

Also, Bitcoin uses (and we will) ECDSA (Elliptic Curve Digital Signature Algorithm) algorithm to sign transactions.

## <span class="ez-toc-section" id="Base58"></span>Base58<span class="ez-toc-section-end"></span> {#base58}

Now let’s get back to the above mentioned Bitcoin address: 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa. Now we know that this is a human-readable representation of a public key. And if we decode it, here’s what the public key looks like (as a sequence of bytes written in the hexadecimal system):

<pre><code class="language-shell">0062E907B15CBF27D5425399EBF6F0FB50EBB88F18C29B7D93
</code></pre>

Bitcoin uses the Base58 algorithm to convert public keys into human readable format. The algorithm is very similar to famous Base64, but it uses shorter alphabet: some letters were removed from the alphabet to avoid some attacks that use letters similarity. Thus, there are no these symbols: 0 (zero), O (capital o), I (capital i), l (lowercase L), because they look similar. Also, there are no + and / symbols.

Let’s schematically visualize the process of getting an address from a public key:

<img decoding="async" loading="lazy" class="aligncenter wp-image-324 size-full" src="http://roliu.work/wp-content/uploads/2018/07/address-generation-scheme.png" alt="" width="1130" height="814" /> 

Thus, the above mentioned decoded public key consists of three parts:

<pre><code class="language-shell hljs">Version  Public key hash                           Checksum
00       62E907B15CBF27D5425399EBF6F0FB50EBB88F18  C29B7D93
</code></pre>

Since hashing functions are one way (i.e., they cannot be reversed), it’s not possible to extract the public key from the hash. But we can check if a public key was used to get the hash by running it thought the save hash functions and comparing the hashes.

Ok, now that we have all the pieces, let’s write some code. Some of the concepts should be more clear when written in code.

## <span class="ez-toc-section" id="Implementing_Addresses"></span>Implementing Addresses<span class="ez-toc-section-end"></span> {#implementing-addresses}

We’ll start with the `Wallet` structure:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> Wallet &lt;span class="hljs-keyword">struct&lt;/span> {
	PrivateKey ecdsa.PrivateKey
	PublicKey  []&lt;span class="hljs-keyword">byte&lt;/span>
}

&lt;span class="hljs-keyword">type&lt;/span> Wallets &lt;span class="hljs-keyword">struct&lt;/span> {
	Wallets &lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>]*Wallet
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewWallet&lt;/span>&lt;span class="hljs-params">()&lt;/span> *&lt;span class="hljs-title">Wallet&lt;/span>&lt;/span> {
	private, public := newKeyPair()
	wallet := Wallet{private, public}

	&lt;span class="hljs-keyword">return&lt;/span> &wallet
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">newKeyPair&lt;/span>&lt;span class="hljs-params">()&lt;/span> &lt;span class="hljs-params">(ecdsa.PrivateKey, []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span>&lt;/span> {
	curve := elliptic.P256()
	private, err := ecdsa.GenerateKey(curve, rand.Reader)
	pubKey := &lt;span class="hljs-built_in">append&lt;/span>(private.PublicKey.X.Bytes(), private.PublicKey.Y.Bytes()...)

	&lt;span class="hljs-keyword">return&lt;/span> *private, pubKey
}
</code></pre>

A wallet is nothing but a key pair. We’ll also need the `Wallets` type to keep a collection of wallets, save them to a file, and load them from it. In the construction function of `Wallet` a new key pair is generated. The `newKeyPair` function is straightforward: ECDSA is based on elliptic curves, so we need one. Next, a private key is generated using the curve, and a public key is generated from the private key. One thing to notice: in elliptic curve based algorithms, public keys are points on a curve. Thus, a public key is a combination of X, Y coordinates. In Bitcoin, these coordinates are concatenated and form a public key.

Now, let’s generate an address:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(w Wallet)&lt;/span> &lt;span class="hljs-title">GetAddress&lt;/span>&lt;span class="hljs-params">()&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	pubKeyHash := HashPubKey(w.PublicKey)

	versionedPayload := &lt;span class="hljs-built_in">append&lt;/span>([]&lt;span class="hljs-keyword">byte&lt;/span>{version}, pubKeyHash...)
	checksum := checksum(versionedPayload)

	fullPayload := &lt;span class="hljs-built_in">append&lt;/span>(versionedPayload, checksum...)
	address := Base58Encode(fullPayload)

	&lt;span class="hljs-keyword">return&lt;/span> address
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">HashPubKey&lt;/span>&lt;span class="hljs-params">(pubKey []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	publicSHA256 := sha256.Sum256(pubKey)

	RIPEMD160Hasher := ripemd160.New()
	_, err := RIPEMD160Hasher.Write(publicSHA256[:])
	publicRIPEMD160 := RIPEMD160Hasher.Sum(&lt;span class="hljs-literal">nil&lt;/span>)

	&lt;span class="hljs-keyword">return&lt;/span> publicRIPEMD160
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">checksum&lt;/span>&lt;span class="hljs-params">(payload []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> []&lt;span class="hljs-title">byte&lt;/span>&lt;/span> {
	firstSHA := sha256.Sum256(payload)
	secondSHA := sha256.Sum256(firstSHA[:])

	&lt;span class="hljs-keyword">return&lt;/span> secondSHA[:addressChecksumLen]
}
</code></pre>

Here are the steps to convert a public key into a Base58 address:

  1. Take the public key and hash it twice with `RIPEMD160(SHA256(PubKey))` hashing algorithms.
  2. Prepend the version of the address generation algorithm to the hash.
  3. Calculate the checksum by hashing the result of step 2 with `SHA256(SHA256(payload))`. The checksum is the first four bytes of the resulted hash.
  4. Append the checksum to the `version+PubKeyHash` combination.
  5. Encode the `version+PubKeyHash+checksum` combination with Base58.

As a result, you’ll get a **real Bitcoin address**, you can even check its balance on [blockchain.info][6]. But I can assure you that the balance is 0 no matter how many times you generate a new address and check its balance. This is why choosing proper public-key cryptography algorithm is so crucial: considering private keys are random numbers, the chance of generating the same number must be as low as possible. Ideally, it must be as low as “never”.

Also, pay attention that you don’t need to connect to a Bitcoin node to get an address. The address generation algorithm utilizes a combination of open algorithms that are implemented in many programming languages and libraries.

Now we need to modify inputs and outputs for them to use addresses:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">type&lt;/span> TXInput &lt;span class="hljs-keyword">struct&lt;/span> {
	Txid      []&lt;span class="hljs-keyword">byte&lt;/span>
	Vout      &lt;span class="hljs-keyword">int&lt;/span>
	Signature []&lt;span class="hljs-keyword">byte&lt;/span>
	PubKey    []&lt;span class="hljs-keyword">byte&lt;/span>
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(in *TXInput)&lt;/span> &lt;span class="hljs-title">UsesKey&lt;/span>&lt;span class="hljs-params">(pubKeyHash []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	lockingHash := HashPubKey(in.PubKey)

	&lt;span class="hljs-keyword">return&lt;/span> bytes.Compare(lockingHash, pubKeyHash) == &lt;span class="hljs-number">0&lt;/span>
}

&lt;span class="hljs-keyword">type&lt;/span> TXOutput &lt;span class="hljs-keyword">struct&lt;/span> {
	Value      &lt;span class="hljs-keyword">int&lt;/span>
	PubKeyHash []&lt;span class="hljs-keyword">byte&lt;/span>
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(out *TXOutput)&lt;/span> &lt;span class="hljs-title">Lock&lt;/span>&lt;span class="hljs-params">(address []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span>&lt;/span> {
	pubKeyHash := Base58Decode(address)
	pubKeyHash = pubKeyHash[&lt;span class="hljs-number">1&lt;/span> : &lt;span class="hljs-built_in">len&lt;/span>(pubKeyHash)&lt;span class="hljs-number">-4&lt;/span>]
	out.PubKeyHash = pubKeyHash
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(out *TXOutput)&lt;/span> &lt;span class="hljs-title">IsLockedWithKey&lt;/span>&lt;span class="hljs-params">(pubKeyHash []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">return&lt;/span> bytes.Compare(out.PubKeyHash, pubKeyHash) == &lt;span class="hljs-number">0&lt;/span>
}
</code></pre>

Notice, that we’re no longer using `ScriptPubKey` and `ScriptSig` fields, because we’re not going to implement a scripting language. Instead, `ScriptSig` is split into `Signature` and `PubKey` fields, and `ScriptPubKey` is renamed to `PubKeyHash`. We’ll implement the same outputs locking/unlocking and inputs signing logics as in Bitcoin, but we’ll do this in methods instead.

The `UsesKey` method checks that an input uses a specific key to unlock an output. Notice that inputs store raw public keys (i.e., not hashed), but the function takes a hashed one. `IsLockedWithKey` checks if provided public key hash was used to lock the output. This is a complementary function to `UsesKey`, and they’re both used in `FindUnspentTransactions` to build connections between transactions.

`Lock` simply locks an output. When we send coins to someone, we know only their address, thus the function takes an address as the only argument. The address is then decoded and the public key hash is extracted from it and saved in the `PubKeyHash` field.

Now, let’s check that everything works correctly:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go createwallet&lt;/span>
Your new address: 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go createwallet&lt;/span>
Your new address: 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go createwallet&lt;/span>
Your new address: 1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go createblockchain -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt&lt;/span>
0000005420fbfdafa00c093f56e033903ba43599fa7cd9df40458e373eee724d

Done!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt&lt;/span>
Balance of '13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt': 10
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go send -from 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h -to 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt -amount 5&lt;/span>
2017/09/12 13:08:56 ERROR: Not enough funds
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go send -from 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt -to 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h -amount 6&lt;/span>
00000019afa909094193f64ca06e9039849709f5948fbac56cae7b1b8f0ff162

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt&lt;/span>
Balance of '13Uu7B1vDP4ViXqHFsWtbraM3EfQ3UkWXt': 4
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h&lt;/span>
Balance of '15pUhCbtrGh3JUx5iHnXjfpyHyTgawvG5h': 6
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy&lt;/span>
Balance of '1Lhqun1E9zZZhodiTqxfPQBcwr1CVDV2sy': 0
</code></pre>

Nice! Now let’s implement transaction signatures.

## <span class="ez-toc-section" id="Implementing_Signatures"></span>Implementing Signatures<span class="ez-toc-section-end"></span> {#implementing-signatures}

Transactions must be signed because this is the only way in Bitcoin to guarantee that one cannot spend coins belonging to someone else. If a signature is invalid, the transaction is considered invalid too and, thus, cannot be added to the blockchain.

We have all the pieces to implement transactions signing, except one thing: data to sign. What parts of a transaction are actually signed? Or a transaction is signed as a whole? Choosing data to sign is quite important. The thing is that data to be signed must contain information that identifies the data in a unique way. For example, it makes no sense signing just output values because such signature won’t consider the sender and the recipient.

Considering that transactions unlock previous outputs, redistribute their values, and lock new outputs, the following data must be signed:

  1. Public key hashes stored in unlocked outputs. This identifies “sender” of a transaction.
  2. Public key hashes stored in new, locked, outputs. This identifies “recipient” of a transaction.
  3. Values of new outputs.

> In Bitcoin, locking/unlocking logic is stored in scripts, which are stored in `ScriptSig` and `ScriptPubKey` fields of inputs and outputs, respectively. Since Bitcoins allows different types of such scripts, it signs the whole content of `ScriptPubKey`.

As you can see, we don’t need to sign the public keys stored in inputs. Because of this, in Bitcoin, it’s not a transaction that’s signed, but its trimmed copy with inputs storing `ScriptPubKey` from referenced outputs.

> A detailed process of getting a trimmed transaction copy is described [here][7]. It’s likely to be outdated, but I didn’t manage to find a more reliable source of information.

Ok, it looks complicated, so let’s start coding. We’ll start with the `Sign` method:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(tx *Transaction)&lt;/span> &lt;span class="hljs-title">Sign&lt;/span>&lt;span class="hljs-params">(privKey ecdsa.PrivateKey, prevTXs &lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>]Transaction)&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">if&lt;/span> tx.IsCoinbase() {
		&lt;span class="hljs-keyword">return&lt;/span>
	}

	txCopy := tx.TrimmedCopy()

	&lt;span class="hljs-keyword">for&lt;/span> inID, vin := &lt;span class="hljs-keyword">range&lt;/span> txCopy.Vin {
		prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
		txCopy.Vin[inID].Signature = &lt;span class="hljs-literal">nil&lt;/span>
		txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
		txCopy.ID = txCopy.Hash()
		txCopy.Vin[inID].PubKey = &lt;span class="hljs-literal">nil&lt;/span>

		r, s, err := ecdsa.Sign(rand.Reader, &privKey, txCopy.ID)
		signature := &lt;span class="hljs-built_in">append&lt;/span>(r.Bytes(), s.Bytes()...)

		tx.Vin[inID].Signature = signature
	}
}
</code></pre>

The method takes a private key and a map of previous transactions. As mentioned above, in order to sign a transaction, we need to access the outputs referenced in the inputs of the transaction, thus we need the transactions that store these outputs.

Let’s review this method step by step:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">if&lt;/span> tx.IsCoinbase() {
	&lt;span class="hljs-keyword">return&lt;/span>
}
</code></pre>

Coinbase transactions are not signed because there are no real inputs in them.

<pre><code class="language-go hljs">txCopy := tx.TrimmedCopy()
</code></pre>

A trimmed copy will be signed, not a full transaction:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(tx *Transaction)&lt;/span> &lt;span class="hljs-title">TrimmedCopy&lt;/span>&lt;span class="hljs-params">()&lt;/span> &lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> inputs []TXInput
	&lt;span class="hljs-keyword">var&lt;/span> outputs []TXOutput

	&lt;span class="hljs-keyword">for&lt;/span> _, vin := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
		inputs = &lt;span class="hljs-built_in">append&lt;/span>(inputs, TXInput{vin.Txid, vin.Vout, &lt;span class="hljs-literal">nil&lt;/span>, &lt;span class="hljs-literal">nil&lt;/span>})
	}

	&lt;span class="hljs-keyword">for&lt;/span> _, vout := &lt;span class="hljs-keyword">range&lt;/span> tx.Vout {
		outputs = &lt;span class="hljs-built_in">append&lt;/span>(outputs, TXOutput{vout.Value, vout.PubKeyHash})
	}

	txCopy := Transaction{tx.ID, inputs, outputs}

	&lt;span class="hljs-keyword">return&lt;/span> txCopy
}
</code></pre>

The copy will include all the inputs and outputs, but `TXInput.Signature` and `TXInput.PubKey` are set to nil.

Next, we iterate over each input in the copy:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">for&lt;/span> inID, vin := &lt;span class="hljs-keyword">range&lt;/span> txCopy.Vin {
	prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
	txCopy.Vin[inID].Signature = &lt;span class="hljs-literal">nil&lt;/span>
	txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
</code></pre>

In each input, `Signature` is set to `nil` (just a double-check) and `PubKey` is set to the `PubKeyHash` of the referenced output. At this moment, all transactions but the current one are “empty”, i.e. their `Signature` and `PubKey` fields are set to nil. Thus, **inputs are signed separately**, although this is not necessary for our application, but Bitcoin allows transactions to contain inputs referencing different addresses.

<pre><code class="language-go hljs">	txCopy.ID = txCopy.Hash()
	txCopy.Vin[inID].PubKey = &lt;span class="hljs-literal">nil&lt;/span>

</code></pre>

The `Hash` method serializes the transaction and hashes it with the SHA-256 algorithm. The resulted hash is the data we’re going to sign. After getting the hash we should reset the `PubKey` field, so it doesn’t affect further iterations.

Now, the central piece:

<pre><code class="language-go hljs">	r, s, err := ecdsa.Sign(rand.Reader, &privKey, txCopy.ID)
	signature := &lt;span class="hljs-built_in">append&lt;/span>(r.Bytes(), s.Bytes()...)

	tx.Vin[inID].Signature = signature
</code></pre>

We sign `txCopy.ID` with `privKey`. An ECDSA signature is a pair of numbers, which we concatenate and store in the input’s `Signature` field.

Now, the verification function:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(tx *Transaction)&lt;/span> &lt;span class="hljs-title">Verify&lt;/span>&lt;span class="hljs-params">(prevTXs &lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>]Transaction)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	txCopy := tx.TrimmedCopy()
	curve := elliptic.P256()

	&lt;span class="hljs-keyword">for&lt;/span> inID, vin := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
		prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
		txCopy.Vin[inID].Signature = &lt;span class="hljs-literal">nil&lt;/span>
		txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
		txCopy.ID = txCopy.Hash()
		txCopy.Vin[inID].PubKey = &lt;span class="hljs-literal">nil&lt;/span>

		r := big.Int{}
		s := big.Int{}
		sigLen := &lt;span class="hljs-built_in">len&lt;/span>(vin.Signature)
		r.SetBytes(vin.Signature[:(sigLen / &lt;span class="hljs-number">2&lt;/span>)])
		s.SetBytes(vin.Signature[(sigLen / &lt;span class="hljs-number">2&lt;/span>):])

		x := big.Int{}
		y := big.Int{}
		keyLen := &lt;span class="hljs-built_in">len&lt;/span>(vin.PubKey)
		x.SetBytes(vin.PubKey[:(keyLen / &lt;span class="hljs-number">2&lt;/span>)])
		y.SetBytes(vin.PubKey[(keyLen / &lt;span class="hljs-number">2&lt;/span>):])

		rawPubKey := ecdsa.PublicKey{curve, &x, &y}
		&lt;span class="hljs-keyword">if&lt;/span> ecdsa.Verify(&rawPubKey, txCopy.ID, &r, &s) == &lt;span class="hljs-literal">false&lt;/span> {
			&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">false&lt;/span>
		}
	}

	&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">true&lt;/span>
}
</code></pre>

The method is quite straightforward. First, we need the same transaction copy:

<pre><code class="language-go hljs">txCopy := tx.TrimmedCopy()
</code></pre>

Next, we’ll need the same curve that is used to generate key pairs:

<pre><code class="language-go hljs">curve := elliptic.P256()
</code></pre>

Next, we check signature in each input:

<pre><code class="language-go hljs">&lt;span class="hljs-keyword">for&lt;/span> inID, vin := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
	prevTx := prevTXs[hex.EncodeToString(vin.Txid)]
	txCopy.Vin[inID].Signature = &lt;span class="hljs-literal">nil&lt;/span>
	txCopy.Vin[inID].PubKey = prevTx.Vout[vin.Vout].PubKeyHash
	txCopy.ID = txCopy.Hash()
	txCopy.Vin[inID].PubKey = &lt;span class="hljs-literal">nil&lt;/span>
</code></pre>

This piece is identical to the one in the `Sign` method, because during verification we need the same data what was signed.

<pre><code class="language-go hljs">	r := big.Int{}
	s := big.Int{}
	sigLen := &lt;span class="hljs-built_in">len&lt;/span>(vin.Signature)
	r.SetBytes(vin.Signature[:(sigLen / &lt;span class="hljs-number">2&lt;/span>)])
	s.SetBytes(vin.Signature[(sigLen / &lt;span class="hljs-number">2&lt;/span>):])

	x := big.Int{}
	y := big.Int{}
	keyLen := &lt;span class="hljs-built_in">len&lt;/span>(vin.PubKey)
	x.SetBytes(vin.PubKey[:(keyLen / &lt;span class="hljs-number">2&lt;/span>)])
	y.SetBytes(vin.PubKey[(keyLen / &lt;span class="hljs-number">2&lt;/span>):])
</code></pre>

Here we unpack values stored in `TXInput.Signature` and `TXInput.PubKey`, since a signature is a pair of numbers and a public key is a pair of coordinates. We concatenated them earlier for storing, and now we need to unpack them to use in `crypto/ecdsa` functions.

<pre><code class="language-go hljs">	rawPubKey := ecdsa.PublicKey{curve, &x, &y}
	&lt;span class="hljs-keyword">if&lt;/span> ecdsa.Verify(&rawPubKey, txCopy.ID, &r, &s) == &lt;span class="hljs-literal">false&lt;/span> {
		&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">false&lt;/span>
	}
}

&lt;span class="hljs-keyword">return&lt;/span> &lt;span class="hljs-literal">true&lt;/span>
</code></pre>

Here it is: we create an `ecdsa.PublicKey` using the public key extracted from the input and execute `ecdsa.Verify` passing the signature extracted from the input. If all inputs are verified, return true; if at least one input fails verification, return false.

Now, we need a function to obtain previous transactions. Since this requires interaction with the blockchain, we’ll make it a method of `Blockchain`:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">FindTransaction&lt;/span>&lt;span class="hljs-params">(ID []&lt;span class="hljs-keyword">byte&lt;/span>)&lt;/span> &lt;span class="hljs-params">(Transaction, error)&lt;/span>&lt;/span> {
	bci := bc.Iterator()

	&lt;span class="hljs-keyword">for&lt;/span> {
		block := bci.Next()

		&lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> block.Transactions {
			&lt;span class="hljs-keyword">if&lt;/span> bytes.Compare(tx.ID, ID) == &lt;span class="hljs-number">0&lt;/span> {
				&lt;span class="hljs-keyword">return&lt;/span> *tx, &lt;span class="hljs-literal">nil&lt;/span>
			}
		}

		&lt;span class="hljs-keyword">if&lt;/span> &lt;span class="hljs-built_in">len&lt;/span>(block.PrevBlockHash) == &lt;span class="hljs-number">0&lt;/span> {
			&lt;span class="hljs-keyword">break&lt;/span>
		}
	}

	&lt;span class="hljs-keyword">return&lt;/span> Transaction{}, errors.New(&lt;span class="hljs-string">"Transaction is not found"&lt;/span>)
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">SignTransaction&lt;/span>&lt;span class="hljs-params">(tx *Transaction, privKey ecdsa.PrivateKey)&lt;/span>&lt;/span> {
	prevTXs := &lt;span class="hljs-built_in">make&lt;/span>(&lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>]Transaction)

	&lt;span class="hljs-keyword">for&lt;/span> _, vin := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
		prevTX, err := bc.FindTransaction(vin.Txid)
		prevTXs[hex.EncodeToString(prevTX.ID)] = prevTX
	}

	tx.Sign(privKey, prevTXs)
}

&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">VerifyTransaction&lt;/span>&lt;span class="hljs-params">(tx *Transaction)&lt;/span> &lt;span class="hljs-title">bool&lt;/span>&lt;/span> {
	prevTXs := &lt;span class="hljs-built_in">make&lt;/span>(&lt;span class="hljs-keyword">map&lt;/span>[&lt;span class="hljs-keyword">string&lt;/span>]Transaction)

	&lt;span class="hljs-keyword">for&lt;/span> _, vin := &lt;span class="hljs-keyword">range&lt;/span> tx.Vin {
		prevTX, err := bc.FindTransaction(vin.Txid)
		prevTXs[hex.EncodeToString(prevTX.ID)] = prevTX
	}

	&lt;span class="hljs-keyword">return&lt;/span> tx.Verify(prevTXs)
}
</code></pre>

These functions are simple: `FindTransaction` finds a transaction by ID (this requires iterating over all the blocks in the blockchain); `SignTransaction` takes a transaction, finds transactions it references, and signs it; `VerifyTransaction` does the same, but verifies the transaction instead.

Now, we need to actually sign and verify transactions. Signing happens in the `NewUTXOTransaction`:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewUTXOTransaction&lt;/span>&lt;span class="hljs-params">(from, to &lt;span class="hljs-keyword">string&lt;/span>, amount &lt;span class="hljs-keyword">int&lt;/span>, bc *Blockchain)&lt;/span> *&lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
	...

	tx := Transaction{&lt;span class="hljs-literal">nil&lt;/span>, inputs, outputs}
	tx.ID = tx.Hash()
	bc.SignTransaction(&tx, wallet.PrivateKey)

	&lt;span class="hljs-keyword">return&lt;/span> &tx
}
</code></pre>

Verification happens before a transaction is put into a block:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-params">(bc *Blockchain)&lt;/span> &lt;span class="hljs-title">MineBlock&lt;/span>&lt;span class="hljs-params">(transactions []*Transaction)&lt;/span>&lt;/span> {
	&lt;span class="hljs-keyword">var&lt;/span> lastHash []&lt;span class="hljs-keyword">byte&lt;/span>

	&lt;span class="hljs-keyword">for&lt;/span> _, tx := &lt;span class="hljs-keyword">range&lt;/span> transactions {
		&lt;span class="hljs-keyword">if&lt;/span> bc.VerifyTransaction(tx) != &lt;span class="hljs-literal">true&lt;/span> {
			log.Panic(&lt;span class="hljs-string">"ERROR: Invalid transaction"&lt;/span>)
		}
	}
	...
}
</code></pre>

And that’s it! Let’s check everything one more time:

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go createwallet&lt;/span>
Your new address: 1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go createwallet&lt;/span>
Your new address: 1NE86r4Esjf53EL7fR86CsfTZpNN42Sfab
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go createblockchain -address 1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR&lt;/span>
000000122348da06c19e5c513710340f4c307d884385da948a205655c6a9d008

Done!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go send -from 1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR -to 1NE86r4Esjf53EL7fR86CsfTZpNN42Sfab -amount 6&lt;/span>
0000000f3dbb0ab6d56c4e4b9f7479afe8d5a5dad4d2a8823345a1a16cf3347b

Success!
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR&lt;/span>
Balance of '1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR': 4
&lt;span class="hljs-meta">
$&lt;/span>&lt;span class="bash"> blockchain_go getbalance -address 1NE86r4Esjf53EL7fR86CsfTZpNN42Sfab&lt;/span>
Balance of '1NE86r4Esjf53EL7fR86CsfTZpNN42Sfab': 6
</code></pre>

Nothing is broken. Awesome!

Let’s also comment out the `bc.SignTransaction(&tx, wallet.PrivateKey)` call in `NewUTXOTransaction` to ensure that unsigned transactions cannot be mined:

<pre><code class="language-go hljs">&lt;span class="hljs-function">&lt;span class="hljs-keyword">func&lt;/span> &lt;span class="hljs-title">NewUTXOTransaction&lt;/span>&lt;span class="hljs-params">(from, to &lt;span class="hljs-keyword">string&lt;/span>, amount &lt;span class="hljs-keyword">int&lt;/span>, bc *Blockchain)&lt;/span> *&lt;span class="hljs-title">Transaction&lt;/span>&lt;/span> {
   ...
	tx := Transaction{&lt;span class="hljs-literal">nil&lt;/span>, inputs, outputs}
	tx.ID = tx.Hash()
	&lt;span class="hljs-comment">// bc.SignTransaction(&tx, wallet.PrivateKey)&lt;/span>

	&lt;span class="hljs-keyword">return&lt;/span> &tx
}
</code></pre>

<pre><code class="language-shell hljs">&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> go install&lt;/span>
&lt;span class="hljs-meta">$&lt;/span>&lt;span class="bash"> blockchain_go send -from 1AmVdDvvQ977oVCpUqz7zAPUEiXKrX5avR -to 1NE86r4Esjf53EL7fR86CsfTZpNN42Sfab -amount 1&lt;/span>
2017/09/12 16:28:15 ERROR: Invalid transaction
</code></pre>

## <span class="ez-toc-section" id="Conclusion"></span>Conclusion<span class="ez-toc-section-end"></span> {#conclusion}

It’s really awesome that we’ve got so far and implemented so many key features of Bitcoin! We’ve implemented almost everything outside networking, and in the next part, we’ll finish transactions.

Links:

  1. [Full source codes][8]
  2. [Public-key cryptography][9]
  3. [Digital signatures][10]
  4. [Elliptic curve][11]
  5. [Elliptic curve cryptography][12]
  6. [ECDSA][13]
  7. [Technical background of Bitcoin addresses][14]
  8. [Address][15]
  9. [Base58][16]
 10. [A gentle introduction to elliptic curve cryptography][5]

The original link：[https://jeiwan.cc/posts/building-blockchain-in-go-part-4/][17]

Chinese translations: [by liuchengxu][18], [by zhangli1][19]

 [1]: https://jeiwan.cc/posts/building-blockchain-in-go-part-4/
 [2]: https://github.com/Jeiwan/blockchain_go/compare/part_4...part_5#files_bucket
 [3]: https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
 [4]: https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki
 [5]: http://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/
 [6]: https://blockchain.info/
 [7]: https://en.bitcoin.it/wiki/File:Bitcoin_OpCheckSig_InDetail.png
 [8]: https://github.com/Jeiwan/blockchain_go/tree/part_5
 [9]: https://en.wikipedia.org/wiki/Public-key_cryptography
 [10]: https://en.wikipedia.org/wiki/Digital_signature
 [11]: https://en.wikipedia.org/wiki/Elliptic_curve
 [12]: https://en.wikipedia.org/wiki/Elliptic_curve_cryptography
 [13]: https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm
 [14]: https://en.bitcoin.it/wiki/Technical_background_of_version_1_Bitcoin_addresses
 [15]: https://en.bitcoin.it/wiki/Address
 [16]: https://en.bitcoin.it/wiki/Base58Check_encoding
 [17]: https://jeiwan.cc/posts/building-blockchain-in-go-part-3/
 [18]: https://github.com/liuchengxu/blockchain-tutorial/blob/master/content/part-5/address.md
 [19]: https://zhangli1.gitbooks.io/dummies-for-blockchain/content/