+++
title = "Build a guessing game using Orao VRF"
date = 2024-05-20
category = "Prog"
draft=false

[taxonomies]
tags = ["Rust", "solana", "CLI", "clap"]
+++

In this tutorial we will learn about Verifiable Random Functions by building a guessing game on the Solana blockchain using Orao Network's Random Verifiable Function service.

We will implement the Solana program onchain, which generates a random number between 0 and 10. We will then build a simple CLI interface to interact with the program. The CLI will prompt the user for a number. After a guess is entered, the program will indicate whether the guess is too low or too high. If the guess is correct, the game will print a congratulatory message and exit.

<!-- more -->

**Note:** This tutorial is only meant to show you how you can generate and use a VRF inside your onchain program.

Use this GitHub to check against your code - [Calyptus-Learn/orao-vrf](https://github.com/Calyptus-Learn/orao-vrf).

## Verifiable Random Function

Imagine that you a building a game on the Solana blockchain. Suppose it is similar to what we are going to build, `a guessing game`, but with a twist that the first person to guess the correct number wins a prize. How would you ensure fairness and transparency when generating the random numbers for the game. Due to the deterministic nature of blockchain, it would be very difficult to do this within your program(smart contract) and relying on a trusted centralized provider, but what if it get compromised?

In comes, Verifiable Random functions.

A VRF is a function f_s(x) = (y,p) with the property that the output y looks random, but is deterministically computed from the input x and secret key s. Furthermore, the function returns a proof p that anyone can check to verify that y is the correct output.

TLDR; VRFs can be thought of as Random Number generators that can be proven.

## How Orao works

Orao works by accepting a clients request (either through the Rust/Typescript SDK or a CPI call from another program), to generate randomness. Once generated, the randomness is sent to each of the authoritative nodes and fulfilled by the on-chain Orao VRF program.

Below is a diagram to demonstrate this process.

<div style="display: flex; justify-content: center;">
<video width="500" height="full" controls>
  <source src="/orao-vrf/orao-vrf-vid.mp4" type="video/mp4">
</video>
</div>

## Environmental Setup

This tutorial assumes you have already set up your Solana environment with the `solana-cli` and `anchor-cli`.

If not,

-   [Install Solana](https://docs.solanalabs.com/cli/install)

```bash
sh -c "$(curl -sSfL https://release.solana.com/v1.17.28/install)"
```

-   [Install anchor using avm](https://www.anchor-lang.com/docs/installation)

```bash
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force

sudo apt-get update && sudo apt-get upgrade && sudo apt-get install -y pkg-config build-essential libudev-dev libssl-dev # for Linux systems only

avm install latest

avm use latest
```

Confirm proper installation by running

```bash
anchor --version && solana --version

# EXPECTED OUTPUT
# anchor-cli 0.29.0
# solana-cli 1.18.8 (src:9a7dd9ca; feat:3469865029, client:SolanaLabs)
```

If you face any problems, please raise an issues in the official [Solana Stack Exchange](https://solana.stackexchange.com/) and some kind stranger will come to your aide.

## Initializing anchor project

Let's get started by creating a new anchor project

```bash
anchor init guessing-game
```

Install the orao VRF crate with the `cpi` features

```bash
cargo add orao-solana-vrf --features="cpi"
```

Once initialized, you should have something similar to this,

```rust
use anchor_lang::prelude::*;

declare_id!("HMDRWmYvL2A9xVKZG8iA1ozxi4gMKiHQz9mFkURKrG4");

#[program]
pub mod orao_vrf {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

Don't worry if you encounter the `struct takes 0 lifetime arguments` error pictured below, it'll go away as soon as we begin adding our accounts.

![Lifetime error upon initializing a new anchor project](/orao-vrf/no-accounts-error.png)

The orao program is made up of various instructions, of interest to us is the [`request`](https://github.com/orao-network/solana-vrf/blob/f438eef602b3c716bef7edabec72a8bafa5d5338/rust/sdk/src/lib.rs#L126-L134) IX which we shall uss to request randomness.

But before that

## A quick primer on CPIs

The Solana runtime makes it possible to call instructions/methods on another program. This is made possible by a concept know as Cross Program Invocation or CPI in short, which allows us to make call across different programs constrained to calls of up to 4 programs deep. It allows the caller program to call an instruction to the callee program with signer an writer privilege being passed between the two of them.

Using the Anchor program, this process is simplified for us by having the [`CpiContext`](https://docs.rs/anchor-lang/latest/anchor_lang/context/struct.CpiContext.html). CpiContext encapsulates all accounts that we need to interact with when making our CPI call.

To note is that the `invoke` and `invoke_signed` methods are used to escalate the signer privileges to the program being called.

## Building the Guessing Game

With most of the theory out of the way, we are ready to start building.

To request randomness from the orao program from our VRF program, we will need to pass in these accounts

-   `payer (mutable, signer)` - pays for the randomness request.
-   `network_state (mutable)` - account that contains details about number of requests received and the [network configuration](https://github.com/orao-network/solana-vrf/blob/f438eef602b3c716bef7edabec72a8bafa5d5338/rust/sdk/src/state.rs#L14-L22) which is an account that stores details about the `fees`, `treasury address`, ... e.t.c

-   `treasury` - treasury account, where fees for using the VRF will be paid.
-   `request` - an uninitialized account that contains the seed used to derive randomness, the actual randomness and a list of other responses.
-   `system_program` - the solana system program.

Let's get started by renaming our `Accounts` struct appropriately to `GuessingGame` and adding the required accounts.

We will use the `initialize` method to initialize our guessing game, i.e create the random account with a random number that we will have our user guess until he get's it correct.

With these changes, your program should start taking shape and you should have something that looks like this

```rs
use anchor_lang::prelude::*;

use orao_solana_vrf::{
    program::OraoVrf, state::NetworkState, CONFIG_ACCOUNT_SEED, RANDOMNESS_ACCOUNT_SEED,
};

declare_id!("HMDRWmYvL2A9xVKZG8iA1ozxi4gMKiHQz9mFkURKrG4"); // ! UPDATE ME

#[program]
pub mod guessing_game {
    use super::*;

    pub fn initialize(ctx: Context<GuessingGame>, force_seed: [u8; 32]) -> Result<()> {
        Ok(())
    }
}

#[derive(Accounts)]
#[instruction(force_seed: [u8; 32])]
pub struct GuessingGame<'info> {
    #[account(mut)]
    pub payer: Signer<'info>,

    #[account(
        mut,
        seeds = [CONFIG_ACCOUNT_SEED.as_ref()],
        bump,
        seeds::program = orao_solana_vrf::ID
    )]
    pub network_state: Account<'info, NetworkState>,

    /// CHECK:
    #[account(mut)]
    pub treasury: AccountInfo<'info>,

    /// CHECK:
    #[account(
        mut,
        seeds = [RANDOMNESS_ACCOUNT_SEED.as_ref(), &force_seed],
        bump,
        seeds::program = orao_solana_vrf::ID
    )]
    pub random: AccountInfo<'info>,

    pub orao_vrf: Program<'info, OraoVrf>,
    pub system_program: Program<'info, System>,
}

