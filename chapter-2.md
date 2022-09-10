# Day 1

### 1. Deploy a contract to account 0x03 called "JacobTucker". Inside that contract, declare a constant variable named is, and make it have type String. Initialize it to "the best" when your contract gets deployed.
![image](https://user-images.githubusercontent.com/15671246/189290811-1a1d46cd-c048-42a6-9701-0da6e3265344.png)

### 2. Check that your variable is actually equals "the best" by executing a script to read that variable. Include a screenshot of the output.
![image](https://user-images.githubusercontent.com/15671246/189290937-00f38ff9-f218-46fe-994f-0435311e3834.png)

# Day 2

### 1. Explain why we wouldn't call `changeGreeting` in a script.

A script can only read or view data, hence the method `changeGreeting` which modifies data cannot be called. 

### 2. What does the `AuthAccount` mean in the `prepare` phase of the transaction?

`AuthAccount` is a type that represents the account sending the transaction and is used to access data stored in that account.   

### 3. What is the difference between the `prepare` phase and the `execute` phase in the transaction?

- `prepare` phase is to access the data stored in the account sending the transaction.
- `execute` phase is to call the functions to modify the data. It cannot directly access the account data.
- Technically, `prepare` phase can also call the functions to modify data and do everything itself. But the two phases are for separation of concerns and clarity.

### 4. 
- Add two new things inside your contract:
    - A variable named `myNumber` that has type `Int` (set it to 0 when the contract is deployed)
    - A function named `updateMyNumber` that takes in a new number named `newNumber` as a parameter that has type `Int` and updates `myNumber` to be `newNumber`
- Add a script that reads `myNumber` from the contract
- Add a transaction that takes in a parameter named `myNewNumber` and passes it into the `updateMyNumber` function. Verify that your number changed by running the script again.
![image](https://user-images.githubusercontent.com/15671246/189297584-eb59c1a1-c6c8-42c1-8a1a-c58a41cf17b6.png)
![image](https://user-images.githubusercontent.com/15671246/189297493-0532522a-678f-41c7-b94a-111dbfa17ed1.png)
![image](https://user-images.githubusercontent.com/15671246/189297639-fc490ee0-ef80-418e-b2d8-502ac6cc513a.png)

# Day 3

### 1. In a script, initialize an array (that has length == 3) of your favourite people, represented as `String`s, and `log` it.
![image](https://user-images.githubusercontent.com/15671246/189350792-eb129217-cd36-489c-a614-7c1d0677f4ff.png)

### 2. In a script, initialize a dictionary that maps the `String`s Facebook, Instagram, Twitter, YouTube, Reddit, and LinkedIn to a `UInt64` that represents the order in which you use them from most to least. For example, YouTube --> 1, Reddit --> 2, etc. If you've never used one before, map it to 0!
![image](https://user-images.githubusercontent.com/15671246/189350955-e8e54675-6f88-4bfa-9934-3c6ae46d001d.png)

### 3. Explain what the force unwrap operator `!` does, with an example different from the one I showed you (you can just change the type).

The force unwrap operator `!` is used to convert optional type to normal type. It operates on optional and either returns the value or `panic`s if the value is nil.
```cadence
var optionalNumber: UInt64? = 11
let number: UInt64 = optionalNumber! // Removes optional and returns UInt64
  
optionalNumber = nil
let nilNumber: UInt64 = optionalNumber! // PANICs with message "unexpectedly found nil while forcing an Optional value".
```
### 4. Using this picture below, explain...

#### - What the error message means

It means the return type of function is `String` but the `return` statement is returing an optional type `String?`.

#### - Why we're getting this error

Because the dictionary returns a value as an optional.

#### - How to fix it

Use unwrap operater `!`
```cadence
return thing[0x03]!
```

# Day 4

1. Deploy a new contract that has a Struct of your choosing inside of it (must be different than Profile).
2. Create a dictionary or array that contains the Struct you defined.
3. Create a function to add to that array/dictionary.
4. Add a transaction to call that function in step 3.
5. Add a script to read the Struct you defined.

![image](https://user-images.githubusercontent.com/15671246/189462680-29ee0788-a54f-4a53-b438-40084f760a62.png)
![image](https://user-images.githubusercontent.com/15671246/189462698-e251f6f5-1e82-409d-bd24-fea343b12f0e.png)
![image](https://user-images.githubusercontent.com/15671246/189462778-1fec3d96-8f02-4758-ab8d-cfc1bff73215.png)
