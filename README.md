# PIPE | DMT Introduction

PIPE is a unique system for managing Bitcoin-native tokens. It draws inspiration from concepts like RUNES and BRC-20, with the aim of combining their strengths.

## Key Functions: Deploy, Mint & Transfer

PIPE operates through three main functions: Deploy, Mint, and Transfer, collectively referred to as DMT. These functions are essential for managing PIPE tokens.

- Deploy: This function signals the creation of a new token and defines its properties, such as name, maximum supply, and rules for issuance.

- Mint: Minting involves creating new tokens based on the rules set during deployment. This function specifies the amount to be minted and the recipient.

- Transfer: Transferring tokens to specific recipients is managed by this function.

## PLEASE NOTE

There is no indexing or wallet for PIPE yet. You can download the following simple javascript that showcases how each deploy, mint and transfer function works: https://trac.network/pipe.zip

Just download, unzip, edit and run in your browser. Initial instructions included in the html file.

## General Rules

Here are some general rules for working with PIPE:

- A PIPE transaction must have at least two outputs: one for the receiver and one with specific data.

- Tokens are represented by tickers and an associated ID, with unique combinations.

- All numbers involved in transactions must be non-negative integers.

- Only one data output per transaction is allowed.

- If a change address isn't specified during a transfer, any remaining tokens are burned.

- Indexers and wallets must handle reorganizations and start indexing from block 809608.

## Deploy Rules

On a high level, the output of a Deploy function is structured as follows:

```
OP_RETURN
P
D
[BASE26 ENCODED TICKER]
[ID]
[OUTPUT]
[DECIMALS]
[MAX]
[LIMIT]
```

Looking closer the values as follows:

- "P" means it's a special code for creating a token.

- "D" means it's for deploying a new token.

- "[BASE26 ENCODED TICKER]" is a unique name for the token.

- "[ID]" is an identification number for the token, and "0" means no specific ID.

- "[OUTPUT]" is the position where the token's owner's address is found in the transaction.

- "[DECIMALS]" is the number of decimal places the token can have (0 to 8).

- "[MAX]" is the maximum supply of the token, represented in a specific format.

- "[LIMIT]" is the maximum amount of tokens that can be created in a single transaction.

The transaction must specify who will receive the newly created tokens, and some technical rules should be followed, like not using certain types of data in the process. The values for "[MAX]" and "[LIMIT]" should be in a specific format, and there are rules about the number of decimal places allowed. Transactions with too many decimals are not allowed.

There's also a change in how values are handled before and after a certain block in the blockchain. Before a specific block, the system should check if the values are in a certain format. After that block, only a specific format is accepted, and any other format will result in an invalid transaction.

Examples:

```
2100 => ok
 2100 => not ok
2100.5 => ok, if decimal length <= [DECIMALS]
2100.50 => not ok
2100.500 => not ok
2100. => not ok
2,100 => not ok
.25 => not ok
18446744073709551616 => not ok (note exceeding the max number)
``` 

Indexers must transform the given [MAX] and [LIMIT] internally into bigints based on [DECIMALS] and perform calculations on those to maintain precision. No calculations or rounding on the original human readable format allowed.

NOTE: until block 809999, indexers need to detect if [MAX] and [LIMIT] are hex values. If not, then the raw values must be used (e.g. '1000' = 1000), else use the hex encoded string.

From block 810000 only hex encoded strings must be accepted and raw values lead to invalid token transactions.

## Optional: PIPE | ART Rules

PIPE | ART Rules is an extra feature for the PIPE system. It allows you to store files, web links, and collection info when you create new digital tokens.

By adjusting some settings like supply, limit, and decimals, you can make these special tokens unique and not easily interchangeable with others. These tokens work the same way as regular tokens when you create or transfer them, so there's no special treatment required.

Unlike regular token details, the information for PIPE | ART tokens is stored as special digital signatures using unique keys. Each key represents a collection of items, and you can specify the number and maximum limit for each item. You can also add extra information to describe each item.