```

To note, with the code snippet above, we added the `#[instruction(force_seed: [u8; 32])]` macro to seed a random number, from a random public key.

Next we implement the CPI method to make the call to the Orao VRF program's request IX

Finally lets implement the `initialize` functions for our contract.

Since the function will be a CPI to an external program, let's implement the CPI call to the orao VRF program inside an `impl` block to house the accounts context for the external CPI call to the Orao program,.
First import the required libraries and the Orao program Accounts struct, which we have access because of the `cpi` features flag we used when installing the crate. Make note of the `cpi::accounts::Request` path as importing the Request Accounts struct directly from the crate won't compile and will class with the existing macro expansions.

```rust
use orao_solana_vrf::{
    cpi::accounts::Request, program::OraoVrf, state::NetworkState, CONFIG_ACCOUNT_SEED,
    RANDOMNESS_ACCOUNT_SEED,
};
```

We can them implement the cpi context method that will call the VRF program within this impl block

```rust
impl<'info> GuessingGame<'info> {
    pub fn request_ctx(&self) -> CpiContext<'_, '_, '_, 'info, Request<'info>> {
        let cpi_program = self.orao_vrf.to_account_info();
        let cpi_accounts = Request {
            payer: self.payer.to_account_info(), // ! because he is paying for thetx
            network_state: self.network_state.to_account_info(),
            treasury: self.treasury.to_account_info(),
            request: self.random.to_account_info(),
            system_program: self.system_program.to_account_info(),
        };

        CpiContext::new(cpi_program, cpi_accounts)
    }
}
```

