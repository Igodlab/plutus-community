# Lecture 2: Validation scripts

### Credits

[Alberto Calzada - albertoCCz](https://github.com/albertoCCz)

## Validator
The _Validator_ is a function that takes three arguments, _Datum_, _Redeemer_ and _Context_, which are of type _Data_, and returns the _Unit_ type (which reads `()` on Haskell) or a _Bool_ type (maybe others, although I think it would make no sense). As an example:

```haskell
mkValidator :: Data -> Data -> Data -> ()
mkValidator _ _ _ = ()
```

This Validator will always accept any transaction, because no matter what arguments it takes (that's why we use the `_`) that the output will be of Unit type, so the transaction will succeed. You can, of course, change the logic of the validator so it matches your business requirements. As an example:

```haskell
mkValidator :: Data -> Data -> Data -> ()
mkValidator _ r _
	| r == I 42 = ()
    	| otherwise = traceError "wrong redeemer"
```

(This won't probably match anyone's business requirements, but it is bit more complex than the previous one ;))

As we can see, we have implemented some simple logic for the redeemer. Now, we first check if `r == I 42` and if it is true we returns `()`, the Unit type,  and the transaction is done. In any other case, we raise an error with help of the `traceError` function, which shows the string we pass it as the error message on the console.


*Quick note: `I 42` is just an object of type _Data_, as `I` is one of the possible constructors of this data type (reference: [Data type](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/Data.hs))



## Validator Script

To translate this validator Haskell function to an actual Plutus validator, we need to compile it. Then it will actually be of type _Validator_. To this end we use the function `mkValidatorScript`, which takes as an argument something of type _CompiledCode_ of `(Data -> Data -> Data -> ())`. As an example:

```haskell
validator :: Validator
validator = mkValidatorScript $$(PlutusTx.compile [|| mkValidator ||])
```

*Quick note: the line `{-# INLINABLE mkValidator #-}` (called _inlinable pragma_) that appears over the definition of the validator function tells the Plutus compiler to inline the validator function logic.  This allows the compiler to use it as a valid argument for the `PlutusTx.compile` function and to effectively compile it.


## Validator's Hash and Script's Address

After building our validator script, we must generate its _Hash_ and _Address_. The necessary code to do it is just boiler plate: it will always be almost the same for every contract we write in Plutus. The code explains itself.

+ Hash:
```haskell
valHash :: Ledger.ValidatorHash
valHash = Scripts.validatorHash validator
```

+ Address:
```haskell
scrAddress :: Ledger.Address
scrAddress = ScriptAddress valHash
```

## Typed Validator

It is possible to substitute the _Data_ type arguments of the validator by more suitable types of data. We can, for example, rewrite the second validator function we saw previously by:

```haskell
mkValidator :: () -> Integer -> ValidatorCtx -> Bool
mkValidator () r _
	| r == 42   = True
    	| otherwise = False
```
The Datum and Redeemer types _Unit_ and _Integer_, respectively, are the custom ones, while the Context _ValidatorCtx_ and the _Bool_ types are the ones will usually use in most cases. As this is the case, the Plutus team has taken care of it and the only adjustments we need to do are those related with the Datum and the Redeemer. This is because, as we have already seen, the compiler expects a validator of type `Data -> Data -> Data -> ()`, so we must add some code to do the trick. This code, which is also boiler plate, reads:

```haskell
data Typed
instance Scripts.ScriptType Typed where
	type instance DatumType Typed    = ()
    	type instance RedeemerType Typed = Integer
```

To compile this typed version of the validator we, again, have to make use of some boiler plate code. I don't understand this code very well, so I'll just copy-paste it:

```haskell
inst :: Scripts.ScriptInstance Typed
inst = Scripts.validator @Typed
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
  where
    wrap = Scripts.wrapValidator @() @Integer
```

One thing worth mentioning here is the last line of code. When defining `wrap` you use the function `wrapValidator` from module `Scripts` and you pass it the _custom_ data types you used in the validator function (preceded by `@`) as parameters. In this case, those data types were the `Unit` type for _Datum_ and the `Integer` type for the _Redeemer_. Then, we just need to change a bit the validator function to convert this compiled code to a _Validator_ type object:

```haskell
validator :: Validator
validator = Scripts.validatorScript inst
```

As we can see we have just passed `inst` to a new function called `validatorScript` from module `Script` without the need to compile it, as we already did it on the previous piece of code.

In principle, the data types we can successfully use when defining the validator are those defined as instances of the [isData class](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/IsData/Class.hs). This is due to the fact that is this class the one in charge to convert our custom data types in objects of _Data_ type. It does so be means of the methods `toData` and `fromData`. Anyway, if we want to use different data types from those instantiated in the referred link, we just need to define this instances. But this might be a very tedious process, so Plutus give us a convenient way to do it easily. For example, if we want to use some custom data type `fabulousRedeemerType`, we just have to add on top of our validator function these lines:

```haskell
newtype fabulousRedeemerType = fabulousRedeemerType Integer
	deriving Show

PlutusTx.unstableMakeIsData ''fabulousRedeemerType
```

The first two lines define my type while in the fourth one we pass this type to the helper function `unstableMakeIsData` as its argument, for which we use two single quotes, and end up having instantiated our new type.














# Lecture 02 Detailed 

## Credits

Condensed version of Lecture #2 of the Plutus Pioneer Program by Lars Brünjes on [Youtube](https://youtu.be/E5KRk5y9KjQ)
Cloned from [Reddit (u/RikAlexander)](https://www.reddit.com/r/cardano/comments/mumpdx/week_02_plutus_pioneer_program/)

## Setup

1 - First get the Local Playground up and running

Check the sidebar for instructions on how to do this.

2 - Second you need the Git Repository of the Plutus Pioneer Program

Also check the sidebar for instructions if needed.

3 - We'll start with the first example of the week02 Video.

    plutus-pioneer-program/code/week01/src/Week02/Gift.hs

open this file in your favorite editor (VS code of course)

4 - copy all contents, and plug it into the playground

this should compile, and we can go into the simulation (the nice blue button with "simulate" written in it)

## Gift

1 - The Script is called "Gift" for a reason. Wallet01 will be gifting the script some amount of Lovelaces. Wallet02 will try to "grab" the funds locked in this script.

Try it. Make Wallet01 give 3 lovelaces, and make Wallet02 grab them.

![Gift Actions](https://preview.redd.it/as1zvr8ctau61.png?width=646&format=png&auto=webp&s=aa7f4f965cb96fa88de3c75f27a10b9bb603eff6)

2 - As you can see the funds have changed to Wallet01 -> 7 Lovelaces, Wallet02 -> 13 Lovelaces

![Gift Final Balances](https://preview.redd.it/jvv5jebetau61.png?width=661&format=png&auto=webp&s=261125bd456105f145b45c9460124ebabc80d850)

Now at this point, of course anyone could grab the funds. Which (normally) you wouldn't want. This is where the Validator comes in to play.

## mkValidator

1 - The main topic of the Lecture was the so called "Validator"

Lines 30 - 32 in the Gift.hs file

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: Data -> Data -> Data -> ()
mkValidator _ _ _ = ()
```

This is the Haskell syntax for essentially a "function".

mkValidator (or Make Validator), is the name of the function

Which has 3 parameters and 1 return type.

```haskell
mkValidator :: Data -> Data -> Data -> ()
```

The return type: () is called a "Unit". But you can more or less think of this as "void".

If we were to write this in "normal" code; it would look something like:

```haskell
public void mkValidator(Data x, Data y, Data z) {}
```

2 - Now for the implementation

```haskell
mkValidator _ _ _ = ()
```

The Three underscores are for Data 1-3. It basically means that we don't care what will happen to them. As we will always just return "Unit".

We could also write it as:

```haskell
mkValidator x y z = ()
```

But \_ is usually used for parameters we don't care about.

3 - The Inlinable Pragma

```haskell
{-# INLINABLE mkValidator #-}
```

This is there to let the compiler know that the function mkValidator may be used as an inline function. (We'll get back to this in a second)

4 - The "Data" datatype. What is it?

Well it can be all sorts of things. It can be a Map, a List, a simple Integer (I) or a string. It all depends on how you "construct" it. It's "real" name is: PlutusTx.Data in the [Data.hs](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/Data.hs) file in the Plutus Repo where we can read more about the Data type.

5 - The 3 Parameters. (x, y, z) They are the Datum, Redeemer and Context.

Datum - arbitrary data; can be used to provide more information to the Validator script (which in turn e.g. could also be used to be validated against the Redeemer)

Redeemer - arbitrary data; used to validate if the user calling the script may unlock the funds or not. (e.g. a password for unlocking)

Context - This holds information about the Transaction

## Validator Script

Lines 34 - 35 of Gift.hs

```haskell
validator :: Validator
validator = mkValidatorScript $$(PlutusTx.compile [|| mkValidator ||])
```

To get our mkValidator to work on the blockchain, it needs to be compiled to Plutus core.

1 - To do this, we'll use something called `Oxford Brackets`.

In short the PlutusTx.compile function needs an inline function; but this can be hard (if not impossible) to write for bigger functions. This is why we need the Inlinable Pragma `{-# INLINABLE mkValidator #-}` It tells the compiler that mkValidator may be used as an inline function. The `[|| mkValidator ||]` "inlines" the function, and hands it over to PlutusTx.compile.

PlutusTx.compile returns the Plutus Core compiled script, so that `mkValidatorScript` can read it, and make the validator blockchain-ready.

## Script Address

We need to generate our Script Address.

1 - To do that we'll generate a Hash based on the `validator` script that we just created.

Lines 37 - 38 of Gift.hs

```haskell
valHash :: Ledger.ValidatorHash
valHash = Scripts.validatorHash validator
```

This is pretty self-explanatory; `valHash` is of type `Ledger.ValidatorHash`

`Scripts.validatorHash` will hash our validator to a `Ledger.ValidatorHash` type Hash

2 - Now that we have the Hash we can generate our Address!

Lines 40 - 41 of Gift.hs

```haskell
scrAddress :: Ledger.Address
scrAddress = ScriptAddress valHash
```

Same here: `scrAddress` is of type Ledger.Address `ScriptAddress` will generate our address based on the `valHash` variable.

## The Wallet code

From line 43 in Gift.hs

This was not covered in the Lecture; we'll probably go more in depth in Week03.

1 - The first few lines though are easy to understand:

Line 43 - 46 in Gift.hs

```haskell
type GiftSchema =
BlockchainActions
    .\/ Endpoint "give" Integer
    .\/ Endpoint "grab" ()
```

This defines our endpoints (which you can both see in the playground)

The `give` endpoint receives an Integer.

And the `grab` endpoint does not receive anything: Type Unit (Void)

## Error Handling

Open the file

    plutus-pioneer-program/code/week01/src/Week02/Burn.hs

This script is almost identical to Gift.hs, but now the Validator looks like this

Lines 31 - 33 in Burn.hs

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: Data -> Data -> Data -> ()
mkValidator _ _ _ = traceError "NO WAY!"
```

As you can see the validator always returns the error "NO WAY!".

The reason this file is called "Burn" is because you can still send funds into the script with the `Give` endpoints, although you will never be able to `Grab` them.

Essentially this will `Burn` the funds send to this script, as it will receive but never release them.

*Note:* Try it in the Playground to help you get used to the playground workflow

## FortyTwo

Burning funds is (probably) not what we want to do. In this example by Lars we will implement a simple Validator based on user input (the Redeemer)

Open the file

    plutus-pioneer-program/code/week01/src/Week02/FortyTwo.hs

And go and check out our new Validator

Lines 31 - 35 in FortyTwo.hs

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: Data -> Data -> Data -> ()
mkValidator _ r _
    | r == I 42 = ()
    | otherwise = traceError "wrong redeemer"
```

This time we'll validate the Redeemer and check if it equals `42` (hence the name of the script).

As you may recall we have the 3 Data type arguments, which equal to:

Datum, Redeemer and Context

Specifically the Redeemer argument is now set to the variable `r`

```haskell
mkValidator _ r _
```

By using [Guards](https://www.futurelearn.com/info/courses/functional-programming-haskell/0/steps/27226) (a Haskell kind of if-else statement)

We'll check if the Redeemer `r` is equal to 42, otherwise we throw the error "wrong redeemer"

Note: as you can see we use `I 42` and not just 42. This is because I is a constructor for a [Data](https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/Data.hs) type.

## Typed

Open the file

    plutus-pioneer-program/code/week01/src/Week02/Typed.hs

Again check our Validator

Lines 32 - 36 in Typed.hs

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: () -> Integer -> ValidatorCtx -> Bool
mkValidator () r _
    | r == 42   = True
    | otherwise = False
```

The big difference here is, instead of specifying the 3 arguments as Data.

We implicitly specify which type we want/expect to receive. In this case:

Datum -> () = Unit = (Void)

Redeemer -> Integer

Context -> ValidatorCtx

Also the Validator is now returning a Bool. (True -> Success! False -> :( )

*Note:* as you can see we don't run our magic number 42 through the `I` constructor anymore `r == 42`, because now `r` is already just an Integer; no need to compare it to another Data type.

Now the lines 38 - 48 in Typed.hs are more or less copy-paste.

```haskell
data Typed
instance Scripts.ScriptType Typed where
    type instance DatumType Typed = ()
    type instance RedeemerType Typed = Integer

inst :: Scripts.ScriptInstance Typed
inst = Scripts.validator @Typed
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
where
    wrap = Scripts.wrapValidator @() @Integer
```

The most important thing here are the lines

```haskell
type instance DatumType Typed = ()
type instance RedeemerType Typed = Integer
```

These lines will set the Datum to Type () = Unit = (Void) and the Redeemer to Type Integer.

Now we have a new instance of a validator only with strict types.

## Custom data types

Open the file:

    plutus-pioneer-program/code/week01/src/Week02/IsData.hs

Let's have a look at our Validator first:

Lines 37 - 39 of IsData.hs

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: () -> MySillyRedeemer -> ValidatorCtx -> Bool
mkValidator () (MySillyRedeemer r) _ = traceIfFalse "wrong redeemer" $ r == 42
```

We now have a new type for our Redeemer: MySillyRedeemer!

Instead of Guards, we now have the traceIfFalse function. The dollar sign basically says: everything behind me should first be evaluated, then passed on to the traceIfFalse function.

So r == 42 is evaluated to either True or False, is then passed on to the traceIfFalse function, which when False will throw the error "wrong redeemer".

Specified on lines 32 - 33 of `IsData.hs` is our new Type

```haskell
newtype MySillyRedeemer = MySillyRedeemer Integer
    deriving Show
```

What we're doing here is creating a new type (`newtype`) with the name `MySillyRedeemer` which acts and does the same as `Integer` (get the Silly part of the name now? :D )

`deriving Show` is Haskell syntax, and means that `MySillyRedeemer` now also inherits the Show functionality, which basically means we can now `Show` or `print` MySillyRedeemer.

Line 35 of IsData.hs

```haskell
PlutusTx.unstableMakeIsData ''MySillyRedeemer
```

Basically says: make MySillyRedeemer of type Data. (which we need for using it as Redeemer)

Now the validator will be happy to receive our MySillyRedeemer as Redeemer argument.

Lines 41 - 51 of IsData.hs

```haskell
data Typed
instance Scripts.ScriptType Typed where
    type instance DatumType Typed = ()
    type instance RedeemerType Typed = MySillyRedeemer

inst :: Scripts.ScriptInstance Typed
inst = Scripts.validator @Typed
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
where
    wrap = Scripts.wrapValidator @() @MySillyRedeemer
```

Compare them with Typed.hs; Integer is now changed to MySillyRedeemer.

## Homework 1

Open the file:

    plutus-pioneer-program/code/week01/src/Week02/Homework1.hs

1 - Copy everything into the Playground

Our homework is it to fix the lines 32 - 51

```haskell
{-# INLINABLE mkValidator #-}
-- This should validate if and only if the two Booleans in the redeemer are equal!
mkValidator :: () -> (Bool, Bool) -> ValidatorCtx -> Bool
mkValidator _ _ _ = True -- FIX ME!

data Typed
instance Scripts.ScriptType Typed where
-- Implement the instance!

inst :: Scripts.ScriptInstance Typed
inst = undefined -- FIX ME!

validator :: Validator
validator = undefined -- FIX ME!

valHash :: Ledger.ValidatorHash
valHash = undefined -- FIX ME!

scrAddress :: Ledger.Address
scrAddress = undefined -- FIX ME!
```

We'll start with the first "fix me". The validator.

The validator will now receive a Redeemer of (Bool, Bool) which is a [Tuple](http://www.cantab.net/users/antoni.diller/haskell/units/unit07.html).

Easy way to look at it: it's an array of 2 Booleans.

2 - To fix the first "fix me" we'll need to check if these two Booleans are Equal.

To extract both Booleans we can just assign them to 2 variables, compare them, and return either True or False.

```haskell
mkValidator () (a, b) _ = a == b
```

This will work fine, but what we learned in IsData.hs is the traceIfFalse function. We can implement it, for a cleaner code with some error reporting:

```haskell
mkValidator () (a, b) _ = traceIfFalse "wrong redeemer" $ a == b
```

This was the hard part of Homework1.

3 - Now it's copy and paste from either IsData.hs or Typed.hs, only with the `(Bool, Bool)` tuple type, instead of mySillyRedeemer or Integer.

```haskell
data Typed
instance Scripts.ScriptType Typed where
    type instance DatumType Typed = ()
    type instance RedeemerType Typed = (Bool, Bool)

inst :: Scripts.ScriptInstance Typed
inst = Scripts.validator @Typed
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
    where
        wrap = Scripts.wrapValidator @() @(Bool, Bool)

validator :: Validator
validator = Scripts.validatorScript inst

valHash :: Ledger.ValidatorHash
valHash = Scripts.validatorHash validator

scrAddress :: Ledger.Address
scrAddress = ScriptAddress valHash

type GiftSchema =
    BlockchainActions
        .\/ Endpoint "give" Integer
        .\/ Endpoint "grab" (Bool, Bool)
```

We can now compile and simulate our script.

Set up the actions like so:

![Homework 1 Actions](https://preview.redd.it/7tdo1n99tau61.png?width=645&format=png&auto=webp&s=19bc99a790439d2cf9b2703fd2fc9bab4a0d234a)

And play with the `Grab` action; if both booleans are equal, the grab will succeed, otherwise it will fail.

## Homework 2

If you made it all the way down here, Homework 2 is a walk in the park.

1 - Copy everything from the file into the playground:

    plutus-pioneer-program/code/week01/src/Week02/Homework2.hs

First check the validator

Lines 43 - 46 in Homework2.hs

```haskell
{-# INLINABLE mkValidator #-}
-- This should validate if and only if the two Booleans in the redeemer are equal!
mkValidator :: () -> MyRedeemer -> ValidatorCtx -> Bool
mkValidator _ _ _ = True -- FIX ME!
```

Which is basically the same as for homework 1, only difference is that now we are passing a custom type as Redeemer `MyRedeemer`.

This is defined on lines 36 - 39 in Homework2.hs

```haskell
data MyRedeemer = MyRedeemer
    { flag1 :: Bool
    , flag2 :: Bool
    } deriving (Generic, FromJSON, ToJSON, ToSchema)
```

2 - Basically all we need to do is change our validator to compare both Booleans in the MyRedeemer type

```haskell
mkValidator :: () -> MyRedeemer -> ValidatorCtx -> Bool
mkValidator () (MyRedeemer a b) _ = a == b
```

and same as for Homework1 we'll use the traceIfFalse function here

```haskell
mkValidator () (MyRedeemer a b) _ = traceIfFalse "wrong redeemer" $ a == b
```

That's it.

3 - For the last few "fix me", it's copy and paste time again from either IsData.hs or Typed.hs or even Homework1.hs. Only now with the MyRedeemer type instead of Integer, (Bool, Bool) or mySillyRedeemer

```haskell
data Typed
instance Scripts.ScriptType Typed where
    type instance DatumType Typed = ()
    type instance RedeemerType Typed = MyRedeemer

inst :: Scripts.ScriptInstance Typed
inst = Scripts.validator @Typed
    $$(PlutusTx.compile [|| mkValidator ||])
    $$(PlutusTx.compile [|| wrap ||])
    where
        wrap = Scripts.wrapValidator @() @MyRedeemer

validator :: Validator
validator = Scripts.validatorScript inst

valHash :: Ledger.ValidatorHash
valHash = Scripts.validatorHash validator

scrAddress :: Ledger.Address
scrAddress = ScriptAddress valHash

type GiftSchema =
    BlockchainActions
        .\/ Endpoint "give" Integer
        .\/ Endpoint "grab" MyRedeemer
```

We're all set!

![Homework 2 Actions](https://preview.redd.it/n1iip6n6tau61.png?width=647&format=png&auto=webp&s=ff22b62b022df9c396b97387ff1a97186d3fac45)

Admire your work in the Playground. Play with it. Test it. Happy Coding! 😊

## Credits

Condensed version of Lecture #2 of the Plutus Pioneer Program by Lars Brünjes on [Youtube](https://youtu.be/E5KRk5y9KjQ)
Cloned from [Reddit (u/RikAlexander)](https://www.reddit.com/r/cardano/comments/mumpdx/week_02_plutus_pioneer_program/)

