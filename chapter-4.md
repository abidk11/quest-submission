#Day 1

### 1. Explain what lives inside of an account.

- **Contract Code:** All the deployed smart contracts are stored here.
- **Account Storage:** All the data lives here. It has 3 containers or locations to store data as per the accessibility.
	- `/storage/`: Only the account owner has access to this location.
	- `/public/`: Everyone has access to this location.
	- `/private/`: Only the account owner and accounts given explicit access by owner has access to this location.

### 2. What is the difference between the /storage/, /public/, and /private/ paths?

- `/storage/`: Only the account owner has access to this location. We need `AuthAccount` to access this location.
- `/public/`: Everyone has access to this location.
- `/private/`: Only the account owner and accounts given explicit access by owner has access to this location.

### 3. What does .save() do? What does .load() do? What does .borrow() do?

- `.save()`: This method saves a resource to the specified path in the `/storage/` location.
- `.load()`: This method loads or gets a resource from the specified path in the `/storage/` location.
- `.borrow()`: This method allows to look into a resource without moving it out of the account storage.

### 4. Explain why we couldn't save something to our account storage inside of a script.

Script can only read data, cannot write it. 

### 5. Explain why I couldn't save something to your account.

Only the owner has access to `/storage/` path where the data is stored. So, only the owner can save data. 

### 6. Define a contract that returns a resource that has at least 1 field in it. Then, write 2 transactions:

```cadence 
pub contract PokeMania {

    pub resource Pokemon {
        pub let id: UInt64
        pub let name: String

        init(_id: UInt64, _name: String) {
            self.id = _id
            self.name = _name
        }
    }

    pub fun createPokemon(id: UInt64, name: String): @Pokemon {
        return <- create Pokemon( _id: id, _name: name)
    }
}
```

### A transaction that first saves the resource to account storage, then loads it out of account storage, logs a field inside the resource, and destroys it.

```cadence
import PokeMania from 0x01

transaction() {

  prepare(acct: AuthAccount) {
    let pokemon <- PokeMania.createPokemon(id: 1, name: "Pikachu")
    acct.save(<- pokemon, to: /storage/pokemania)

    let loadedPokemon <- acct.load<@PokeMania.Pokemon>(from: /storage/pokemania) 
      ?? panic("No Pokemon found in pokemania :(")
    log(loadedPokemon.name)
    destroy loadedPokemon
  }

  execute {
    
  }
}
```

### A transaction that first saves the resource to account storage, then borrows a reference to it, and logs a field inside the resource.

```cadence
import PokeMania from 0x01

transaction() {

  prepare(acct: AuthAccount) {
    let pokemon <- PokeMania.createPokemon(id: 1, name: "Pikachu")
    acct.save(<- pokemon, to: /storage/pokemania)

    let loadedPokemon = acct.borrow<&PokeMania.Pokemon>(from: /storage/pokemania) 
      ?? panic("No Pokemon found in pokemania :(")
    log(loadedPokemon.name)
  }

  execute {
    
  }
}
```

# Day 2

### 1. What does .link() do?

`.link()` creates a link or a pointer to resource stored in the `/storage/` path. Actually, the data is only stored in the `/storage/` path and the links are stored in `/public/` and `/private/` paths which allow others to access the data. These links are called capabilities. 

### 2. In your own words (no code), explain how we can use resource interfaces to only expose certain things to the /public/ path.

As we know, we can create a more restrictive type of a resource by casting it to a resource interface containing only the allowed fields or methods. We can use this to restrict the capability when creating a link. We can do this by mentioning the resource interface along with the resource type in the link method. 
  
### 3. Deploy a contract that contains a resource that implements a resource interface. Then, do the following:

```cadence
pub contract PokeMania {

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
        }
    }

    pub fun createPokemon(id: UInt64, name: String, favouriteMove: UInt64): @Pokemon {
        return <- create Pokemon( _id: id, _name: name, _favouriteMove: favouriteMove)
    }
}
```

### i. In a transaction, save the resource to storage and link it to the public with the restrictive interface.