Let's call the request function from the initialize IX to generate a random number and assign it to our randomness account

```rust
    pub fn initialize(ctx: Context<GuessingGame>, force_seed: [u8; 32]) -> Result<()> {
        orao_solana_vrf::cpi::request(ctx.accounts.request_ctx(), force_seed)?;

        Ok(())
    }
```

To call an external IX from our program, we need to add the Account context for that program and pass in any required arguments, which significantly simplifies our work instead of raw dogging our functions by manually passing in the required accounts, instructions and the required data with the `invoke` and `invoke_signed` methods.

Your full code should resemble

```rust
use anchor_lang::prelude::*;

use orao_solana_vrf::{
    cpi::accounts::Request, program::OraoVrf, state::NetworkState, CONFIG_ACCOUNT_SEED,
    RANDOMNESS_ACCOUNT_SEED,
};

declare_id!("HMDRWmYvL2A9xVKZG8iA1ozxi4gMKiHQz9mFkURKrG4"); // ! UPDATE ME

#[program]
pub mod guessing_game {
    use super::*;

    pub fn initialize(ctx: Context<GuessingGame>, force_seed: [u8; 32]) -> Result<()> {
        orao_solana_vrf::cpi::request(ctx.accounts.request_ctx(), force_seed)?;

        Ok(())
    }
}

#[derive(Accounts)]
#[instruction(force_seed: [u8; 32])]
pub struct GuessingGame<'info> {
    #[account(mut)]
    pub payer: Signer<'info>,

    #[account(
        mut,
        seeds = [CONFIG_ACCOUNT_SEED.as_ref()],
        bump,
        seeds::program = orao_solana_vrf::ID
    )]
    pub network_state: Account<'info, NetworkState>,

    /// CHECK:
    #[account(mut)]
    pub treasury: AccountInfo<'info>,

    /// CHECK:
    #[account(
        mut,
        seeds = [RANDOMNESS_ACCOUNT_SEED.as_ref(), &force_seed],
        bump,
        seeds::program = orao_solana_vrf::ID
    )]
    pub random: AccountInfo<'info>,

    pub orao_vrf: Program<'info, OraoVrf>,
    pub system_program: Program<'info, System>,
}

impl<'info> GuessingGame<'info> {
    pub fn request_ctx(&self) -> CpiContext<'_, '_, '_, 'info, Request<'info>> {
        let cpi_program = self.orao_vrf.to_account_info();
        let cpi_accounts = Request {
            payer: self.payer.to_account_info(), // ! because he is paying for thetx
            network_state: self.network_state.to_account_info(),
            treasury: self.treasury.to_account_info(),
            request: self.random.to_account_info(),
            system_program: self.system_program.to_account_info(),
        };

        CpiContext::new(cpi_program, cpi_accounts)
    }
}
```

## Testing

We will begin by first writing our typescript tests to check whether everything is working as expected.

To lessen our workload we will also be installing the Orao TS Client. The TS client library from Orao that provides methods such as fetching the network config address and the orao programId instead of having to keep track of these ourselves.

```bash
yarn add @orao-network/solana-vrf
```

Let's take a look at our `guessing-game.ts` file inside the `tests` directory. Nothing special happening here. We have one test which is for the `initialize` IX that we removed earlier. (let's get rid of this test).

