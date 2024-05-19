+++
title = "Mint Solana NFT Using Anchor and Metaplex"
date = 2023-09-25
category = "Prog"
draft=false

[taxonomies]
tags = ["Rust", "solana"]
+++

In the [previous article](../mint-solana-rust) I showed you how to mint Solana NFT with the Metaplex sdk and the Solana SDK in native Rust.

In this article we will mint our NFt using [anchor](https://www.anchor-lang.com/), which is a Solana program framework. Learn more about anchor in this [link](https://https://www.anchor-lang.com) <!-- todo ADD LINK TO SOLANA MODULE HERE -->

<!-- more -->

# Table of contents

-   [Environmental Setup](#env-setup)
-   [Interacting With Token Program](#token-program)
-   [Interacting With Metaplex Metadata Program](#metadata-program)
-   [Calling Our Program from TS client](#ts-client)

**Note:** when reading this article a `// snip` or `# snip` comment means that code has been cut.

## Environmental Set-up <a name="env-setup"></a>

You can skip this setup if you already have your environment set up properly.

For the uninitiated in Solana, we will get started by first installing the Solana Suite which is provided as a CLI suite.

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

We will finally install the anchor-cli which is provided by the anchor version manager. Find the guide on this [official page](https://www.anchor-lang.com/docs/installation).

Install avm using cargo,

```bash
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
```

Use avm to install anchor cli

```bash
avm install latest
avm use latest
```

confirm anchor is installed properly by running

```bash
anchor --version
```

At the time of writing this, here is how my environment look like, after running `anchor --version && solana --version`

```bash
anchor-cli 0.28.0
solana-cli 1.16.13 (src:b2a38f65; feat:3949673676, client:SolanaLabs)
```

Once everything is set-up properly, we can get started using the `anchor cli` to initialize our new anchor project with.

```bash
anchor init solana-nft-anchor
```

Change into your created anchor project and open it up with your preferred text editor. In our case, using vscode, we'll run

```bash
cd solana-nft-anchor
code .
```

The first thing we will need is to enable `init_if_needed` feature for anchor-lang crate. This feature allows us to create an account if it isn't created automatically for us.

```rust
cargo add anchor-lang --features=init-if-needed
```

We will also install the `anchor-spl` crate with `metadata` features. The `anchor-spl` create provides us with almost all the necessary functionality required to interact with the Token Program while the `metadata` feature brings into scope functionality required to interact the Metaplex's Token Metadata Program.

```bash
cargo add anchor-spl --features=metadata
```

Running a quick `anchor build`, ensures that everything is working in tandem with each other and we are not dealing with dependency conflicts.

We are now ready to start writing our anchor program to mint nfts for us. Open the `programs/solana-nft-anchor/lib.rs` file and it should have contents similar to this

```rust
use anchor_lang::prelude::*;

declare_id!("9TEtkW972r8AVyRmQzgyMz8GpG7WJxJ2ZUVZnjFNJgWM"); // should'nt be similar to mine

#[program]
pub mod solana_nft_anchor {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

If you encounter this error `struct takes 0 lifetime arguments but lifetime argument was supplied expected 0 lifetime arguments`, you can ignore it as it will go away as soon as we start working on our code.

We will rename `Initialize` to `InitNFT` and bring into scope the accounts we'll interact with when creating our nft in the `Accounts` struct.

The first account we will bring into scope will be the signer. This account the authority, fee payer for the transactions we make and signer of the transaction.

```rust

// snip

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    signer: AccountInfo<'info>
}
```

But how does solana [`parallel program execution layer(Sealevel)`](https://medium.com/solana-labs/sealevel-parallel-processing-thousands-of-smart-contracts-d814b378192) differentiate a signer account from a normal one? We mark it as one. There are two ways to do this, using the [`Signer `](https://docs.rs/anchor-lang/latest/anchor_lang/accounts/signer/struct.Signer.html) account variant or by using anchor constraints which is what we are doing.
Anchor constraints defined using `#[account(<constraints>)]` are built-in features to simplify common security checks, e.g mark accounts as mutable or not.

In our code snipped above, we have marked the account as a mutable account(because we are going to be mutating the account balance when paying for the transactions) and as a signer using `#[account(mut, signer)]`

To also note is the use of Rustdoc comment `/// CHECK: ok, we are passing in this account ourselves`. This is vital when using the `AccountInfo` wrapper. We will discuss this after introducing the `Account` wrapper in the next section.

---

## Interacting With Token Program <a name="token-program"></a>

To create our NFT we will need to interact with the token program and the metadata program. When working in anchor and Solana, it is required to **explicitly declare** all the accounts which you'll interact with.

Which bring us to the second account which we bring into scope, the `Mint` account. The mint account of a token contains details about the token such as mint authority, freeze authority, total supply ...etc.

![Screenshot of details the mint account contains!](/solana-nft-anchor/mint-account.png "mint account")

<!-- ! WE ARE BLINDLY TRUSTING THIS ACCOUNT BEING PASSED IS CORRECT -->

```rust
use anchor_spl::token::Mint;

// snip

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    signer: AccountInfo<'info>,
    #[account(
        init,
        payer = signer,
        mint::decimals = 0,
        mint::authority = signer.key(),
        mint::freeze_authority = signer.key(),
    )]
    mint: Account<'info, Mint>,
}
```

We have also added more constraints for our `mint` account. Let's go over them. The first constraint `init` is like "a wrapper" around the `system_instruction::create_account()` functions which instructs the System Program to create the account. this initialization takes place in three steps: allocate space, transfer lamports for rent and assign the account to the owning program. This is where the second constraint comes in `payer = signer` is used to pay the rent for the account creation. what is rent? For you to store data on Solana, you must pay a sort of deposit. This incentivizes the validators to store your data. If not paid, your data will be pruned from the blockchain. [Read more here](https://bpf.wtf/sol-state-history/).
The next set of constraints `mint::decimals = 0` set the decimals of our nft token. You can't have a 0.25 NFT!. Finally we set the `mint::authority = signer.key(), mint::freeze_authority = signer.key(),` field to our address.

Looking at this newly added account, the declaration is different from the first account. We are now using the `Account` account type instead of `AccountInfo`.

The anchor **AccountInfo** type is a way to define accounts that does not implement any checks on the account being passed. We are blindly trusting the account being passed as the correct account without verifying the structure of the data or owner of the account. As such we have to also explicitly mark it as trustworthy using the rustdoc comments `/// CHECK: <comment explaining why are we blindly trusting this account>`.

The **Account** account type is a more secure way of declaring your accounts. It contains all the methods of `AccountInfo`, but it verifies program ownership and deserializes underlying data into the specified type. In our above, Account checks that the owner of the our `mint` is indeed the Token program `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` and that the account contains all the required fields for a Mint account.

The fields contained that the **Mint account** contains

-   **mint_authority** - Address allowed to mint more token. For our NFT, this field is going to be set to zero. No one is allowed to mint more tokens. (useful for fungible tokens that have an unlimited supply)
-   **supply**: The total supply of tokens. In our case when minting an NFT, the supply is going to be set to 1.
-   **decimals**: Number of decimals to consider when interpreting the balance of the token. For our NFT this is going to be zero. Why? Because there's no such thing as having 0.5 NFT.
-   **is_initialized**: Has the token mint been initialized. True, when we call the mint instruction.
-   **freeze_authority**: Address allowed to freeze the token account.

**Note:** The mint does not hold any tokens.

If the mint account does not hold your tokens then where are they stored? When creating a token on Solana, you need the **Token account** to hold your newly minted tokens. The token account contains fields such as

-   **mint** - The mint address associated with this account.
-   **owner** - The address that has authority over this account.
-   **amount** - The number of tokens in this account.
-   **delegate** - Address of another address that can manager your token account. That means, transfer or freeze your asset.
-   **state** - The account state. It is an enum of three possible values, **Uninitialized**, for when the account does not exists, **Initialized**, for when the account has been created and exists, **Frozen**, for when the account has been frozen by the freeze authority.
-   **is_native**, is this token the native solana token.
-   **delegated_amount** - amount delegated to the `delegate` field mentioned above.
-   **close_authority** - Address which can close this account.

But, there is a downside with this method.

The first downside. Suppose you an NFT hoarder, having collected 1000 NFT. When your friend wants to send you an NFT from a mint which you already own, he will need to know the correct token account to send this NFT to. That means, keeping track of all those 1000 token accounts.

The second downside. Suppose you want to introduce you non-crypto native friend into NFTs. Your friend has never minted from the collection before. If you want to send him his first NFT from a collection he has never minted, your friend needs to have a token account from that mint's NFT. This makes transferring assets difficult and cumbersome. It also means that airdrop campaigns become impossible.

This is where the motivation to reduce the friction when working with solana tokens came in, which lead to spec a new way for the token account to map to the user's wallet, using the [**Associated Token Account**](https://spl.solana.com/token).

The **Associated Token Account** is a PDA that is deterministically derived using the `Associated Token Program`, the `user's address` and `mint account`.

Let's take the accounts required for the mint

```rust
use anchor_lang::prelude::*;
use anchor_spl::{
    associated_token::AssociatedToken,
    token::{Mint, Token, TokenAccount},
};

// snip

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    pub signer: AccountInfo<'info>,
    #[account(
        init,
        payer = signer,
        mint::decimals = 0,
        mint::authority = signer.key(),
        mint::freeze_authority = signer.key(),
    )]
    pub mint: Account<'info, Mint>, // new
    #[account(
        init_if_needed,
        payer = signer,
        associated_token::mint = mint,
        associated_token::authority = signer,
    )]
    pub associated_token_account: Account<'info, TokenAccount>, // new

    pub token_program: Program<'info, Token>, // new
    pub associated_token_program: Program<'info, AssociatedToken>, // new
    pub system_program: Program<'info, System>, // bew
    pub rent: Sysvar<'info, Rent> // new
}
```

We have the accounts we will be interacting with when creating our mint and token accounts.

The `mint` account is defined by the `Mint` Account type.

We have an `associated_token_account`, with multiple anchor constraints. We use the `init_if_needed` flag to initialize this token account, if it does not exist on our wallet and to use this feature you need to defined a payer who will cover the cost associated with creating a new account. We also pass in the `authority` and `mint` as constraints, to link the mint to the token account.

I have also added a space to serve as a visual separation between the accounts and Programs. Remember(I know I sound like a broke record, but its important to remember this), everything on Solana is an account. This Programs are also accounts. The difference is that they have their `executable` field marked as true. Read more about [Solana account model](https://solanacookbook.com/core-concepts/accounts.html#account-model).

Anchor also provides another primitive to make working with Programs easier, We use `Program` to mark an account as executable. This also implements similar checks to Accounts. Meaning you can't pass in the address of a malicious program trying to pass it off as the Token Program. The `ID`s won't match and the programs executions will be halted.

Let's talk about the four programs.

1. `token_program` - the Token program
2. `associated_token_program` - handles creation of our ATA(Associated Token Account).
3. `system_program` - because the associated token program might end up needing to create a new ATA, we need to pass in this program which is responsible for creating all accounts.
4. `rent` - on Solana, you need to pay for space when you are storing data on the blockchain. All accounts on Solana(now) are required to be rent-exempt, that means putting down a 2-years worth of sol to store data on the chain. (not that expensive by the way). As such we need to interact with the rent program for this.

With that, we are now ready to call the instruction to create our first NFT on Solana. Let us first call the instructions to create the mint and token account. To do this we will need to interact with the Token Program. One way to do this would be to use [`invoke`](https://docs.solana.com/developing/programming-model/calling-between-programs) as I did in my [previous article](../mint-solana-rust) and make a cross program call into it. Using invoke is tricky because it requires you to pass in the correct number of accounts and correct accounts you are going to interact with. <!-- This is so because Solana executes program in parallel and this having all the account that it will read and write to, makes it easier to parallelize tasks such as a million requests to only read from a certain account -->. You miss one account and boom, your tx fails and you get a cryptic error telling you, "program failed because you missed an account" and yet it can't tell you which account you missed 🤦. Enough of my rant.

Anchor simplifies this for us, by having a [`CpiContext`](https://docs.rs/anchor-lang/latest/anchor_lang/context/struct.CpiContext.html) struct which encapsulates all accounts that we need to interact with when making our CPI call. It also has already defined methods for commonly called instructions.

<!-- todo: review below -->

Let's take a look at how we can use the CpiContext to initialize a token mint. To initialize the `CpiContext`, call the associative function `new` which takes in two arguments.

1. The external program we are cpi-ing into.
2. Anchor defined accounts we will pass in to make the call to the external program successful. As opposed to a normal `invoke` invocation to an external program, using anchor defined Accounts means we will only declare the account which are vital and not all the programs we which almost alway never change.

For example, let take a look at the code to initialize a new token mint and associated token account using it.

```rust
 pub fn init_nft(ctx: Context<InitNFT>) -> Result<()> {
        // create mint account
        let cpi_context = CpiContext::new(
            ctx.accounts.token_program.to_account_info(),
            MintTo {
                mint: ctx.accounts.mint.to_account_info(),
                to: ctx.accounts.associated_token_account.to_account_info(),
                authority: ctx.accounts.signer.to_account_info(),
            },
        );

        mint_to(cpi_context, 1)?;
        Ok(())
    }
```

As highlighted above, we first create our cpi context by calling `CpiContext::new()` method. We are interacting with the token program and thus we pass it in first. For the second send we will pass in a struct that contains defines the accounts we will interact with.

Here is all the code together.

```rust
use anchor_lang::prelude::*;
use anchor_spl::{
    associated_token::AssociatedToken,
    token::{mint_to, Mint, MintTo, Token, TokenAccount},
};

declare_id!("9TEtkW972r8AVyRmQzgyMz8GpG7WJxJ2ZUVZnjFNJgWM"); // shouldn't be similar to mine

#[program]
pub mod solana_nft_anchor {

    use super::*;

    pub fn init_nft(ctx: Context<InitNFT>) -> Result<()> {
        // create mint account
        let cpi_context = CpiContext::new(
            ctx.accounts.token_program.to_account_info(),
            MintTo {
                mint: ctx.accounts.mint.to_account_info(),
                to: ctx.accounts.associated_token_account.to_account_info(),
                authority: ctx.accounts.signer.to_account_info(),
            },
        );

        mint_to(cpi_context, 1)?;
        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    pub signer: AccountInfo<'info>,
    #[account(
        init,
        payer = signer,
        mint::decimals = 0,
        mint::authority = signer.key(),
        mint::freeze_authority = signer.key(),
    )]
    pub mint: Account<'info, Mint>,
    #[account(
        init_if_needed,
        payer = signer,
        associated_token::mint = mint,
        associated_token::authority = signer,
    )]
    pub associated_token_account: Account<'info, TokenAccount>,

    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub system_program: Program<'info, System>,
    pub rent: Sysvar<'info, Rent>,
}
```

<!-- todo: code for the TS client -->

Calling this method as it is would create our nft, but what good is a monkey non fungible without the monkey image? 😂. In the next section we dive into using the Metaplex Token Metadata Program.

---

## Interacting With Metaplex Metadata Program <a name="metadata-program"></a>

Metaplex offers a collection of tools, smart contracts and more, designed to make the process of creating and launching NFTs easier.

In this guide we shall be using their [`Token Metadata Program`](https://developers.metaplex.com/token-metadata) to add metadata to our spl-token.

<!-- todo: push this PDA's explanation to when talking about ATAs? -->

But before that, we need to understand how Metaplex works under the hood. Remember ATAs(Associated Token Account). They are part of a special type of accounts owned and controlled by a program (smart contract) known as `Program Derived Accounts`. Simply put, they are public keys that do not have a corresponding public key. They have various use-cases such as signing transactions, storing SOL and storing data, as used seen in ATAs. They are usually derived using the program public key and seeds which are chosen by the developer and passed into the [` find_program_address()`](https://docs.rs/solana-program/latest/solana_program/pubkey/struct.Pubkey.html#method.find_program_address) function as bytes. This sha256 hash function to look for an address that is not on the ed25519 elliptic curve (addresses on the curve are keypairs). Further details about PDAs can be [found here](https://solanacookbook.com/core-concepts/pdas.html).

The Metaplex Metadata Program also uses PDAs. Like the Associated Token Program, the Metadata Program uses PDAs for the **metadata account** that attaches itself to the Mint Account. The metadata account is derived using the following seeds, `metadata`, Token Metadata Program pubkey i.e `metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s` and finally the public key of the `mint` account. Anchor has a different way of deriving PDAs but here is the code snippet for doing the above using a native Solana Rust program.

```rust
let metadata_seeds = &[PREFIX.as_bytes(), program_id.as_ref(), mint_pubkey.as_ref()];
let (pubkey, _) = Pubkey::find_program_address(metadata_seeds, &ID);
```

Now that we know how to derive the metadata account let's look at the fields that this account holds, show in the image below.

![Diagram of the details contained in the Metadata Account!](/solana-nft-anchor/metadata-pda-fields.png "metadata account")

---

**key**. This is the first field and it is an enum that lets Metaplex identify the 'type of metaplex account', to work with. It similar to the [anchor discriminator](https://book.anchor-lang.com/anchor_bts/discriminator.html). Why is it needed. The token program accesses different accounts such as the `edition account` for print editions of an NFT, the `token record` for programmable NFTs e.t.c. In our case for the, metadata account, this field is marked with the `MetadataV1` enum variant.

Here is an exhaustive list of these markers and accounts and an explanation of what they are used for.

-   _Uninitialized_: The account does not exist yet and needs to be created.
-   [_MetadataV1_](https://docs.metaplex.com/programs/token-metadata/accounts#metadata) This account holds the token metadata(the one we're currently working with).
-   [_MasterEditionV1_ and _MasterEditionV2_](https://docs.metaplex.com/programs/token-metadata/accounts#master-edition) The master edition account allows NFTs to be **printed** a limited or unlimited times amount of times. When we say printed, what we mean is making copies of the NFT
-   [_EditionV1_](https://docs.metaplex.com/programs/token-metadata/accounts#edition) The edition account derived from a mint account represents an NFT that was copied from a Master Edition NFT.
-   [_EditionMarker_ and _EditionMarkerV2_](https://docs.metaplex.com/programs/token-metadata/accounts#edition-marker) This account is used to internally keep track of which editions were printed and which ones have not.
-   [_TokenRecord_](https://docs.metaplex.com/programs/token-metadata/accounts#token-record) used by programmable NFTs only. Token Record accounts enable us to attach custom data to token accounts rather than mint account.
-   [_MetadataDelegate_](https://docs.metaplex.com/programs/token-metadata/accounts#metadata-delegate-record) These accounts are used to store multiple delegate authorities for a given metadata account.
-   [_CollectionAuthorityRecord_](https://docs.metaplex.com/programs/token-metadata/accounts#collection-authority-record) Keeps track of which authorities are allowed to set and/or verify the collection of the metadata account.
-   [_UseAuthorityRecord_](https://docs.metaplex.com/programs/token-metadata/accounts#use-authority-record) Keeps track of which authorities are allowed to reduce the uses([might be deprecated soon](https://github.com/metaplex-foundation/mip/discussions/27)) field on the metadata account.
-   [_TokenOwnedEscrow_](https://docs.metaplex.com/programs/token-metadata/nft-escrow#token-owned-escrow) An escrow account that is managed by the holder of the NFT.
-   _ReservationListV1_ and _ReservationListV2_ (DEPRECATED) Used for reservation lists. if present on the list you can get an edition number given by your position on the list.

<!-- **Note:** Always use the latest of these as Metaplex is constantly adding new features and [deprecating other features](https://github.com/metaplex-foundation/mip/blob/main/mip-6.md). For example, even though `MasterEditionV1` is supported, `MasterEditionV2` supports -->

---

**update_authority** - address allowed to make changes to the metadata account.

---

**mint** - address of the mint account.

---

**data** - the asset data for with the token. This includes values such as the _name_, _symbol_, _uri_, _creator royalties_ and the _creators_

---

**primary_sale_happened** - indicates whether the token has already been sold at least once.

---

**is_mutable** - indicates whether the metadata account can ever be changed.

---

**edition_nonce** - nonce used to verify the edition nonce's of printed NFTs.

---

**token_standard** - the type of token that the metadata account holds. It is an optional enum that consists of the following variants

-   _NonFungible_ - A non-fungible with a master edition.
-   _FungibleAsset (semi fungible)_ - An spl token with a supply > 1, but which has NFT attributes such as an image and an attributes array.
-   _Fungible_ - A token with metadata and supply > 1.
-   _ProgrammableNonFungible_ - Special type of NonFungible that's frozen al times and enforces custom authorization rules.

---

**collection** - an optional struct that contains the public key of a collectionNft or None if not present.

---

**uses** - Another optional field that makes NFTs usable. Meaning you can load it with a certain amount of "uses" and use it until it has run out.

There's a [proposal to deprecate it](https://github.com/metaplex-foundation/mip/discussions/27) soon.

---

**collection_details** - An optional field with a V1 field that contains the number of NFTs for an NFT collection.

_It's deprecated and might be removed soon_

---

**programmable_config** - Also an optional field that if set, contains the address to an optional rule set account which contains constraints pertaining to the programmable non-fungible.

Enough talk, let's Buidl!!!!

In our `InitNFT` struct we will bring into scope the account we need to add metadata to our spl-token. These are, `metadata_account` to store our metadata, `master_edition_account` to set up the master edition NFT. We haven't talked about the master edition account. Why is it important? It proves the non-fungibility of our token. It will check if the decimals field on the mint account is indeed zero and that the supply field is set to 1. It is also useful in determining if we can make a print edition from the NFT. We will finally bring into scope the token_metadata_program which is responsible for processing the instructions to create both the accounts and instantiate them correctly with the fields we provide.

To put the above paragraphs into code, we first install a version `mpl-token-metadata` crate.

```bash
cargo add mpl-token-metadata@1.13.1
```

```rust
use anchor_spl::{
    associated_token::AssociatedToken,
    metadata::{Metadata}, // new
    token::{mint_to, Mint, MintTo, Token, TokenAccount},
};
use mpl_token_metadata::{
    pda::{find_master_edition_account, find_metadata_account}, // new
};

// snip

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    pub signer: AccountInfo<'info>,
    #[account(
        init,
        payer = signer,
        mint::decimals = 0,
        mint::authority = signer.key(),
        mint::freeze_authority = signer.key(),
    )]
    pub mint: Account<'info, Mint>,
    #[account(
        init_if_needed,
        payer = signer,
        associated_token::mint = mint,
        associated_token::authority = signer,
    )]
    pub associated_token_account: Account<'info, TokenAccount>,
    /// CHECK - address
    #[account(
        mut,
        address=find_metadata_account(&mint.key()).0,
    )]
    pub metadata_account: AccountInfo<'info>, // new
    /// CHECK: address
    #[account(
        mut,
        address=find_master_edition_account(&mint.key()).0,
    )]
    pub master_edition_account: AccountInfo<'info>, // new

    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub token_metadata_program: Program<'info, Metadata>, // new
    pub system_program: Program<'info, System>,
    pub rent: Sysvar<'info, Rent>,
}
```

To make sure that the right accounts are passed in, we are using the `address=""` constraint to make sure that the accounts passed in are indeed the metadata and master edition account respectively. As we did before, un typed accounts should be accompanied by a rustdoc comment `/// CHECK: <reason why the account is untyped>` explaining why it is untyped. You might be wondering why we did not use the typed metadata `Account<'info, MetadataAccount>` and master edition `Account<'info, MasterEditionAccount>` account. That is because anchor expects those account types to be initialized beforehand.

After bringing the needed accounts and program into scope, we now need to instantiate them. The `anchor-spl` with `metadata` crate comes with a useful number of Cross Program Invocation(CPI) instructions we can use.

As we did with the mint account we will be using the `CpiContext::new()` method to help us make sure we have all the required accounts when making the CPI call to the `metadata_program`.

We will use the `create_metadata_accounts_v3()` to create the metadata account. It takes in five arguments,

1. Our CpiContext struct with the required program id and accounts
2. Our asset data aptly named as DataV2. name, symbol, uri ... are all defined here.
3. is_mutable, a boolean determining on whether we can make changes to our metadata_account.
4. update_authority_is_signer, a boolean on whether the update authority is going to be the signer creating this transaction.
5. collection details, optional field containing no. of NFTs in our collection.

```rust
// snip
use anchor_spl::{
    associated_token::AssociatedToken,
    metadata::{
        create_master_edition_v3, create_metadata_accounts_v3, CreateMasterEditionV3,
        CreateMetadataAccountsV3, Metadata, MetadataAccount,
    }, // new
    token::{mint_to, Mint, MintTo, Token, TokenAccount},
};
use mpl_token_metadata::{
    pda::{find_master_edition_account, find_metadata_account},
    state::DataV2 // new
};

// snip

pub fn init_nft(
    ctx: Context<InitNFT>,
    name: String,   // new
    symbol: String, // new
    uri: String,    // new
) -> Result<()> {
    // create mint account
    let cpi_context = CpiContext::new(
        ctx.accounts.token_program.to_account_info(),
        MintTo {
            mint: ctx.accounts.mint.to_account_info(),
            to: ctx.accounts.associated_token_account.to_account_info(),
            authority: ctx.accounts.signer.to_account_info(),
        },
    );

    mint_to(cpi_context, 1)?;

    // create metadata account
    let cpi_context = CpiContext::new(
        ctx.accounts.token_metadata_program.to_account_info(),
        CreateMetadataAccountsV3 {
            metadata: ctx.accounts.metadata_account.to_account_info(),
            mint: ctx.accounts.mint.to_account_info(),
            mint_authority: ctx.accounts.signer.to_account_info(),
            update_authority: ctx.accounts.signer.to_account_info(),
            payer: ctx.accounts.signer.to_account_info(),
            system_program: ctx.accounts.system_program.to_account_info(),
            rent: ctx.accounts.rent.to_account_info(),
        },
    );

    let data_v2 = DataV2 {
        name: name,
        symbol: symbol,
        uri: uri,
        seller_fee_basis_points: 0,
        creators: None,
        collection: None,
        uses: None,
    };

    create_metadata_accounts_v3(cpi_context, data_v2, false, true, None)?;

    Ok(())
}
```

Because the `DataV2` struct need to take in the asset data, we have changed the definition of our `init_nft` function to also include the `name`, `symbol` and `uri` parameter which we are passing as our asset data. This is also the same way we would get the other fields in the `DataV2` struct such as the `seller_fee_basis_points`, `creators` if there were more than one ... e.t.c.

And finally we finish off our program by creating the master edition account, by invoking the `create_master_edition_v3` instruction, which takes in the accounts needed to initialize the master edition account and an optional max_supply argument. The max_supply takes in the maximum number of editions that can be printed from this NFT. We do not want to allow print editions to be made from our NFT and we set it to None.

```rust
use anchor_spl::{
    associated_token::AssociatedToken,
    metadata::{
        create_master_edition_v3, create_metadata_accounts_v3, CreateMasterEditionV3,
        CreateMetadataAccountsV3, Metadata,
    }, // new
    token::{mint_to, Mint, MintTo, Token, TokenAccount},
};

// snip

//create master edition account
let cpi_context = CpiContext::new(
    ctx.accounts.token_metadata_program.to_account_info(),
    CreateMasterEditionV3 {
        edition: ctx.accounts.master_edition_account.to_account_info(),
        mint: ctx.accounts.mint.to_account_info(),
        update_authority: ctx.accounts.signer.to_account_info(),
        mint_authority: ctx.accounts.signer.to_account_info(),
        payer: ctx.accounts.signer.to_account_info(),
        metadata: ctx.accounts.metadata_account.to_account_info(),
        token_program: ctx.accounts.token_program.to_account_info(),
        system_program: ctx.accounts.system_program.to_account_info(),
        rent: ctx.accounts.rent.to_account_info(),
    },
);

create_master_edition_v3(cpi_context, None)?;
```

In ~115 LOC we have written a program that mints an NFT for us on-chain. Here is what the full finished program looks like.

```rust
use anchor_lang::prelude::*;
use anchor_spl::{
    associated_token::AssociatedToken,
    metadata::{
        create_master_edition_v3, create_metadata_accounts_v3, CreateMasterEditionV3,
        CreateMetadataAccountsV3, Metadata,
    },
    token::{mint_to, Mint, MintTo, Token, TokenAccount},
};
use mpl_token_metadata::{
    pda::{find_master_edition_account, find_metadata_account},
    state::DataV2,
};

declare_id!("9TEtkW972r8AVyRmQzgyMz8GpG7WJxJ2ZUVZnjFNJgWM"); // shouldn't be similar to mine

#[program]
pub mod solana_nft_anchor {

    use super::*;

    pub fn init_nft(
        ctx: Context<InitNFT>,
        name: String,
        symbol: String,
        uri: String,
    ) -> Result<()> {
        // create mint account
        let cpi_context = CpiContext::new(
            ctx.accounts.token_program.to_account_info(),
            MintTo {
                mint: ctx.accounts.mint.to_account_info(),
                to: ctx.accounts.associated_token_account.to_account_info(),
                authority: ctx.accounts.signer.to_account_info(),
            },
        );

        mint_to(cpi_context, 1)?;

        // create metadata account
        let cpi_context = CpiContext::new(
            ctx.accounts.token_metadata_program.to_account_info(),
            CreateMetadataAccountsV3 {
                metadata: ctx.accounts.metadata_account.to_account_info(),
                mint: ctx.accounts.mint.to_account_info(),
                mint_authority: ctx.accounts.signer.to_account_info(),
                update_authority: ctx.accounts.signer.to_account_info(),
                payer: ctx.accounts.signer.to_account_info(),
                system_program: ctx.accounts.system_program.to_account_info(),
                rent: ctx.accounts.rent.to_account_info(),
            },
        );

        let data_v2 = DataV2 {
            name: name,
            symbol: symbol,
            uri: uri,
            seller_fee_basis_points: 0,
            creators: None,
            collection: None,
            uses: None,
        };

        create_metadata_accounts_v3(cpi_context, data_v2, false, true, None)?;

        //create master edition account
        let cpi_context = CpiContext::new(
            ctx.accounts.token_metadata_program.to_account_info(),
            CreateMasterEditionV3 {
                edition: ctx.accounts.master_edition_account.to_account_info(),
                mint: ctx.accounts.mint.to_account_info(),
                update_authority: ctx.accounts.signer.to_account_info(),
                mint_authority: ctx.accounts.signer.to_account_info(),
                payer: ctx.accounts.signer.to_account_info(),
                metadata: ctx.accounts.metadata_account.to_account_info(),
                token_program: ctx.accounts.token_program.to_account_info(),
                system_program: ctx.accounts.system_program.to_account_info(),
                rent: ctx.accounts.rent.to_account_info(),
            },
        );

        create_master_edition_v3(cpi_context, None)?;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitNFT<'info> {
    /// CHECK: ok, we are passing in this account ourselves
    #[account(mut, signer)]
    pub signer: AccountInfo<'info>,
    #[account(
        init,
        payer = signer,
        mint::decimals = 0,
        mint::authority = signer.key(),
        mint::freeze_authority = signer.key(),
    )]
    pub mint: Account<'info, Mint>,
    #[account(
        init_if_needed,
        payer = signer,
        associated_token::mint = mint,
        associated_token::authority = signer
    )]
    pub associated_token_account: Account<'info, TokenAccount>,
    /// CHECK - address
    #[account(
        mut,
        address=find_metadata_account(&mint.key()).0,
    )]
    pub metadata_account: AccountInfo<'info>,
    /// CHECK: address
    #[account(
        mut,
        address=find_master_edition_account(&mint.key()).0,
    )]
    pub master_edition_account: AccountInfo<'info>,

    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub token_metadata_program: Program<'info, Metadata>,
    pub system_program: Program<'info, System>,
    pub rent: Sysvar<'info, Rent>,
}
```

To finish off the program side of development, we are going to build and deploy our program.

Let's first configure our `Anchor.toml`, to deploy to Devnet when we run the deploy command.

To do this we will change the `cluster` to point to `devnet`

```toml
# snip

[provider]
cluster = "devnet"
# snip
```

We are now ready to deploy to devnet. Go ahead and build your program and deploy using the following commands.

```bash
anchor build

anchor deploy
```

If commands run accordingly, you should be greeted with a `Deploy success` message.

<!-- todo: testing anchor programs locally -->

## Calling Our Program from TS client <a name="ts-client"></a>

For our client we will be using [umi](https://github.com/metaplex-foundation/umi) from Metaplex. Umi is a "Solana Framework for JavaScript clients".

Let's install the packages we will be using. Umi will help us in deriving the pda's for the metadata account and master edition while spl token will help us when deriving the associated token account.

```bash
yarn add @solana/spl-token @metaplex-foundation/mpl-token-metadata @metaplex-foundation/umi @metaplex-foundation/umi-bundle-defaults @metaplex-foundation/umi-signer-wallet-adapters
```

Once installed, we will go ahead and create a new Umi instance using the `createUmi` function and register our local provider wallet with the token metadata program with umi's interfaces.

```ts
// snip

const provider = anchor.AnchorProvider.env();
anchor.setProvider(provider);
const program = anchor.workspace.SolanaNftAnchor as Program<SolanaNftAnchor>;

const signer = provider.wallet;

const umi = createUmi("https://api.devnet.solana.com")
	.use(walletAdapterIdentity(signer))
	.use(mplTokenMetadata());

//snip
```

With our config set up, let work on calling our `init_nft` method. We will need to derive our associated token account, metadata account and master edition. Operations which are made easy with the helper functions from the packages we just installed.

**Note:** the `@solana/web3.js` PublicKey interface is not compatible with umi's publicKey interface so be sure to wrap the public key values with it should an umi functions require it as input.

```ts
// generate the mint account
const mint = anchor.web3.Keypair.generate();

// Derive the associated token address account for the mint
const associatedTokenAccount = await getAssociatedTokenAddress(
	mint.publicKey,
	signer.publicKey
);

// derive the metadata account
let metadataAccount = findMetadataPda(umi, {
	mint: publicKey(mint.publicKey),
})[0];

//derive the master edition pda
let masterEditionAccount = findMasterEditionPda(umi, {
	mint: publicKey(mint.publicKey),
})[0];
```

Once we have derived our public keys, we will also need the asset data, i.e the name, symbol and uri. We will not do metadata upload in this tutorial but in a separate umi guide later.

Right now we will utilize the data that I use for the previous tutorial.

```ts
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { SolanaNftAnchor } from "../target/types/solana_nft_anchor";
import { walletAdapterIdentity } from "@metaplex-foundation/umi-signer-wallet-adapters";
import { getAssociatedTokenAddress } from "@solana/spl-token";
import {
	findMasterEditionPda,
	findMetadataPda,
	mplTokenMetadata,
	MPL_TOKEN_METADATA_PROGRAM_ID,
} from "@metaplex-foundation/mpl-token-metadata";
import { createUmi } from "@metaplex-foundation/umi-bundle-defaults";
import { publicKey } from "@metaplex-foundation/umi";

import {
	TOKEN_PROGRAM_ID,
	ASSOCIATED_TOKEN_PROGRAM_ID,
} from "@solana/spl-token";

describe("solana-nft-anchor", async () => {
	// Configured the client to use the devnet cluster.
	const provider = anchor.AnchorProvider.env();
	anchor.setProvider(provider);
	const program = anchor.workspace
		.SolanaNftAnchor as Program<SolanaNftAnchor>;

	const signer = provider.wallet;

	const umi = createUmi("https://api.devnet.solana.com")
		.use(walletAdapterIdentity(signer))
		.use(mplTokenMetadata());

	const mint = anchor.web3.Keypair.generate();

	// Derive the associated token address account for the mint
	const associatedTokenAccount = await getAssociatedTokenAddress(
		mint.publicKey,
		signer.publicKey
	);

	// derive the metadata account
	let metadataAccount = findMetadataPda(umi, {
		mint: publicKey(mint.publicKey),
	})[0];

	//derive the master edition pda
	let masterEditionAccount = findMasterEditionPda(umi, {
		mint: publicKey(mint.publicKey),
	})[0];

	const metadata = {
		name: "Kobeni",
		symbol: "kBN",
		uri: "https://raw.githubusercontent.com/687c/solana-nft-native-client/main/metadata.json",
	};

	it("mints nft!", async () => {
		const tx = await program.methods
			.initNft(metadata.name, metadata.symbol, metadata.uri)
			.accounts({
				signer: provider.publicKey,
				mint: mint.publicKey,
				associatedTokenAccount,
				metadataAccount,
				masterEditionAccount,
				tokenProgram: TOKEN_PROGRAM_ID,
				associatedTokenProgram: ASSOCIATED_TOKEN_PROGRAM_ID,
				tokenMetadataProgram: MPL_TOKEN_METADATA_PROGRAM_ID,
				systemProgram: anchor.web3.SystemProgram.programId,
				rent: anchor.web3.SYSVAR_RENT_PUBKEY,
			})
			.signers([mint])
			.rpc();

		console.log(
			`mint nft tx: https://explorer.solana.com/tx/${tx}?cluster=devnet`
		);
		console.log(
			`minted nft: https://explorer.solana.com/address/${mint.publicKey}?cluster=devnet`
		);
	});
});
```

Let go ahead and run the test to mint our nft.

```bash
anchor test
```

Running anchor test will build and deploy your program first before running the ts client first.

To skip this process you can add the `--skip-build` and `--skip-deploy` flags. I prefer to use this since I already build and deployed the program at the previous stage.

```bash
anchor test --skip-build --skip-deploy
```

After running the command above, you should be greeted with your newly minted NFT tx.

Clicking on the transaction link to the explorer, under the Token balances, you should have an change of `+1`.
![Screenshot of the mint nft tx!](/solana-nft-anchor/mint-nft-tx.png "mint nft transaction")

To view the minted NFT, open the `minted nft` link and you should have something similar to this.

![Screenshot of the minted nft on the solana explorer!](/solana-nft-anchor/minted-nft.png "details of minted nft on explorer")

Find the full article in [this repo](https://jimii.pages.dev/mint-solana-anchor/)