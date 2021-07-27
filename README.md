# cardano-plutus-pioneer-week03
##Homework for the Cardano plutus pioneer program

Homework #1.
-----------------------------------------------
When wallet1 give "gifts" to wallet1 and wallet2 (including a deadLine):
Gifts to wallet2 should only be able to grab before the deadLine.
Gifts to wallet3 should only be allowed to grab (by wallet3) after the deadline.

My first understanding of the task was that we only should change the on-chain validator. The solution seemed to be pretty straight forward.
My solution included changing the signature of the "signedByBeneficiary" method allowing it to check for arbitrary pubkeyHashes.
However, the wallet3 was never able to grab any gifts.
So, I had to reduce the validator code to it's most simple form and build it up from there. This to get a better understanding of what happened and where the bug was.
Each of these steps has it's own commit (listed and commented below).
After some steps, and looking more deeply into the logs, i figured out the the Wallet3's 'grab' transaction was never sent to the blockchain,
hence the on-chain validator was never executed. Then I started to look at the 'off-chain'code.
This was an important code line in the 'grab' function:

let utxos1 = Map.filter (isSuitable $ \dat -> beneficiary1 dat == pkh && now <= deadline dat) utxos

This one says that it will only continue with utxo's given to benficiary1 only if the current timestamp is before the deadline.
So, it the deadline has passed, the wallet will not even bother to send the transaction.
Remember that the nice thing in Cardano is that the wallet is able to validate the transaction before sending it to the blockchain.
This means that if the validation succeeds in the wallet (of-chain), it will also succeed on-chain (when the validating node is executing the on-chain validation).
Validation of-chain should be similar to the validations on-chain.

I had to change the given of-chain validation code line, by removing the check for deadline passed or not.
I know that the validation is now a bit different between on-chain and off-chain.
I have to admit that I don't understand the  next validation line in the of-chain code:

 utxos2 = Map.filter (isSuitable $ \dat -> beneficiary2 dat == pkh && now >  deadline dat) utxos

 Problem is that 'beneficiary2' is the wallets own pubkeyHash.
 It has nothing to do with the beneficiary specified in the "give" wallet, which always ends up as 'beneficiary1' in the 'grab' code.

 Would be nice to discuss this with someone :-)
 
 The 3 changes that are important:
 1. [BF1 allowed to grab all gifts, not care about deadline] https://github.com/ostakvik/cardano-plutus-pioneer-week03/commit/459f74bf7aa65e16ca5be27724d4368bf69c900a#diff-57bfcd8ef1fef27018f8b4881e2594b0f9f659421303fc0b54be30219c24df6e0a
 2. BF1 allow to grab gift if deadline not reached]  https://github.com/ostakvik/cardano-plutus-pioneer-week03/commit/4f02531570355b62adb43c284bd86551da0efd87#diff-57bfcd8ef1fef27018f8b4881e2594b0f9f659421303fc0b54be30219c24df6e
 3. [BF2 allowed to grab if deadline has passed]https://github.com/ostakvik/cardano-plutus-pioneer-week03/commit/2b3fb2a1f241768b4c7d034c7a87b59237e9655d#diff-57bfcd8ef1fef27018f8b4881e2594b0f9f659421303fc0b54be30219c24df6e




 
 