The first line below, sets up our solana test environment, instructing anchor to use our local command-line wallet that we generated when we installed the solana CLI suite and run `solana-keygen new` and a keypair file `~/.config/solana/id.json` was generated. By default the connection string it will use will be the public API endpoints provided for free. If you face an error such as `failed to get recent blockhash: TypeError: fetch failed`, you should change this and use a private RPC provider to make the error go away.

The second line is the typescript type interface to interact with our program via the generated IDL.

```ts
anchor.setProvider(anchor.AnchorProvider.env());

const program = anchor.workspace.GuessingGame as Program<GuessingGame>;
```

Let write the test to call the `guess` IX. We will be interacting with the provider details a lot so let's move it to it's own variable

```ts
const provider = anchor.AnchorProvider.env();
```

We need to generate a random public key for our `force_seed`. We also need to convert it to a bit array using the `toBuffer()` methods because our IX expects a slice of `u8`'s.

```ts
let force_seed = anchor.web3.Keypair.generate().publicKey.toBuffer();
```

Next we instantiate a new class using the Orao client, generate our random account address, and add the VRF Treasury address

```ts
const vrf = new Orao(provider);
const random = randomnessAccountAddress(force_seed);
const treasury = new anchor.web3.PublicKey(
	"9ZTHWWZDpB36UFe1vszf2KEpt83vwi27jDqtHQ7NSXyR"
);
```

Tying it all together, our test will look something like this

```ts
describe("orao-vrf", () => {
	// Configure the client to use the local cluster.
	const provider = anchor.AnchorProvider.env();
	anchor.setProvider(provider);

	const program = anchor.workspace.GuessingGame as Program<GuessingGame>;

	let force_seed = anchor.web3.Keypair.generate().publicKey.toBuffer();

	const vrf = new Orao(provider);
	const random = randomnessAccountAddress(force_seed);

	it("Got a random number from the Orao VRF program!", async () => {
		console.log("is this working");
		const tx = await program.methods
			.init([...force_seed])
			.accounts({
				payer: provider.wallet.publicKey,
				treasury,
				oraoVrf: vrf.programId,
				random,
				networkState: networkStateAccountAddress(),
				systemProgram: anchor.web3.SystemProgram.programId,
			})
			.rpc({ skipPreflight: true });

		await vrf.waitFulfilled(force_seed);
	});
});
```

When we run our tests, we make a request to the Orao VRF. But we need to fulfil that request and that's where we use the line below

```ts
await vrf.waitFulfilled(force_seed);
```

We get our random number using the line above, which returns a byte array of random numbers.

Let's build and deploy our program on devnet. Update your `Anchor.toml` provider to use the devnet cluster

```toml
[provider]
cluster = "devnet"
wallet = "~/.config/solana/id.json"
```

Now, we are ready to build and deploy our program by running

```bash
anchor build && anchor deploy
```

If you happen to encounter the errors below

```bash
error: package `solana-program v1.18.10` cannot be built because it requires rustc 1.75.0 or newer, while the currently active rustc version is 1.68.0-dev
Either upgrade to rustc 1.75.0 or newer, or use
cargo update -p solana-program@1.18.10 --precise ver
where `ver` is the latest version of `solana-program` supporting rustc 1.68.0-dev
```

Fix it by downgrading the `tomo_edit` using this command

```bash
cargo update -p toml_edit@0.21.1 --precise 0.21.0
```

```bash
Error: Deploying program failed: RPC response error -32002: Transaction simulation failed: Error processing Instruction 0: account data too small for instruction [3 log messages]
There was a problem deploying: Output { status: ExitStatus(unix_wait_status(256)), stdout: "", stderr: "" }.
```

You will need to extend your deployed program account using. Replace `INSERT_PROGRAM_ADDRESS_HERE` with your program address which you can get using `anchor keys list`

```bash
solana program extend INSERT_PROGRAM_ADDRESS_HERE 20000 -u d -k ~/.config/solana/id.json
```

