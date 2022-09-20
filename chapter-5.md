# Day 1

### 1. Describe what an event is, and why it might be useful to a client.

- An event is a way to communicate to the outside world that something happened in the contract. 
- A client can know important things happening inside the contract without repeteadly querying the contract. It can use this info to execute some business logic or update the UI. 

### 2. Deploy a contract with an event in it, and emit the event somewhere else in the contract indicating that it happened.

```cadence
pub contract PokeMania {

    pub event PokemonCreated(id: UInt64)

    pub resource interface IPokemon {
        pub let id: UInt64
        pub let name: String
    }

    pub resource Pokemon: IPokemon {
        pub let id: UInt64
        pub let name: String
        pub let favouriteMove: UInt64

        init(_id: UInt64, _name: String, _favouriteMove: UInt64) {
            self.id = _id
            self.name = _name
            self.favouriteMove = _favouriteMove

            emit PokemonCreated(id: self.id)
        }
    }
	
}
```

### 3. Using the contract in step 2), add some pre conditions and post conditions to your contract to get used to writing them out.

```cadence
pub contract PokeMania {

    pub event PokemonCreated(id: UInt64)

    pub resource interface IPokemon {
        pub let id: UInt64
        pub let name: String
    }

    pub resource Pokemon: IPokemon {
        pub let id: UInt64
        pub let name: String
        pub let favouriteMove: UInt64

        init(_id: UInt64, _name: String, _favouriteMove: UInt64) {
            pre {
                _name.length > 0: "_name is not valid!"
                _favouriteMove > 0 && _favouriteMove < 100: "_favouriteMove is not valid!" 
            }

            post {
                self.name == _name
                self.favouriteMove == _favouriteMove
            }

            self.id = _id
            self.name = _name
            self.favouriteMove = _favouriteMove

            emit PokemonCreated(id: self.id)
        }
    }
    
}
```

### 4. For each of the functions below (numberOne, numberTwo, numberThree), follow the instructions.
```cadence
pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name.
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // TODO
  // Tell me whether or not this function will return a value.
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number.
    // Also, tell me the value of `self.number` after it's run.
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      return self.number
    }

    init() {
      self.number = 0
    }

  }

}
```

- **numberOne:** Yes, `'Jacob'.length` is 5.
- **numberTwo:** Yes, `'Jacob'.length` is greater than 0 and `name.concat(" Tucker")` is `Jacob Tucker`.
- **numberThree:** No, the post condition fails (`before(self.number)` is 0, result is 1, so `before(self.number) == result + 1` becomes `0 == 1 + 1` which is false) and the value of `self.number` remains unchanged, 0. 

# Day 2

### 1. Explain why standards can be beneficial to the Flow ecosystem.

- Standards help us to know what a contract is without reading its code. For ex, we know any contract that implements NonFungibleToken standard is an NFT.
- Standards help a client using multiple contracts to have same code to interact with all of them. For ex, all NFT contracts have a Collection resource with `deposit` and `withdraw` methods as per the NonFungibleToken standard. Client just need to have code specific to this standard and it will work for all the NFTs.

### 2. What is YOUR favourite food?

Thanks for asking! Biryani (Mutton/ Chicken/ Prawn)

### 3. Please fix this code (Hint: There are two things wrong):
```cadence
pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff {
    pub var favouriteActivity: String
  }
}

pub contract Test {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```

```cadence
// No changes in ITest interface

// Implement interface ITest
pub contract Test: ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  // Remove IStuff interface

  // Implement interface ITest.IStuff
  pub resource Stuff: ITest.IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```

# Day 3

### 1. What does "force casting" with `as!` do? Why is it useful in our Collection?

- "Force casting" with `as!` downcasts a generic type to a more specific type. It panics if the type cannot be casted.
- We want to store our NFTs in our Collection. But since our contract implements the NonFungibleToken standard, our `deposit` methods has generic NFT param allowing any NFT to be stored. `as!` allows us to check and restrict the param to our specific NFT.

### 2. What does `auth` do? When do we use it?

- `auth` allows us to get an "authorized reference" to the generic reference type by putting `auth` in front of it.
- We use it to downcast a generic reference type to a specific reference type.

### 3. This last quest will be your most difficult yet. Take this contract:
```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```
### and add a function called borrowAuthNFT just like we did in the section called "The Problem" above. Then, find a way to make it publically accessible to other people so they can read our NFT's metadata. Then, run a script to display the NFTs metadata for a certain id.
### You will have to write all the transactions to set up the accounts, mint the NFTs, and then the scripts to read the NFT's metadata. We have done most of this in the chapters up to this point, so you can look for help there :)

Contract
```cadence
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  // Add a new interface with the function to borrow the auth NFT.
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NonFungibleToken.NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT
    pub fun borrowAuthNFT(id: UInt64): &NFT
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic, CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    // Implement the function to borrow the auth NFT.
    pub fun borrowAuthNFT(id: UInt64): &NFT {
      let ref = (&self.ownedNFTs[id] as auth &NonFungibleToken.NFT?)!
      return ref as! &NFT;
    }
        
    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

Transaction to create collection
![image](https://user-images.githubusercontent.com/15671246/191241743-4bbce7fa-9c47-4dcb-84d6-08dc84a905f2.png)

Transaction to create NFT
![image](https://user-images.githubusercontent.com/15671246/191241921-32ef4b20-e1a4-4ea2-80ab-e1c505ccd064.png)

Script to get NFT metadata
![image](https://user-images.githubusercontent.com/15671246/191242050-01307d4b-e9ad-43f2-94c2-3984f881b563.png)
