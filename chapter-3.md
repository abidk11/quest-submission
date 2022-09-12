# Day 1

### 1. In words, list 3 reasons why structs are different from resources.

- Structs can be copied. Resources cannot be copied.
- Structs can be lost or overwritten. Resources cannot be lost or overwritten.
- Structs can be created anywhere. Resources can only be created inside a contract.

### 2. Describe a situation where a resource might be better to use than a struct.

A resource is better for storing anything of value like an NFT or crypto.

### 3. What is the keyword to make a new resource?

`create`

### 4. Can a resource be created in a script or transaction (assuming there isn't a public function to create one)?

No, a resource can only be created inside a contract.

### 5. What is the type of the resource below?

```cadence
pub resource Jacob {

}
```
`@Jacob`

### 6. Let's play the "I Spy" game from when we were kids. I Spy 4 things wrong with this code. Please fix them.
```cadence
pub contract Test {

    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): Jacob { // there is 1 here
        let myJacob = Jacob() // there are 2 here
        return myJacob // there is 1 here
    }
}
```

Fixed code
```cadence
pub contract Test {
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }
    
    pub fun createJacob(): @Jacob { // Use @Jacob instead of Jacob for resource type.
        let myJacob <- create Jacob() // Use create keyword to create the resource and <- operator to move it.
        return <- myJacob // Use <- operator to return the resource.
    }
}    
```

# Day 2

### 1. Write your own smart contract that contains two state variables: an array of resources, and a dictionary of resources. Add functions to remove and add to each of them. They must be different from the examples above.

```cadence
pub contract PokeMania {

    pub var arrayOfPokemons: @[Pokemon]
    pub var dictionaryOfPokemons: @{UInt64: Pokemon}
	
    pub resource Pokemon {
        pub let id: UInt64
        pub let name: String

        init(_id: UInt64, _name: String) {
            self.id = _id
            self.name = _name
        }
    }

    pub fun addPokemonToArray(pokemon: @Pokemon) {
        self.arrayOfPokemons.append(<- pokemon)
    }

    pub fun removePokemonFromArray(index: Int): @Pokemon {
        return <- self.arrayOfPokemons.remove(at: index)
    }

    pub fun addPokemonToDictionary(pokemon: @Pokemon) {
        let oldPokemon <- self.dictionaryOfPokemons[pokemon.id] <- pokemon
        destroy oldPokemon
    }
    
    pub fun removePokemonFromDictionary(key: UInt64): @Pokemon {
        let pokemon <- self.dictionaryOfPokemons.remove(key: key) ?? panic("Could not find the pokemon!")
        return <- pokemon
    }
	
    pub fun getReference(key: UInt64): &Pokemon? {
        return &self.dictionaryOfPokemons[key] as &Pokemon?
    }

    init() {
        self.arrayOfPokemons <- []
        self.dictionaryOfPokemons <- {}
    }

}
```

# Day 3

### 1. Define your own contract that stores a dictionary of resources. Add a function to get a reference to one of the resources in the dictionary.

```cadence
pub contract PokeMania {

    pub var pokemons: @{UInt64: Pokemon}
	
    pub resource Pokemon {
        pub let id: UInt64
        pub let name: String

        init(_id: UInt64, _name: String) {
            self.id = _id
            self.name = _name
        }
    }

    pub fun addPokemonToDictionary(pokemon: @Pokemon) {
        let oldPokemon <- self.pokemons[pokemon.id] <- pokemon
        destroy oldPokemon
    }

    pub fun removePokemonFromDictionary(key: UInt64): @Pokemon {
        let pokemon <- self.pokemons.remove(key: key) ?? panic("Could not find the pokemon!")
        return <- pokemon
    }
	
    pub fun getReference(key: UInt64): &Pokemon {
        return (&self.pokemons[key] as &Pokemon?)!
    }

    init() {
        self.pokemons <- {
            1: <- create Pokemon(_id: 1, _name: "Pikachu")
        }
    }

}
```

### 2. Create a script that reads information from that resource using the reference from the function you defined in part 1.

```cadence
import PokeMania from 0x01

pub fun main(): String {
    let pokemonRef = PokeMania.getReference(key: 1)
    return pokemonRef.name
}
```

### 3. Explain, in your own words, why references can be useful in Cadence.

References are useful to access resources. Without references, we will have to move the resource everytime we want to access it. We will also have to move it back and make sure it is moved back in the right location, so additional work.  

# Day 4

### 1. Explain, in your own words, the 2 things resource interfaces can be used for (we went over both in today's content)

- To specify a set of requirements for resources. For ex, we can have an interface with a `name` field requirement. Any resource implementing this interface will have to have the `name` field.
- To restrict access to specific parts of a resource. For ex, suppose we have an interface with a `name` field requirement. And a resource implementing this interface with a method `updateName` to update the `name` field. We can restrict this resource to only expose the `name` field and not the `updateName` method using {RESOURCE_INTERFACE} notation.

