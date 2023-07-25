+++
title = "Mint Solana NFT Using Rust & Metaplex SDK"
date = 2023-07-25
category = "Prog"
draft=false

[taxonomies]
tags = ["Rust", "solana"]
+++

Solana is a public, permissionless blockchain launched in 2020. It leverages computer hardware for its high throughput and performance. Solana has been the pioneer in cutting-edge innovation leading in the adoption of tech like [concurrent Merkle trees](https://docs.solana.com/learn/state-compression#what-is-a-concurrent-merkle-tree), [interfaces](https://forum.solana.com/tag/interfaces), [token-2022](https://spl.solana.com/token-2022)...etc

I briefly talked about some [differences between Solana and EVM chains previously](../eifweekoneandtwo/#solana-comparison) and in this article I will take you through the process of minting an NFT using the Rust Solana SDK and the MetapleX SDK.

<!-- more -->

<!-- todo - create a frontend that allows anyone to create json from uri -->

# Table of contents

-   [Environmental Setup](#env-setup)
-   [Introduction to Tokens in Solana ](#token-intro)
-   [Intro to Native Rust Solana Development 1](#native-rust-intro)
-   [Client Side Set up 1](#client-side-1)
-   [More on Native Rust Solana Dev](#more-rust-dev-solana)
-   [Client Side Set up 2](#client-side-2)
-   [Further Reading](#further-reading)

## Environmental Set-up <a name="env-setup"></a>

**Consider:** [Solana Playground](https://beta.solpg.io/) - A dedicated online playground where you can write and deploy your Solana programs using anchor, native Rust or Seahorse-lang online.

You can also skip this section if you already your Solana environment setup.

Convenient (**must have**) tool that will make development easier

-   [Rust Installed](https://www.rust-lang.org/learn/get-started)
-   [solana cli](https://docs.solana.com/cli)
-   [Rust Analyzer](https://rust-analyzer.github.io/) - Install it via extensions, if working on VS Code, other wise any Rust Language Server works.

To install the Solana CLI run the command below,

```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

Depending on your system, you might get an installer messaging

```bash
Please update your PATH environment variable to include the solana programs:
```

Follow the installer instructions and Confirm the solana suite is installed properly by running,

```bash
solana --version
```

Next, we need to set up a paper wallet, that we will use to test our programs locally and deploy them.

```bash
solana-keygen new
```

This command creates a new paper wallet in the `~/.config/solana/id.json` directory, though you have the option to tell it where to save the file using the `-o, --outfile <FILEPATH>` flag.

In this tutorial we will be writing a [native Rust Solana Program](https://docs.rs/solana-program/latest/solana_program/) though it's advisable to use the [anchor framework](https://www.anchor-lang.com/).

**Why Native?** Because at the moment the solana cli and anchor are incompatible.

## Introduction to Tokens in Solana <a name="token-intro"></a>

To start off, in Solana, everything is an account. You have data accounts that, you guessed it, store data and executable account(smart contracts) that run code when certain conditions are met.

In the context of tokens/NFTs, Unlike in Ethereum, where you have predefined standards/interfaces for creating a contract that mint's tokens(nfts included), solana has the Token program. That means you do not have to deploy a contract every time you want to mint a new token. This also makes it really easy to mint token using TypeScript or Python, if you have the web3.js Solana SDK.

<!-- Recent innovations on the Token program, i.e token-2022 means new features can be added to our tokens like `transfer fees`, `restrict transfers` and even introduce things like a `metadata pointer`(shameless plug, but I added this feature to the CLI ✌🏼) to the token program.

Check out [this talk](https://www.youtube.com/watch?v=Gr0cSdqIjD4) to learn more about the Token-2022 program. -->

Lets talk about some terminologies that usually trip up new developers,

-   The **Token** that you want to create. It can be a Fungible ERC-20 like Token or a non-fungible ERC-721 like NFT.
    How are Solana tokens different? In Solana to represent a **non fungible token**, usually has `0` decimals meaning it can not be subdivided any further.
    **Fungible tokens** on the other hand, have the supply greater than one and can have decimals.

-   The **mint accounts** or **token mint** is an account that contains information about a specific token type.
    For example, the `mint authority`(who can create more tokens), `freeze authority` - who can freeze accounts for a certain mint account, `total supply` e.t.c
    The mint account does not hold any token. It just defines the rules for the token you are creating.
-   The **token account** is a general term for any account that holds tokens for a particular mint.
-   The **Associated Token Account**, refers to a token account that's associated with a particular primary address.
    They are deterministic account that are derived using the **address** and the **mint account**.
    For example, say a user has 200 tokens associated with a specific mint. He would need to manage all the keys for these tokens. If the user wanted to send or receive a token, he would need to know the token account for a new address beforehand.
    With `ATA` the user can send and receive tokens using his main address. Since the `ATA` is derived deterministically, it'll be guaranteed to just work.

    More in the [official documentation](https://spl.solana.com/associated-token-account#background).

-   **Metaplex** is a protocol built on top of Solana to mint and trade NFTs.
    It allows us to attach metadata, images, audio or video to tokens.

-   **Mater Edition** is a type of NFT that manages and mints a series of **prints** or **edition** from a single piece
    of artwork.
    This is what enables us to create collection and map related nfts items from the same collection together.
    For example, if you have an artpiece and create a master edition for it, you could create 100 unique 'prints' each linked back to the master edition carrying a unique **edition number** like `1 of 100, 20 of 100` etc

-   To Bring it all back together:
    If you want to create an 10 nfts called **MyToken**, you would `instruct` the **Token Program** to create/mint the `MyToken` by start initializing the **mint account**. It would be this account that contains details such as the token decimals, in this case `0` and a total supply of `1`. These NFTs are stored in the **Token Account**. When one of these nfts is transferred to a wallet address, an **Associated Token Account** is created.
    To mint something like a `collection of nfts`, you would use something like **master edition** from Metaplex.

It can be hard learning new things, so take a break and revisit these concepts after a few days.

## Intro to Native Rust Solana Development. <a name="native-rust-intro"></a>

To get started we will initialize a new empty rust `lib` project with cargo.

```bash
cargo new --lib solana-nft-native
```

These are the crates that we will need to create our program.

**Note:** _We are using versioned crates because there is a [borsh version mismatch](https://solana.stackexchange.com/questions/6991/how-to-fix-error-the-trait-borshserialize-is-not-implemented-for-pubkey) between what metaplex and solana-program-library uses_.

-   [`solana-program`](https://docs.rs/solana-program/latest/solana_program/) crate which provides the necessary APIs that allow us to interact with Solana.
-   [`borsh`](https://borsh.io/) allows us to efficiently encode and decode data structures in a compact binary format.
-   [`borsh derive`](https://crates.io/crates/borsh-derive/0.8.1) is a procedural macro that allows us to `#[derive(BorshSerialize, BorshDeserialize)]` our data structs.
-   [`spl-token`](https://docs.rs/spl-token/latest/spl_token/) to allow gives us access to the spl-token program.
-   [`spl-associated-token-account`](https://docs.rs/spl-associated-token-account/2.0.0/spl_associated_token_account/) allowing us to derive associated token accounts.
-   [`mpl-token-metadata`](https://docs.rs/mpl-token-metadata/1.13.0/mpl_token_metadata/) allowing us to use Metaplex functions to add metadata to our token.

```bash
cargo add solana-program@=1.14.20
cargo add borsh@0.9.3
cargo add borsh-derive@0.9.3
cargo add spl-token@3.3.0 --features=no-entrypoint
cargo add spl-associated-token-account@1.0.3 --features=no-entrypoint
cargo add mpl-token-metadata@1.2.5 --features=no-entrypoint
```

We will also make to our `Anchor.toml` file.

```toml
[lib]
crate-type = ["cdylib", "lib"]
```

`[lib]` means that our project, compiles to a library crate, which can be used and linked by other libraries and executable. In the context of our program, we have defined the `crate-type` options as `lib` and `cdylib`, [read more](https://doc.rust-lang.org/cargo/reference/cargo-targets.html#library)

**Note:** Solana programs must specify the crate-type as `cdylib`. This enables them to be automatically be discovered and built by the `cargo build-spf`.

Test everything works properly by building the program.

```bash
 cargo build-sbf
```

If your program builds successfully, let's head into our `src/lib.rs` file, clear the contents of this file.
populate it with the contents below.

This is a simple `hello world` program program that logs `hello world!` when called on a solana cluster like `devnet`.

```rust
use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, msg, pubkey::Pubkey,
};

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, World!");

    Ok(())
}
```

Let's break down the line:

We get started by bringing into scope the packages that we added using cargo.

```rs
use solana_program::{
    account_info::AccountInfo, entrypoint, entrypoint::ProgramResult, msg, pubkey::Pubkey,
};
```

-   `account_info::AccountInfo` - provides the metadata of an account which it carries, including the data it holds, pubkey, etc.

-   `entrypoint` - used to mark the entrypoint of a program. Whenever a call is made to a solana program, it's the function in this macro that will be run.

-   `entrypoint::ProgramResult` - signifies the result of a program. Did it execute properly or did it return an error.

-   `msg` - used to log messages during the execution of a program since `dbg!` and `println!` does not work.
    Keep in mind that:

    -   the more frequent your `msg` calls are, the more **computational overhead** you will have per transaction.
    -   the longer your message length, the more **data** it will consume.

-   `pubkey::Pubkey` - solana public key struct. used when you define public key values.

Next we use the `entrypoint!` macro to define the function that will be called when a tx is made to our program. In our case it is the `process_instruction` function.

```rust
entrypoint!(process_instruction);
```

Finally we have our `process_instruction` function.

```rust
pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, World!");

    Ok(())
}
```

Our `process_instruction` function has three parameters,

1. `_program_id` - this the the address of your program. you receive it once you've deployed your program.
2. `_accounts` - **NOTE:** solana programs are stateless. What that means is that there is not data stored on a solana program. Instead you have to create other accounts should you want to write data on-chain and pass them as `_accounts` arguments. You pass all the account to which you will read/mutate as a slice of accounts.
3. `_instruction_data` - This is where you pass in any serialized data that, mutates/writes to accounts that the program interacts with.

<!-- client side -->

## Client Side Set-up 1 <a name="client-side-1"></a>

Let's initialize a new ts project which will act as our client to interact with our program.
_I am not a big fan of monorepos so this will be in another directory_

```bash
mkdir solana-nft-native-client
cd solana-nft-native-client
npm init -y
git init
tsc --init
mkdir src
cd src
touch main.ts utils.ts
```

Install our versioned ts packages

```bash
npm i @metaplex-foundation/mpl-token-metadata@2.13.0 @solana/spl-token@0.3.8 @solana/web3.js@1.78.0  borsh@0.7.0 buffer@6.0.3 fs@0.0.1-security
npm i --save-dev typescript@4.9.5 @types/bn.js@5.1.1
```

After installing our packages, we will also add an npm command `dev`, to call ts-node on our client side code. Make sure you have [`ts-node`](https://www.npmjs.com/package/ts-node) installed.

. Your finished `package.json` should resemble,

```ts
{
	"scripts": {
		"dev": "ts-node src/main.ts"
	},
	"dependencies": {
		"@metaplex-foundation/mpl-token-metadata": "^2.13.0",
		"@solana/spl-token": "^0.3.8",
		"@solana/web3.js": "^1.78.0",
		"borsh": "^0.7.0",
		"buffer": "^6.0.3",
		"fs": "^0.0.1-security"
	},
	"devDependencies": {
		"@types/bn.js": "^5.1.1",
		"typescript": "^4.9.5"
	}
}
```

We will also switch our node version to `v16.17.1` using [`nvm`](https://github.com/nvm-sh/nvm). This is because `solana-test-validator` does not work with anything above `v17`.

```bash
nvm use v16.17.1
```

Let's write a script to test our hello world program.

First we will create our utility functions to derive our `command line wallet` Keypair and `program` Keypair.

**NOTE:** make sure your `PROGRAM_KEYPAIR_PATH` and `USER_KEYPAIR_PATH` paths are set correctly.

Populate your `src/utils.ts` with,

```ts
import { readFileSync } from "fs";
import { Keypair } from "@solana/web3.js";
import { homedir } from "os";

const PROGRAM_KEYPAIR_PATH =
	homedir() +
	"/Documents/web3/solana/solana-nft-native/target/deploy/solana_nft_native-keypair.json";
const USER_KEYPAIR_PATH = homedir() + "/.config/solana/id.json";

export function userKeyPair(): Keypair {
	return Keypair.fromSecretKey(
		Buffer.from(JSON.parse(readFileSync(USER_KEYPAIR_PATH, "utf-8")))
	);
}

export function programKeyPair() {
	return Keypair.fromSecretKey(
		Buffer.from(JSON.parse(readFileSync(PROGRAM_KEYPAIR_PATH, "utf-8")))
	);
}

console.log("user keypair", userKeyPair().publicKey.toString());
console.log("program keypair", programKeyPair().publicKey.toString());
```

Running the script above with `npm run dev` should print the two public keys.

Next we will write client code to call our program. If everything runs successfully, then our program should log `hello world`.

Populate `main.ts` with

```ts
import {
	Connection,
	Keypair,
	PublicKey,
	sendAndConfirmTransaction,
	Transaction,
	TransactionInstruction,
} from "@solana/web3.js";
import { userKeyPair, programKeyPair } from "./utils";

let connection = new Connection("http://localhost:8899");
let programId = programKeyPair().publicKey;
const payer = userKeyPair();

async function helloWorld(): Promise<String> {
	const ix = new TransactionInstruction({
		keys: [],
		programId,
	});

	const tx = new Transaction().add(ix);

	return await sendAndConfirmTransaction(connection, tx, [payer]);
}

helloWorld()
	.then((txSig) => console.log("tx sig: ", txSig))
	.catch((err) => console.error("program execution unsuccessful", err));
```

In the code snippet above, we define three global variables:

-   `connection` which we use to communicate with the solana network. Right now, it is set to connect to our local solana network.
-   `programId` is our program's public key
-   `payer` is our keypair which we will use to pay for transactions.

To understand the function, you must understand how transactions work in solana. In Solana a transaction can contain one of multiple instructions. And what are instruction? Simply, it is a method or functions of a Solana program.

In solana an instruction takes in three arguments, the first being the `addresses` of the programs the instruction will interact with the `programId `and finally `data` which you want to write to an account. In our `ix`, the data is omitted because we are not writing to any account.

The `sendAndConfirmTransaction` method allows us to send and sign transactions.

To ran our client, we will need to set up a local validator. In a new terminal window run

```bash
solana-test-validator
```

Next we will need the solana cli to send our transaction to this test-validator we are running with

```bash
solana config set --url localhost
```

We will now compile our program and deploy it. Run the `build-sbf` command in your program directory.

```bash
cargo build-sbf
```

We will now deploy our compiled program. This will be the `.so` (shared object) file in the `target/deploy` directory.

```bash
solana program deploy solana_nft_native.so
```

Head back to the directory where you wrote your client code.

Open a new terminal window or tab. We will use this to view our program logs

```bash
solana logs --url localhost
```

We will now run our client script and if successful, we should see the logs with the string `Hello, World!` logged out.
![Screenshot of our program logging our Hello World!](/solana-nft-native/program-log-hello-world.png "hello world logs")

With the basics in place, let's head back to where our program code lives and write the code to mint our nft.

## More on Native Rust Solana Dev <a name="more-rust-dev-solana"></a>

Clear everything except our package imports. We will start by importing `borsh` for serialization.

```rust
use borsh::{BorshDeserialize, BorshSerialize}; //new
use mpl_token_metadata::instruction as mpl_instruction; //new
use solana_program::{
    account_info::{next_account_info, AccountInfo}, //new
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program::invoke,    //new
    pubkey::Pubkey,
    rent::Rent,         //new
    system_instruction, //new
    sysvar::Sysvar,     //new
};
use spl_token::{instruction as token_instruction, state::Mint}; //new

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, World!");

    Ok(())
}

```

The new imports:

-   `borsh::{BorshDeserialize, BorshSerialize};` - serialize and deserialize our data.
-   `mpl_token_metadata::instruction as mpl_instruction;` - provides functions for us to interact with the Metaplex Token Metadata program.
-   `account_info::{next_account_info` function is used to parse accounts passed in as input to our Solana program. Remember that Solana need to know before hand the accounts that it will touch, and this is done by providing the addresses of said accounts.
-   `program::invoke` is used to make [**Cross Program Invocations**](https://docs.solana.com/developing/programming-model/calling-between-programs). It takes two arguments. The instructions to run and the accounts that it will touch.
-   `rent::Rent` is used to pay for the storage space we will take up when storing data on the blockchain.
-   `system_instruction` is a module that allows us to construct system level instructions that directly interact with the Solana blockchain. For example, when creating an account, you would make a call using a system instruction.
-   `sysvar::Sysvar` - is a module that provides various system variables about the state of the blockchain. They are **read only**. For example you might want to use the `Clock Sysvar` that contains information about the [`epochs and slots`](https://www.helius.dev/blog/solana-slots-blocks-and-epochs)
-   `spl_token::{instruction as token_instruction, state::Mint};` - provides methods for us to interact with the token program.

When creating our token, we will need to pass in a **title**, **symbol** and our **metadata uri**. This data will need to be serialized and deserialized so that the client and the program can make sense of it. To do this we will use the `Serialize and Deserialize` borsh traits in a `derive` attribute.

We will pass on these arguments to the `Metaplex Metadata Program`

```rust
#[derive(BorshSerialize, BorshDeserialize)]
pub struct CreateTokenArgs {
    pub nft_title: String,
    pub nft_symbol: String,
    pub nft_uri: String,
}
```

Next, let's invoke the function that will call our `token` and `metaplex` program. We will call it `create_nft`.

```rust

fn create_nft(accounts: &[AccountInfo], create_token_data: CreateTokenArgs) -> ProgramResult {
    let accounts_iter = &mut accounts.iter();

    let mint_account = next_account_info(accounts_iter)?;
    let mint_authority = next_account_info(accounts_iter)?;
    let metadata_account = next_account_info(accounts_iter)?;
    let payer = next_account_info(accounts_iter)?;
    let rent = next_account_info(accounts_iter)?;
    let system_program = next_account_info(accounts_iter)?;
    let token_program = next_account_info(accounts_iter)?;
    let token_metadata_program = next_account_info(accounts_iter)?;

    Ok(())
}

```

The functions takes a slice of type `AccountInfo` and create a mutable iterator for the accounts input. it is from this accounts that we pull out the accounts that are required to create an nft while interacting with the metadata and token programs.

-   The `mint_account` contains details about the mint.
-   The `mint_authority` contains the address of the person who will control the `mint`
-   The `metadata_account` as it's name suggests will store the metadata details
-   The `payer` contains the account that will bare the costs for minting the nft and rent for opening the required accounts.
-   `rent`. When opening an account, you need to set aside SOL to pay for it. You can recover this once you close an account.
-   `system_program` This executable accounts/program is what is responsible for opening up new accounts. In Solana a good mental model to think about things is that every program/accounts will always have an owner that controls it.
-   `token_program` will be the address for our token program, while `token_metadata_program` will be the address for the token metadata program.

We are now ready to start calling instructions that will create our nft.

```rust
// invoke the system program to create our account
    msg!("Creating mint account...");
    msg!("Mint: {}", mint_account.key);
    invoke(
        &system_instruction::create_account(
            &payer.key,
            &mint_account.key,
            (Rent::get()?).minimum_balance(Mint::LEN),
            Mint::LEN as u64,
            &token_program.key,
        ),
        &[
            mint_account.clone(),
            payer.clone(),
            system_program.clone(),
            token_program.clone(),
        ],
    )?;
```

What is this ↑ piece of code doing? Well, we are using `invoke` to make a cross program call between our `system_program` and our `token_program`. To make a CPI call, you need to pass the instruction you want to call and all the accounts that the call interacts with as arguments.

In the first parameter for our instruction we are calling the `create_account` instruction from `system_program` to initialize a new account. These are the parameters we are required to pass to our instruction -

1.  `&payer.key` Is the public key that will pay for the accounts creation,
2.  `&mint_account.key` Is the public key for the account being created,
3.  `(Rent::get()?).minimum_balance(Mint::LEN),` when creating an account in solana, you are required to pay for blockspace. There is not such thing as a free lunch. To store data on Solana in an account, you need to pay for it.
    To note, is that there code above is checking whether our account is **rent exempt**. We check the `minimum_balance` required to keep the account **rent exempt**
    **rent exempt** is a concept in Solana that allows an account to not incur any rent rent collection for a duration of two years.
    `Rent` is the program that manages this.

4.  `Mint::LEN as u64` is passed in to determine the length/space our account will take. The default size is `82 bytes`
5.  `&token_program.key` is the address of the account that owns or account. In this case it's the token_program.

The next parameter is a slice of the account that `invoke` will touch.

Next we initialize the created account as the mint account

```rust
    //initialize the created account as the mint
    msg!("Initializing mint account...");
    msg!("Mint: {}", mint_account.key);
    invoke(
        &token_instruction::initialize_mint(
            &token_program.key, //token program public key
            &mint_account.key, //mint public key
            &mint_authority.key, //mint authority
            Some(&mint_authority.key), //freeze authority
            0, //decimals
        )?,
        &[
            mint_account.clone(),
            mint_authority.clone(),
            token_program.clone(),
            rent.clone(),
        ],
    )?;
```

As with the previous `invoke` call, we pass two arguments to this method. One is the instructions that we will invoke and the second is the accounts that the cross program call will touch.

our `token_instruction::initialize_mint` takes in 5 parameters, as indicated by the comments.
To note, is the no of decimals. For a token in Solana to be non-fungible, it should not be divisible, hence **zero**.

To finish off our program, we call the token metadata program from Metaplex. We will use the [`create_metadata_accounts_v3`](https://docs.rs/mpl-token-metadata/latest/mpl_token_metadata/instruction/fn.create_metadata_accounts_v3.html) instruction to create our Metadata.

```rust
 //call Metaplex token metadata program
    msg!("Creating metadata account...");
    msg!("Metadata account address: {}", metadata_account.key);
    invoke(
        &mpl_instruction::create_metadata_accounts_v3(
            *token_metadata_program.key, //program id
            *metadata_account.key,       // metadata address
            *mint_account.key,           //mint address
            *mint_authority.key,         //mint authority address
            *payer.key, //payer public key address
            *mint_authority.key, //update authority address
            create_token_metadata.nft_title, //nft title
            create_token_metadata.nft_symbol, //nft symbol
            create_token_metadata.nft_uri, //nft uri
            None, //creators. for something you pass in a VEC of their addresses
            0, //royalties
            true, //is the payer the update authority
            false, //can we update the token metadata
            None, //collection the nft belongs to. struct of pubkey and
            None, //uses
            None, //collection details
        ),
        &[
            metadata_account.clone(),
            mint_account.clone(),
            mint_authority.clone(),
            payer.clone(),
            token_metadata_program.clone(),
            rent.clone(),
        ],
    )?;

    msg!("Token mint created successfully.");
```

The code pattern is the same as the previous one. We first call `invoke` with two arguments. The instruction and the accounts we touch.

The `create_metadata_accounts_v3` will create our metadata account and write to it our metadata information. Each argument is commented to show what it represents.

To finally, finally finish off our program, we being our `create_token` into our `process_instruction` function which will now look like this

```rust
pub fn process_instruction(
    _program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    let token_metadata = CreateTokenArgs::try_from_slice(instruction_data)?; //new

    create_nft(accounts, token_metadata)?;

    Ok(())
}
```

The method is not something that is new. What is new, is `try_from_slice`. Remember that to efficiently sent messages to and from Solana and reduce computational overhead, your data needs to be serialized to a format(binary format) that can efficiently do this.

The` try_from_slice` method is part of the `BorshDeserialize trait`, and it's used to convert a byte slice `(&[u8])` into an instance of our `CreateTokenArgs`.

And yes, You will need to serialize data from the client side too. This is why a lot of dev prefer using [anchor](https://www.anchor-lang.com) as it automatically handles this boilerplate of serializing and deserializing data.

Altogether your `src/lib.rs` should resemble.

```rust
use borsh::{BorshDeserialize, BorshSerialize};
use mpl_token_metadata::instruction as mpl_instruction;
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program::invoke,
    program_pack::Pack,
    pubkey::Pubkey,
    rent::Rent,
    system_instruction,
    sysvar::Sysvar,
};
use spl_token::{instruction as token_instruction, state::Mint};

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    accounts: &[AccountInfo],
    instruction_data: &[u8],
) -> ProgramResult {
    let token_metadata = CreateTokenArgs::try_from_slice(instruction_data)?;

    create_nft(accounts, token_metadata)?;

    Ok(())
}

#[derive(BorshSerialize, BorshDeserialize)]
pub struct CreateTokenArgs {
    pub nft_title: String,
    pub nft_symbol: String,
    pub nft_uri: String,
}

fn create_nft(accounts: &[AccountInfo], create_token_metadata: CreateTokenArgs) -> ProgramResult {
    let accounts_iter = &mut accounts.iter();

    //pick out our accounts
    let mint_account = next_account_info(accounts_iter)?;
    let mint_authority = next_account_info(accounts_iter)?;
    let metadata_account = next_account_info(accounts_iter)?;
    let payer = next_account_info(accounts_iter)?;
    let rent = next_account_info(accounts_iter)?;
    let system_program = next_account_info(accounts_iter)?;
    let token_program = next_account_info(accounts_iter)?;
    let token_metadata_program = next_account_info(accounts_iter)?;

    // invoke the system program to create our account
    msg!("Creating mint account...");
    msg!("Mint: {}", mint_account.key);
    invoke(
        &system_instruction::create_account(
            &payer.key,
            &mint_account.key,
            (Rent::get()?).minimum_balance(Mint::LEN),
            Mint::LEN as u64,
            &token_program.key,
        ),
        &[
            mint_account.clone(),
            payer.clone(),
            system_program.clone(),
            token_program.clone(),
        ],
    )?;

    //initialize the created account as the mint
    msg!("Initializing mint account...");
    msg!("Mint: {}", mint_account.key);
    invoke(
        &token_instruction::initialize_mint(
            &token_program.key,
            &mint_account.key,
            &mint_authority.key,
            Some(&mint_authority.key),
            0,
        )?,
        &[
            mint_account.clone(),
            mint_authority.clone(),
            token_program.clone(),
            rent.clone(),
        ],
    )?;

    //call Metaplex token metadata program
    msg!("Creating metadata account...");
    msg!("Metadata account address: {}", metadata_account.key);
    invoke(
        &mpl_instruction::create_metadata_accounts_v3(
            *token_metadata_program.key,      //program id
            *metadata_account.key,            // metadata address
            *mint_account.key,                //mint address
            *mint_authority.key,              //mint authority address
            *payer.key,                       //payer public key address
            *mint_authority.key,              //update authority address
            create_token_metadata.nft_title,  //nft title
            create_token_metadata.nft_symbol, //nft symbol
            create_token_metadata.nft_uri,    //nft uri
            None,  //creators. for something you pass in a VEC of their addresses
            0,     //royalties
            true,  //is the payer the update authority
            false, //can we update the token metadata
            None,  //collection the nft belongs to. struct of pubkey and
            None,  //uses
            None,  //collection details
        ),
        &[
            metadata_account.clone(),
            mint_account.clone(),
            mint_authority.clone(),
            payer.clone(),
            token_metadata_program.clone(),
            rent.clone(),
        ],
    )?;

    msg!("Token mint created successfully.");

    Ok(())
}
```

To finish off our program side development, we are going to `build and deploy` our program on `devnet`.

```bash
solana config set --url devnet
cargo build-sbf
solana program deploy solana_nft_native.so
```

**Note:** Your so file is in the `target/deploy` directory.

## Client Side Set-up 2 <a name="client-side-2"></a>

We are almost at the end of our journey of creating nfts for Solana.

Going back to where you set up your client code before. You can comment out the `helloWorld` function we wrote. We will create a class to serialize our data.

```ts
class TokenArgs {
	nft_title: string;
	nft_symbol: string;
	nft_uri: string;

	constructor(nft_title: string, nft_symbol: string, nft_uri: string) {
		this.nft_title = nft_title;
		this.nft_symbol = nft_symbol;
		this.nft_uri = nft_uri;
	}

	serializeCreateTokenData(data: TokenArgs): Uint8Array {
		const schema = new Map([
			[
				TokenArgs,
				{
					kind: "struct",
					fields: [
						["nft_title", "string"],
						["nft_symbol", "string"],
						["nft_uri", "string"],
					],
				},
			],
		]);
		return Buffer.from(borsh.serialize(schema, data));
	}
}
```

We have abstracted the logic to serializing our data to the `TokenArgs` class.

In it we have our constructor which sets our `nft_title`, `nft_symbol` and `nft_uri` member variables.

Next we have our `serializeTokenData` method which contains our borsh schema and it's this methods that serializes our data, ready to be send to our program.

We then set up our `createToken` function. We will create our mint accounts keypair using `Keypair.generate();` functions and derive a pda which will store our metadata. The seeds to do so are `metadata`, `TOKEN_METADATA_PROGRAM_ID` and `mintKeypair.publicKey`.

```ts
import { PROGRAM_ID as TOKEN_METADATA_PROGRAM_ID } from "@metaplex-foundation/mpl-token-metadata";

const mintKeypair: Keypair = Keypair.generate();

const metadataAddress = PublicKey.findProgramAddressSync(
	[
		Buffer.from("metadata"),
		TOKEN_METADATA_PROGRAM_ID.toBuffer(),
		mintKeypair.publicKey.toBuffer(),
	],
	TOKEN_METADATA_PROGRAM_ID
)[0];
```

Let finish off by declaring our data variables and assigning them appropriately.

```ts
async function createToken(): Promise<String> {
	let connection = new Connection("http://localhost:8899");
	let programId = programKeyPair().publicKey;
	const payer = userKeyPair();

	const mintKeypair: Keypair = Keypair.generate();

	const metadataAddress = PublicKey.findProgramAddressSync(
		[
			Buffer.from("metadata"),
			TOKEN_METADATA_PROGRAM_ID.toBuffer(),
			mintKeypair.publicKey.toBuffer(),
		],
		TOKEN_METADATA_PROGRAM_ID
	)[0];

	const instructionData = new TokenArgs(
		"Kobeni",
		"KBN",
		"https://raw.githubusercontent.com/687c/solana-nft-native-client/main/metadata.json"
	);

	let ix = new TransactionInstruction({
		keys: [
			{ pubkey: mintKeypair.publicKey, isSigner: true, isWritable: true }, // Mint account
			{ pubkey: payer.publicKey, isSigner: false, isWritable: true }, // Mint authority account
			{ pubkey: metadataAddress, isSigner: false, isWritable: true }, // Metadata account
			{ pubkey: payer.publicKey, isSigner: true, isWritable: true }, // Payer
			{ pubkey: SYSVAR_RENT_PUBKEY, isSigner: false, isWritable: false }, // Rent account
			{
				pubkey: SystemProgram.programId,
				isSigner: false,
				isWritable: false,
			}, // System program
			{ pubkey: TOKEN_PROGRAM_ID, isSigner: false, isWritable: false }, // Token program
			{
				pubkey: TOKEN_METADATA_PROGRAM_ID,
				isSigner: false,
				isWritable: false,
			}, // Token metadata program
		],
		data: instructionData.serialize(),
		programId: programId,
	});

	const modifyComputeUnits = ComputeBudgetProgram.setComputeUnitLimit({
		units: 1000000,
	});

	const tx = new Transaction().add(modifyComputeUnits).add(ix);

	return await sendAndConfirmTransaction(connection, tx, [
		payer,
		mintKeypair,
	]);
}
```

In our script above, we modify our tx and increase our compute units. This will prevent us from running into unwanted, `compute limit exceeded errors`.

**Note:**, you need to pass in all account that sign, in the signers array. In our case, these are, `payer` and `mintKeypair`.

```ts
createToken()
	.then((txSig) =>
		console.log(`tx hash: https://solscan.io/tx/${txSig}?cluster=devnet`)
	)
	.catch((err) => console.error("program execution unsuccessful", err));
```

And finally, finally, let's call our client script, `npm run dev`.

Open the link returned after running your script. It should take you to the solscan explorer.

![Screenshot of the result of calling our program!](/solana-nft-native/tx-hash-solscan.png "tx hash")

To look at our token on solscan, we will click on the second signer. The one surrounded by a yellow border. That is our token mint account.

Open the second account and it should show you the details about the nft you created.

![Screenshot of our nft page on solscan!](/solana-nft-native/token-page-solscan.png "token page solscan")

Find the code for the [client side here](https://github.com/687c/solana-nft-native-client) and for the [Solana program here](https://github.com/687c/solana-nft-native).

And finally, huge shout out to the Solana [program examples repository](https://github.com/solana-developers/program-examples) from where the code for this article was lifted. 😂

In the next part we modify our code to mint the tokens directly to our wallet or a specified address.

## Further Reading <a name="further-reading"></a>

-   [Soldev Course Chapter](https://www.soldev.app/course/token-program) on Tokens.
-   [Anoushk's Great write-up](https://mirror.xyz/anoushk.eth/HLL3JE47SJE5AnvzMVrfz7Rec7FDupXK5948aYNsDUU) using anchor.
-   [Solana Program Examples](https://github.com/solana-developers/program-examples/tree/main/tokens/nft-minter)
