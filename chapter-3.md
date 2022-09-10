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

A resource can only be created inside a contract.

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

```cadence
    pub fun createJacob(): @Jacob { 
        let myJacob <- create Jacob()
        return <- myJacob 
    }
```