You can either include the actual file data and descriptions or simply provide links to external resources. You can even specify who should receive tokens when you create them, bypassing the usual minting process, but this can only be done once based on the deployment's limit.

This PIPE | ART feature is added in the same transaction as the main token creation.

Here's how the special data for PIPE | ART tokens is structured:

```
[PUBKEY]
OP_CHECKSIG
OP_0
OP_IF
P
A
(I|R)
[MIMETYPE]
[INLINE DATA | REFERENCE STRING]
N
[NUMBER]
[MAX-NUMBER]
B
[BENEFICIARY-OUTPUT]
(empty | T | TR)
[TRAIT-CONTENT]
01
OP_ENDIF
```

"[PUBKEY]": The collection's address. To add more items to a collection, use the same private key for deployments.

"[MIMETYPE]": The type of content for inline data, represented as a hexadecimal string. If it's a reference string instead, indexers should ignore it or leave it empty.

"[INLINE DATA | REFERENCE STRING]": If you use "I," you should include the actual data. If you use "R," the next input should be a reference to a file. Only one reference is allowed for "R," while "I" can have multiple data inputs.

"[NUMBER]": A positive number representing the item's position in the collection. It must not exceed the "[MAX-NUMBER]," and indexers should ignore deployments if the same number is used for a "[PUBKEY]" that already exists.

"[MAX-NUMBER]": A positive number representing the maximum items in the collection. The creator can increase this number, but it can never decrease. The maximum value is 999,999,999.

"[BENEFICIARY-OUTPUT]": A positive number starting from 0, allowing you to mint tokens directly with the deployment, based on the limit. 0 means this feature is disabled. Indexers should subtract 1 from "[BENEFICIARY-OUTPUT]" to properly assign the desired output.

"[TRAIT-CONTENT]": This can be key-value pairs represented as hexadecimal strings, or references to additional traits stored off-chain, or it can be left out entirely. The labels "T" or "TR" can also be ignored.

The "01" at the end is not required for indexing but should be added by clients.

Recommendations for creating non-fungible tokens

|| ERC-721 | ERC-1155 |
| -------------| ------------- | ------------- |
| Decimals | 0 | 0 |
| Supply | 1 | 1 - 999999 |
| Limit | 1 | 1

## Mint Rules

Mint structure:

```
OP_RETURN
P
M
[BASE26 ENCODED TICKER]
[ID]
[OUTPUT]
[MINT AMOUNT]
```

When you want to make a new token, you follow this structure:

First, there's a special code "P" to show it's for creating tokens.

Then comes "M," indicating you're minting tokens.

"[BASE26 ENCODED TICKER]" is a unique name for the token.

"[ID]" is an identification number for the token, and "0" means there's no specific ID.

"[OUTPUT]" tells where the tokens should go, usually an address.

"[MINT AMOUNT]" is how many tokens you want to create, represented in a specific format. It must be more than 0.

If the amount you're minting doesn't exceed the limit and the remaining supply from the initial token creation, those new tokens go to the specified recipient. Indexers and wallets need to keep track of how many tokens are credited to the recipient and link it to the transaction output.

You can't use UTXOs (previous token-related transactions) that already have tokens when you create new tokens. The transaction for creating tokens must specify who receives them, following the general rules.

The token creation and minting can happen in the same blockchain block, but any minting transaction will be rejected if it has an index lower than the initial creation transaction.

The "[MINT AMOUNT]" must be in a specific format, representing a readable number. It can't have extra zeros or decimal points beyond the specified limit. Any transactions with too many decimals are rejected, and the maximum number is '18446744073709551615'.

Examples:

```
2100 => ok
 2100 => not ok
2100.5 => ok, if decimal length <= [DECIMALS]
2100.50 => not ok
2100.500 => not ok
2100. => not ok
2,100 => not ok
.25 => not ok
18446744073709551616 => not ok (note exceeding the max number)
``` 