The error above occurs on some of the `.17` releases because deploying a program with 2X the size was removed. On newer releases you can use the `--auto-extend-program` flag when deploying the program. Read more in the [Merged PR](https://github.com/anza-xyz/agave/pull/791)

Finally you should now have generated your Randomness account, when calling the tests using.

```bash
anchor test --skip-build --skip-deploy # tells anchor to not build and deploy our program since we did it in the previous step
```

Let's now work on the `guess` IX. The workflow will look something like this

1. read randomness account to see the random numbers generated
2. read the user's input from function arguments
3. print a program log of whether the use guessed the correct number, one too high or one too low from the first element in the random slice

Taking on 1, first, reading an account using anchor is easy. Since anchor handles generating all the borsh boilerplate for us, deserializing account data is trivial and easy to do with using `Account` types and luckily for us Orao has already typed this account for use. Let's use it to get the data inside it.

Let's also create a new file `misc.rs` where we will put this logic `touch misc.rs`. Import this module into our lib.rs file, using the statement `pub mod misc`

The method below will return the account data or break with an error should the account be empty or not of type `Randomness`

```rust
use anchor_lang::{
    solana_program::{account_info::AccountInfo, program_error::ProgramError},
    AccountDeserialize,
};
use orao_solana_vrf::state::Randomness;

pub fn get_account_data(account_info: &AccountInfo) -> Result<Randomness, ProgramError> {
    if account_info.data_is_empty() {
        return Err(ProgramError::UninitializedAccount);
    }

    let account = Randomness::try_deserialize(&mut &account_info.data.borrow()[..])?;

    if false {
        Err(ProgramError::UninitializedAccount)
    } else {
        Ok(account)
    }
}
```

To handle task 2, we will need to adjust our original Accounts context to include the `user_guess` arguments as without it the generated random account address will not match what we want.

**Note:** Make sure you update the `initialize` instruction to include this parameter and pass it in with the client function we created.

```rust

....

#[derive(Accounts)]
#[instruction(user_guess: u64, force_seed: [u8; 32])] // new

...

```

Let's write the logic for the guess function. We will use the first element of the slice as our random number because the VRF program generates a slice.

```rust
    pub fn guess(ctx: Context<GuessingGame>, user_guess: u64, _force_seed: [u8; 32]) -> Result<()> {
        let account_data = get_account_data(&ctx.accounts.random)?;

        // use the first 8 bytes from the byte slice
        let byte_array: [u8; 8] = account_data.randomness[0..size_of::<u64>()]
            .try_into()
            .unwrap();
        let secret_number = u64::from_le_bytes(byte_array);
        let secret_number = secret_number % 10;

        match user_guess.cmp(&secret_number) {
            Ordering::Less => msg!("Too small!"),
            Ordering::Greater => msg!("Too big!"),
            Ordering::Equal => msg!("You win! {:?}", secret_number),
        }

        Ok(())
    }
```

After deserializing the account data, we fetch the first 8 bytes from the randomness slice and convert that to a u64 number which we compare against the user's guess. We also make sure that the number generated ranges from 0 - 10;

### Guess IX client side

Nothing much will change with out client code this time, the arguments we pass to our IX methods and the txHash, which we will use to check out our guess on the explorer

```ts
it("guesses a random!", async () => {
	let txHash = await program.methods
		.guess(new BN(10), [...force_seed])
		.accounts({
			payer: provider.wallet.publicKey,
			treasury,
			oraoVrf: vrf.programId,
			random,
			networkState: networkStateAccountAddress(),
			systemProgram: anchor.web3.SystemProgram.programId,
		})
		.rpc({ skipPreflight: true });

	console.log(
		`tx: https://explorer.solana.com/tx/${txHash}?cluster=devnet\n`
	);
});
```

When we deploy our program and run our tests, we navigate to the explorer to check if our guess was correct. In my case, the number was larger than the random number generated, [guess transaction](https://explorer.solana.com/tx/5yf8CrJEQUMBuZgTCdU2vQVK3vWVfTghynGTssUVz735otn5GvCqbciPX9M7VrKz3gXAYLSJwZhuxwqY9kppbV8h?cluster=devnet)

![using exa to list the files in the working directory](/orao-vrf/guess-tx-details.png)

### References

Check out examples here - https://github.com/orao-network/solana-vrf/tree/master/rust/examples
