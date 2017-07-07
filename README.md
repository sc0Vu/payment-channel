# payment-channel

Ethereum Payment Channel in 50 lines of code

# Decodes in lines

Channel constructor

	  function Channel(address to, uint timeout) payable {
	  	channelRecipient = to;
	  	channelSender = msg.sender; // set channelSender as sender of the message
	  	startDate = now; // set startDate as current timestamp
	  	channelTimeout = timeout;
	  }

Close channel

	  function CloseChannel(bytes32 h, uint8 v, bytes32 r, bytes32 s, uint value){

	  	address signer;
	  	bytes32 proof;

	  	// get signer from signature
	  	// ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address): recover address associated with the public key from elliptic curve signature, return zero on error
	  	signer = ecrecover(h, v, r, s);

	  	// signature is invalid, throw
	  	if (signer != channelSender && signer != channelRecipient) throw;

      // keccak256(...) returns (bytes32): compute the Ethereum-SHA-3 (Keccak-256) hash of the (tightly packed) arguments
      // sha3(...) returns (bytes32): an alias to keccak256()
	  	proof = sha3(this, value);

	  	// signature is valid but doesn't match the data provided
	  	if (proof != h) throw;

	  	if (signatures[proof] == 0)
	  		signatures[proof] = signer;
	  	else if (signatures[proof] != signer){
	  		// channel completed, both signatures provided
	  		if (!channelRecipient.send(value)) throw;
	  		selfdestruct(channelSender);
	  	}

	  }

Chanel timeout

	  function ChannelTimeout(){
	  	if (startDate + channelTimeout > now) // check timeout
	  		throw;

	  	selfdestruct(channelSender);
	  }

# 

The story is Bob want to pay Alice if she do her work well.

And Bob doesn't want to pay her first, he want to pay when he check her work.

In this situation, he can create a channel to Alice.


After Alice done her work, she can close chanel with her work and value.

With just only one person close channel, Alice cannot withdraw her coin.

She can do that if Bob close channel too.


After Bob checkout her work and close channel with value.

Finally Alice get her coin.