Indexers need to internally convert the given "[MINT AMOUNT]" into a special number format based on the specified decimal places to maintain precision. They should not make any calculations or round the original readable format.

Until a certain block (809999), indexers need to check if the "[MINT AMOUNT]" is in a specific format. If not, they use the raw value (e.g., '1000' is treated as 1000). After that block (810000 and onwards), only the specific format is accepted, and raw values will result in invalid token transactions.

## Transfer Rules

Transfer may contain a number quadruple of pushes after 'T'. Each quadruple may address different tickers, IDs, outputs and amounts.
This allows to specify change addresses (and limited multi-send) within a single transaction. There is no limit on the amount of quadruples other than the max. allowed script size for the output.

Transfer structure:

```
There's a special code "P" to show it's for creating tokens.
Then comes "T," indicating you're transferring tokens.
You can have several sets of information for different tokens, recipients, and amounts, known as "quadruples."
Each quadruple includes:
The unique name of the token.
An identification number for the token, or "0" if there's no specific ID.
The index where the tokens should go, usually an address.
The amount of tokens you want to send, represented in a specific format.
...end quadruple
...next quadruple...
```

"P": shortcut for PIPE, signalling this is a PIPE function.

"T": shortcut for Transfer, signalling the following data is to send tokens.

Quadruple:

"[BASE26 ENCODED TICKER]": human readable ticker name, encoded as described in General Rules.

"[ID]": the arbitray ID for the ticker from 0 - 999999 as unsigned integer. "0" semantically signals that there is no ID assigned (note that ticker:ID must be unique).

"[OUTPUT]": the index as unsigned integer of the output containing the address/pubkey of the beneficiary. [OUTPUT] is not allowed to point to an output containing OP_RETURN.

"[TRANSFER AMOUNT]": The amount to transfer as hex encoded string.

The amount you're sending is subtracted from the tokens in the input UTXOs (previous token-related transactions). Any remaining tokens should go to a change address or can be "burned" if you don't want them anymore. If you have UTXOs with enough token balances, the sent amount goes to the specified recipient.

It's important to note that you can only use each recipient address once to avoid mixing different token types in a single UTXO. The transaction will be rejected if there's not enough token balance or if there are duplicate recipient addresses. It's also not recommended to mix different token types in a single transaction because of size limitations.

The transfer transaction must specify who receives the tokens, following the general rules. You can create, mint, and transfer tokens in the same blockchain block, but a transfer will be rejected if its transaction order is earlier than the creation or minting transaction that provides the tokens for the transfer.

The amount you're transferring must be in a specific format, representing a readable number. It can't have extra zeros or decimal points beyond the specified limit. Any transactions with too many decimals are rejected, and the maximum number is '18446744073709551615'.

Examples:

```
2100 => ok
 2100 => not ok
2100.5 => ok, if decimal length <= [DECIMALS]
2100.50 => not ok
2100.500 => not ok
2100. => not ok
2,100 => not ok
.25 => not ok
18446744073709551616 => not ok (note exceeding the max number)
``` 

Indexers need to convert the given transfer amount into a special number format based on the specified decimal places to maintain precision. They shouldn't make any calculations or round the original readable format.

The transfer operation must be "atomic," meaning if one part of the transfer fails, the entire operation fails. No changes in token balances will occur in this case.

Until a specific block (809999), indexers need to check if the transfer amount is in a specific format. After that block (810000 and onwards), only the specific format is accepted, and raw values will result in invalid token transactions.

## Transfer without function

Sending tokens without using a specific function should be possible, but there are some rules to follow:

You must have at least one input that has tokens associated with it in the transaction.

All the tokens of the first type you find (based on their unique name and ID) will be added together from all the inputs and sent to the first output that's not an "OP_RETURN" (a specific type of output used for data).

Any other types of tokens will be ignored and spent, meaning they won't be sent to the receiver.

This means that when sending tokens without using a specific function, you need to be careful not to mix different types of tokens in the inputs unless you want to lose some tokens in the process.