### 2. Define your own contract. Make your own resource interface and a resource that implements the interface. Create 2 functions. In the 1st function, show an example of not restricting the type of the resource and accessing its content. In the 2nd function, show an example of restricting the type of the resource and NOT being able to access its content.

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

    pub fun noInterface() {
        let newPokemon: @Pokemon <- create Pokemon(_id: 1, _name: "Pikachu", _favouriteMove: 1)
        log(newPokemon.favouriteMove) // 1

        destroy newPokemon
    }

    pub fun yesInterface() {
        let newPokemon: @Pokemon{IPokemon} <- create Pokemon(_id: 1, _name: "Pikachu", _favouriteMove: 1)
        log(newPokemon.favouriteMove) // ERROR: `member of restricted type is not accessible: favouriteMove`

        destroy newPokemon
    }
}
```

### 3. How would we fix this code?
```cadence
pub contract Stuff {

    pub struct interface ITest {
        pub var greeting: String
        pub var favouriteFruit: String
    }

    // ERROR:
    // `structure Stuff.Test does not conform 
    // to structure interface Stuff.ITest`
    pub struct Test: ITest {
        pub var greeting: String

        pub fun changeGreeting(newGreeting: String): String {
            self.greeting = newGreeting
            return self.greeting // returns the new greeting
        }

        init() {
            self.greeting = "Hello!"
        }
    }

    pub fun fixThis() {
        let test: Test{ITest} = Test()
        let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") // ERROR HERE: `member of restricted type is not accessible: changeGreeting`
        log(newGreeting)
    }
}
```

Fixed code 
```cadence
pub contract Stuff {

    pub struct interface ITest {
        pub var greeting: String
        pub var favouriteFruit: String
    }

    pub struct Test: ITest {
        pub var greeting: String
        // Add favouriteFruit as it is required by interface.
        pub var favouriteFruit: String

        pub fun changeGreeting(newGreeting: String): String {
            self.greeting = newGreeting
            return self.greeting // returns the new greeting
        }
	
        init() {
            self.greeting = "Hello!"
            self.favouriteFruit = "Berries"
        }
    }

    pub fun fixThis() {
        // Remove restriction to interface as it does not allow changeGreeting method.
        let test: Test = Test()
        let newGreeting = test.changeGreeting(newGreeting: "Bonjour!")
        log(newGreeting)
    }
}
```

# Day 5

### For today's quest, you will be looking at a contract and a script. You will be looking at 4 variables (a, b, c, d) and 3 functions (publicFunc, contractFunc, privateFunc) defined in SomeContract. In each AREA (1, 2, 3, and 4), I want you to do the following: for each variable (a, b, c, and d), tell me in which areas they can be read (read scope) and which areas they can be modified (write scope). For each function (publicFunc, contractFunc, and privateFunc), simply tell me where they can be called.

```cadence
access(all) contract SomeContract {
    pub var testStruct: SomeStruct

    pub struct SomeStruct {

        //
        // 4 Variables
        //

        pub(set) var a: String

        pub var b: String

        access(contract) var c: String

        access(self) var d: String

        //
        // 3 Functions
        //

        pub fun publicFunc() {}

        access(contract) fun contractFunc() {}

        access(self) fun privateFunc() {}


        pub fun structFunc() {
            /**************/
            /*** AREA 1 ***/
            /**************/
        }

        init() {
            self.a = "a"
            self.b = "b"
            self.c = "c"
            self.d = "d"
        }
    }

    pub resource SomeResource {
        pub var e: Int

        pub fun resourceFunc() {
            /**************/
            /*** AREA 2 ***/
            /**************/
        }

        init() {
            self.e = 17
        }
    }

    pub fun createSomeResource(): @SomeResource {
        return <- create SomeResource()
    }

    pub fun questsAreFun() {
        /**************/
        /*** AREA 3 ****/
        /**************/
    }

    init() {
        self.testStruct = SomeStruct()
    }
}
```
```cadence
import SomeContract from 0x01

pub fun main() {
  /**************/
  /*** AREA 4 ***/
  /**************/
}
```

```cadence
pub(set) var a: String 
```
- Read scope: everywhere (AREA 1,2,3,4)
- Write scope: everywhere (AREA 1,2,3,4)

```cadence
pub var b: String 
```
- Read scope: everywhere (AREA 1,2,3,4)
- Write scope: Current & Inner (AERA 1)

```cadence
access(contract) var c: String 
```
- Read scope: Contianing contract (AERA 1,2,3)
- Write scope: Current & Inner (AERA 1)

```cadence
access(self) var d: String 
```
- Read scope: Current & Inner (AERA 1)
- Write scope: Current & Inner (AERA 1)

```cadence
pub fun publicFunc() {}	
```
everywhere (AREA 1,2,3,4)

```cadence
access(contract) fun contractFunc() {} 
```
Contianing contract (AERA 1,2,3)

```cadence
access(self) fun privateFunc() {} 
```
Current & Inner (AERA 1)