```cadence
import PokeMania from 0x01

transaction() {

  prepare(acct: AuthAccount) {
    let pokemon <- PokeMania.createPokemon(id: 1, name: "Pikachu", favouriteMove: 1)
    acct.save(<- pokemon, to: /storage/pokemania)
    acct.link<&PokeMania.Pokemon{PokeMania.IPokemon}>(/public/pokemania, target: /storage/pokemania)
    log("Pikachu stored and linked!")
  }

  execute {
    
  }
}
```

### ii. Run a script that tries to access a non-exposed field in the resource interface, and see the error pop up.

```cadence
import PokeMania from 0x01

pub fun main(address: Address): UInt64 {
  let publicCapability: Capability<&PokeMania.Pokemon{PokeMania.IPokemon}> =
    getAccount(address).getCapability<&PokeMania.Pokemon{PokeMania.IPokemon}>(/public/pokemania)

  let pokemon: &PokeMania.Pokemon{PokeMania.IPokemon} = publicCapability.borrow() 
    ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  // ERROR: "member of restricted type is not accessible: favouriteMove"
  return pokemon.favouriteMove
}
```

### iii. Run the script and access something you CAN read from. Return it from the script.

```cadence
import PokeMania from 0x01

pub fun main(address: Address): String {
  let publicCapability: Capability<&PokeMania.Pokemon{PokeMania.IPokemon}> =
    getAccount(address).getCapability<&PokeMania.Pokemon{PokeMania.IPokemon}>(/public/pokemania)

  let pokemon: &PokeMania.Pokemon{PokeMania.IPokemon} = publicCapability.borrow() 
    ?? panic("The capability doesn't exist or you did not specify the right type when you got the capability.")

  return pokemon.name
}
```
	
# Day 3

### 1. Why did we add a Collection to this contract? List the two main reasons.

- We can store only one resource at one path. So, without collection, we will have to store each NFT at different path and will have remember/ manage that many path.
- Only account owner can access their storage directly. So, others won't be able to store or give us any NFT. 

### 2. What do you have to do if you have resources "nested" inside of another resource? ("Nested resources")

Whenever we have resources "nested" inside of another resource, we need to define a `destroy()` function to destory all the nested resources with the `destroy` keyword.

### 3. Brainstorm some extra things we may want to add to this contract. Think about what might be problematic with this contract and how we could fix it.
### Idea #1: Do we really want everyone to be able to mint an NFT? 🤔.
	
No, that will allow anyone to mint and claim as many NFTs as one want. Instead, we can have a resource interface to give mint capability to only those who should have it.
	
### Idea #2: If we want to read information about our NFTs inside our Collection, right now we have to take it out of the Collection to do so. Is this good?

No, moving a resource just to peek into it is not a good idea. Instead, we should use a reference.

# Day 4

### 1. Because we had a LOT to talk about during this Chapter, I want you to do the following:
### Take our NFT contract so far and add comments to every single resource or function explaining what it's doing in your own words. Something like this:

```cadence
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains a name,
  // favouriteFood, and luckyNumber
  pub resource NFT {
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

  // This is a resource interface that allows us to
  // 1. add an NFT to the collection
  // 2. get list of id of NFTs in the collection
  // 3. get reference to an NFT in the collection
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }
  
  // This is resource that implements the above interface.
  // In addition, it also allows to
  // 1. withdraw an NFT from the collection
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    // This is a function to add an NFT in the collection.
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

	  // This is a function to remove an NFT from the collection
	  // and return it.
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // This is a function to get list of id of NFTs in the collection.
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }
    
    // This is a function to get reference to an NFT in the collection. 
    pub fun borrowNFT(id: UInt64): &NFT {
      return (&self.ownedNFTs[id] as &NFT?)!
    }
    
    // This function initializes the internal NFT dictionary.
    init() {
      self.ownedNFTs <- {}
    }

    // This function destroys the internal NFT dictionary.
    destroy() {
      destroy self.ownedNFTs
    }
  }

  // This function creates the Collection resource 
  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  // This is a resource that allows anyone having it to mint the NFT.
  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    // This function creates the Minter resource
    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  // This function 
  // - sets the total supply value
  // - creates the Minter resource and saves it in the owner account
  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```
