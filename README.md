# build-solana-img

참고자료: https://buildspace.so/p/build-solana-web3-app

- 이미지 링크를 올릴 수 있는 DApp
- like 기능 추가

필수

- node
- solana cli
- rust
- anchor(framework for solona program)
- phantom wallet

## client

- `npm i`
- `npm start`

---

date : 2022-10-14 20:27

# [build-solana](https://buildspace.so/p/build-solana-web3-app/lessons/building-gif-grid)

---

## CONNECT TO SOLANA WALLET

1. web repo
   https://github.com/buildspace/gif-portal-starter?utm_source=buildspace.so&utm_medium=buildspace_project
2. phantom wallet
3. wallet connect with react

## WRITE A SOLANA PROGRAM

### Get local Solana env running.

1. install rust
2. install solana cli
3. 테스트 프레임워크 `npm install -g mocha`
4. 앵커 프레임워크 `cargo install --git https://github.com/project-serum/anchor anchor-cli --locked`
5. 프로젝트에 solana web3 설치 `npm install @project-serum/anchor @solana/web3.js`
6. 앵커 프로젝트 생성
7. 솔라나 로컬 지갑 생성
   1. `solana-keygen new`
   2. `solana address`

### 🥳 Let's run our program

1.  Compile our program.
2.  Spin up `solana-test-validator` and deploy the program to our **local** Solana network w/ our wallet. This is kinda like deploying our local server w/ new code.
3.  Actually call functions on our deployed program. This is kinda like hitting a specific route on our server to test that it's working.

### 😍 Write your first Solana program.

####  A basic program

```rust

use anchor_lang::prelude::*;


// Basically, this is the "program id" and has info for Solana on how to run our program. Anchor has generated this one for us. We'll be changing it later.

declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");



// This is how we tell our program, "Hey — everything in this little module below is our program that we want to create handlers for that other people can call". You'll see how this comes into play. But, essentially this lets us actually call our Solana program from our frontend via a fetch request. We'll be seeing this `#[blah]` syntax a few places.

They're called macros — and they basically attach code to our module. It's sorta like "inheriting" a class.

#[program]

pub mod myepicproject {
	use super::*;


	// `pub mod` tells us that this is a Rust "module" which is an easy way to define a collection of functions and variables — kinda like a class if you know what that is. And we call this module `myepicproject`. Within here we write a function `start_stuff_off` which takes something called a `Context` and outputs a `Result <()>`. You can see this function doesn't do anything except call `Ok(())` which is just a `Result` type

	pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
		Ok(())
	}

}

#[derive(Accounts)]
pub struct Initialize {}
```

### 💎 Write a script to see it working locally

As long as you see a "transaction signature", you're good! This means you successfully called the startStuffOff function and this signature is basically your receipt.

Pretty epic. You've written a Solana program, deployed it to your local Solana node, and are now actually speaking to your deployed program on your local Solana network.

NICEEEEEEE. I know it may not seem like much but you now have a basic flow to do stuff.

Write code in lib.rs
Test specific functions using tests/myepicproject.js.
Get used to this cycle! It's the fastest way to iterate on your Solana programs :).

### ⚡️ Store basic data on our contract.

1. Cool so we just want to store a basic integer with the number of `total_gifs` people have submitted. So, every time someone adds a new gif we'd just do `total_gifs += 1`.
2. Remember earlier I said Solana programs are **stateless**. They **don't** hold data permanently. This is very different from the world of Ethereum smart contracts — which hold state right on the contract.
3. But, Solana programs can interact w/ "accounts".
4. Again, accounts are basically files that programs can read and write to. The word "accounts" is confusing and super shitty.
   1. For example, when you create a wallet on Solana —
   2. you create an "account".
   3. But, your program can also make an "account" that it can write data to.
   4. Programs themselves are considered "accounts".
   5. **Everything is an account lol**.
   6. Just remember an account isn't just like your actual wallet
   7. **it's a general way for programs to pass data between each other**.

### 🤠 Initializing an account

Basically, it tells our program what kind of account it can make and what to hold inside of it. So, here, `BaseAccount` holds one thing and it's an integer named `total_gifs`.

Then, here we actually specify how to initialize it and what to hold in our `StartStuffOff` context.

1.  `init` will tell Solana to create a new account owned by our current program.
2.  `payer = user` tells our program who's paying for the account to be created. In this case, it's the `user` calling the function.
3.  We then say `space = 9000` which will allocate 9000 bytes of space for our account. You can change this # if you wanted, but, 9000 bytes is enough for the program we'll be building here!

Why are we paying for an account? Well — storing data isn't free! How Solana works is users will pay "rent" on their accounts. You can read more on it [here](https://docs.solana.com/developing/programming-model/accounts#rent?utm_source=buildspace.so&utm_medium=buildspace_project) and how rent is calculated. Pretty wild, right? If you don't pay rent, validators will clear the account!

[Here's](https://docs.solana.com/storage_rent_economics?utm_source=buildspace.so&utm_medium=buildspace_project) another article from the docs on rent I liked as well!

> "With this approach, accounts with two-years worth of rent deposits secured are exempt from network rent charges. By maintaining this minimum-balance, the broader network benefits from reduced liquidity and the account holder can rest assured that their `Account::data` will be retained for continual access/usage."

We then have `pub user: Signer<'info>` which is data passed into the program that proves to the program that the user calling this program actually owns their wallet account.

Finally, we have `pub system_program: Program` which is actually pretty freaking cool. It's basically a reference to the [SystemProgram](https://docs.solana.com/developing/runtime-facilities/programs#system-program?utm_source=buildspace.so&utm_medium=buildspace_project). The SystemProgram is the program that basically runs Solana. It is responsible for a lot of stuff, but one of the main things it does is create accounts on Solana. The SystemProgram is a program the creators of Solana deployed that other programs like ours talk to haha — it has an id of `11111111111111111111111111111111`.

Lastly, we do this thing in our function where we just grab `base_account` from the `StartStuffOff` context by doing `Context<StartStuffOff>`.

### 🍿 Store structs on our program.

Let's now set it up where we can store an array of structs with more data we care about like: **a link to the gif and the public address of the person who submitted it.**

Then, we'd be able to retrieve this data on our client!

## 🚀 Deploy program to the devnet.

### Set up your environment for devnet

It's actually pretty tricky to deploy to devnet. Stay with me here and make sure not to miss any steps :).

First, switch to devnet:

```bash
solana config set --url devnet
```

Once you do that, run:

```bash
solana config get
```

From here, we'll need to airdrop ourselves some SOL on the devnet. It's actually pretty easy, we just run twice:

```bash
solana airdrop 2
```

Now, run:

```bash
solana balance
```

### ✨ Changing up some variables

In `Anchor.toml`, change `[programs.localnet]` to `[programs.devnet]`.
Then, change `cluster = "localnet"` to `cluster = "devnet"`.

Now, run:

```bash
anchor build
```

This will create a new build for us with a program id. We can access it by running:

```bash
solana address -k target/deploy/myepicproject-keypair.json
```

This will output your program id, **copy it**. We'll talk about it more in a second.
Now, go to `lib.rs`. You'll see this id at the top.

```bash
declare_id!("Fg6PaFpoGXkYsidMpWTK6W2BeZ7FEfcYkg476zPFsLnS");
```

So...what is this? It confused me a lot when I was first researching lol.

1. Basically, it's an id initially generated by `anchor init` that specifies our program's id.
2. That's important because the id of the program specifies how to load and execute the program and contains info on how the Solana runtime should execute the program.
3. The program id also helps the Solana runtime see all the accounts created by the program itself — so, remember how our program creates "accounts" that hold data related to the program?
4. Well, with this ID Solana can quickly see all the accounts generated by the program and easily reference them.

So, **we need to change this program id** in `declare_id!` to the one output by `solana address -k target/deploy/myepicproject-keypair.json`. Why? Well, the one `anchor init` gave us was just a placeholder. Now we'll have a program id we can actually deploy with!

### 🚀 Deploy to devnet!

And finally, you're good to deploy :)! Go ahead and run:

```bash
anchor deploy
```

### **YO — YOU JUST DEPLOYED TO THE ACTUAL SOLANA BLOCKCHAIN. NICE.**

This of course is not "Mainnet", but the "Devnet" is run by actual miners and is legit.

**There aren't that many "Solana developers". So this point you're probably in the top 10% of Solana developers lol. Congrats!**

_Note: From this point on, please don't make changes to lib.rs until I say so. Basically, whenever you change your program you need to redeploy and follow the steps above all over again. I always easily miss steps and get weird bugs lol. Let's focus on the web app now, and after I'll show you a good workflow for changing your program + redeploying after!_

### 🚨 Progress Report

_Please do this else Farza will be sad :(_

You deployed a Solana program!!! Hell yes -- this is huge!!

We've seen that the best builders have made it a habit to "build in public". All this means is sharing a few learnings about the milestone they've just hit!

This is a good time to tweet out that you're learning about Solana and just deployed your first program to the Solana Devnet. Inspire others to join web3!

Be sure to include your Solana Explorer link and attach a screenshot of your deployed program as well perhaps. Or, add a screenshot of it on Solana Explorer!! Tag `@_buildspace` if you're feeling fancy ;).

## ☎️ Call deployed program from web app.

### ✅ Running a test on devnet

I actually like to run an `anchor test` at this point before I start integrating my web app. This could potentially save us from some random, annoying errors.

What's actually interesting here is if you go to the [Solana Explorer](https://explorer.solana.com/?cluster=devnet&utm_source=buildspace.so&utm_medium=buildspace_project) and paste in your program id (just like you did before) you'll see some new transactions there. These were caused by the test you just ran! Feel free to check them out.

I should mention something super important here. **When you ran `anchor test` just now, it'll actually re-deploy the program and then run all the functions on the script.**

You may be asking yourself, "Why did it re-deploy? Why isn't it just talking to the program already deployed? Also, if we re-deployed wouldn't it have been deployed to a completely different program id?".

- So — Solana programs are [upgradeable](https://docs.solana.com/cli/deploy-a-program#redeploy-a-program?utm_source=buildspace.so&utm_medium=buildspace_project).
- That means when we re-deploy we're updating the same program id to point to the latest version of the program we deployed.
- And, what's cool here is the _accounts_ that the programs talk to will stick around — remember, these accounts keep data related to the program.

**That means we can upgrade programs while keeping the data piece separate**. Pretty cool, right :)?

_Note: this is **very very** different from Ethereum where you can never change a smart contract once it's deployed!_

### 🤟 Hooking up our IDL file to the web app

So, we now have a deployed Solana program. Let's connect it up to our web app :).

1. The first thing we need is the `idl` file that was magically output by `anchor build` earlier without you knowing.
2. You should see it in `target/idl/myepicproject.json`.
3. The `idl` file is actually just a JSON file that has some info about our Solana program like the names of our functions and the parameters they accept.
4. This helps our web app actually know how to interact w/ our deployed program.
5. You'll also see near the bottom it has our program id!
6. This is how our web app will know what program to actually connect to.
7. There are _millions_ of programs deployed on Solana and this address is how our web app can get quick access to our program specifically.

How do we give the idl to our web app?

- We can either copy the idl file and open it on the web app
- or we can actually upload the idl to solana directly and fetch it from our web app later!

```
anchor idl init  -f target/idl/myepicproject.json `solana address -k target/deploy/myepicproject-keypair.json`
```

Basically we are telling anchor to upload our idl for our program address, nice!!

> Please note that everytime you redeploy your solana program, you need to tell solana how your program api looks like, we can do so with `anchor idl upgrade` instead of `anchor idl init`

Head over to your web app.

### 🌐 Change the network Phantom connects to

### 👻 Fund Phantom wallet

- We also need to fund our Phantom wallet w/ some fake SOL.
- **Reading** **data** on accounts on Solana is free.
- But doing things like creating accounts and adding data to accounts costs SOL.

You'll need the public address associated w/ your Phantom wallet which you can grab at the top by clicking your address:

```bash
solana airdrop 2 INSERT_YOUR_PHANTOM_PUBLIC_ADDRESS_HERE  --url devnet
```

### 🍔 Setup a Solana `provider` on our web app

In your web app, we'll need to install two packages. You may remember installing these for your Anchor project, we'll also be using them in our web app :).

```bash
npm install @project-serum/anchor @solana/web3.js
```

Before we can interact with the packages that we installed earlier, we need to import them into our web app! Add the following lines of code at the top of App.js:

```javascript
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";
import { Program, AnchorProvider, web3 } from "@project-serum/anchor";
```

**You can't communicate with Solana at all unless you have a connected wallet. We can't even retrieve data from Solana unless we have a connected wallet!**

This is a big reason Phantom is helpful. It gives our users a simple, secure way to connect their wallet to our site so we can create a `provider` that lets us talk to programs on Solana :).

`SystemProgram` is a reference to the [core program](https://docs.solana.com/developing/runtime-facilities/programs#system-program?utm_source=buildspace.so&utm_medium=buildspace_project) that runs Solana we already talked about. `Keypair.generate()` gives us some parameters we need to create the `BaseAccount` account that will hold the GIF data for our program.

Then, we reuse our programID to tell the Solana Runtime which program we are trying to talk to, finally we make sure we connect to devnet by doing `clusterApiUrl('devnet')`.

This `preflightCommitment: "processed"` thing is interesting. You can read on it a little [here](https://solana-labs.github.io/solana-web3.js/modules.html#Commitment?utm_source=buildspace.so&utm_medium=buildspace_project). Basically, we can actually choose _when_ to receive a confirmation for when our transaction has succeeded. Because the blockchain is fully decentralized, we can choose how long we want to wait for a transaction. Do we want to wait for just one node to acknowledge our transaction? Do we want to wait for the whole Solana chain to acknowledge our transaction?

In this case, we simply wait for our transaction to be confirmed by the _node we're connected to_. This is generally okay — but if you wanna be super super sure you may use something like `"finalized"` instead. For now, let's roll with `"processed"`.

### 🏈 Retrieve GIFs from our program's account

It's actually super simple now to call our program now that we have everything set up. It's a simple `fetch` to get the account — similar to how you'd call an API. Remember this chunk of code?

### 🤬 Wtf an error!?

When you refresh your page, you'll get an error that looks something like this:

But, what this error actually means is our program's `BaseAccount` does not exist.

Which makes sense, we haven't yet initialized the account via `startStuffOff`!! Our account doesn't just magically get created. Let's do that.

### 🔥 Call `startStuffOff` to initialize program

Let's build a simple function to call `startStuffOff`. You are going to want to add this under your `getProvider` function!

This looks exactly like we had it working in the test script!

### 🥳 Let's test!

Let's go ahead and test! If you refresh the page and have your wallet connected, you'll see "Do One-Time Initialization For GIF Program Account". When you click this, you'll see Phantom prompt you to pay for the transaction w/ some SOL!!

If everything went well, then you'll see this in the console:

So, here we created an account _and then_ retrieved the account itself!! And, `gifList` is empty since we haven't added any GIFs to this account yet!!! **NICEEEEE.**

**So, now you'll notice that every time we refresh the page — it asks us to create an account again. We'll fix this later but why does this happen? I made a little video about it below**

`let baseAccount = Keypair.generate();`
매번 키페어를 생성하기때문

## 🎨 Submitting GIFs to Solana program.

Okay — we are finally at the point where we can save some GIFs. It's so easy to do. We're just going to change up our `sendGif` function a little bit so we now call `addGif` and then call `getGifList` so that our web app refreshes to show our latest submitted GIF!

### 🙈 Solve the issue of the account not persisting

What's happening here is we're generating a new account for our program to talk to **every time.** So, the fix here is we just need to have one keypair that all our users share.

That's it. Now, we have a permanent keypair! If you go and refresh, you'll see that after you initialize the account — it stays even after refresh :)!!! Feel free to submit a few GIFs from here.

You can also run `createKeyPair.js` as many times as you want and this will let you create a new `BaseAccount`. Though, this also means that the new account will be completely empty and have no data. It's important to understand you **aren't deleting accounts if you run** `createKeyPair.js` again. You're simply create a new account for your program to point to.

## 🧹 Finishing touches to your web app and program.

### ✈️ Making updates to your program

Let's say you want to add some new functionality to your Solana program.

- I said earlier not to touch `lib.rs` because working locally and redeploying gets weird.
- So, basically w/ Solana I almost _never_ work on `localnet` usually.
- **It's just really annoying to switch between `localnet` and `devnet` constantly.**
- Instead, I update my program and then, to test it, just run my script via `anchor test` on `tests/myepicproject.js` to make sure stuff is working and Anchor will actually run the test on `devnet` directly which is pretty cool.
- Then, when I'm ready to test the updates to my program on my web app — I just do an `anchor deploy`.
- From there you need to make sure you grab the updated IDL file for your web app.

**Whenever you re-deploy, you need to update the IDL file on your web app**

Just like before, you'd need to upload your idl to solana but instead of calling `init` this time you will call `upgrade`

```bash
anchor idl upgade -f target/idl/myepicproject.json `solana address -k target/deploy/myepicproject-keypair.json
```

## 🏠 Show off a user's public address on the web app

We're currently not using the user's public address for anything right now in:

```rust
pub struct ItemStruct {
    pub gif_link: String,
    pub user_address: Pubkey,
}
```

It might be cool to show off the user's public address under the GIF they submitted!! Doing something like `item.userAddress.toString()` on your web app should work. I'll let you figure out how to implement it if you want to.

### 🙉 Let users upvote GIFs

It'd be cool if each GIF started at 0 votes, and people could come in an "upvote" their favorite GIFs on your web app.

1. Not going to tell you how to make it though ;).
2. Figure it out if you want to!
3. Hint: you'll need to make an `update_item` function in your Solana program.
4. In there, you'd need to figure out how to go inside `gif_list`
5. , find the GIF that's being upvoted, and then do something like `votes += 1` on it.

### 💰 Send a Solana "tip" to submitters of the best GIFs

One thing we didn't cover at all here is how to send money to other users!

- It would be super cool if you loved a certain GIF submitted by another user SO MUCH that you could to send that user a little tip.
- Maybe like 50 cents or like a dollar worth of SOL.
- Maybe you'd click "tip", input how much SOL you wanted to tip, and click send to send it right to that user's wallet!

- **Solana's super low gas fee means sending small amounts of money like this actually makes sense.**
- If you do this, you could even make a version of Patreon or BuyMeACoffee on Solana. Not that crazy. You have all the basic skills now.

## 🎉 Finalize and celebrate.

### 😍 Hello, Solana Master

Super exciting that you made it to the end. Pretty big deal!! Solana is **super early** and very powerful and you've now gotten your hands dirty w/ the core tech. Hell yes. You have all the core skills to now go and build whatever you want.

Thank you for contributing to the future of web3 by learning this stuff. The fact that you know how this works and how to code it up is a superpower. Use your power wisely ;).

### 🥞 Careers in Web3

Tons of people have also gotten full-time jobs at top web3 companies via buildspace. I'm constantly seeing people nail their interviews after they do a few buildspace projects.

![Untitled](https://camo.githubusercontent.com/c467ff4530007751ca0ab96c2122734535946d62ac5111716c7c82a73a3f6a50/68747470733a2f2f692e696d6775722e636f6d2f5172466a6c4e482e706e67)

**People seem to think web3 just needs people who can code smart contracts or write code that interfaces w/ the blockchain. Not true.**

There is so much work to do and most of the work doesn't even have to do w/ smart contracts lol. **Being an engineer in web3 just means you take your web2 skills and apply them to web3.**

I want to quickly go over wtf it means to "work in web3" as an engineer. *Do you need to be a pro at Solana? Do you need to know how every little thing about the blockchain works?*

For example, let's say you're a great frontend engineer. If you finished this project, **you have almost everything you need to be a great frontend engineer at a web3 company**. For example, the company may say "Hey — pls go and build our connect to wallet feature" — and you'd already have a solid idea on how to do that :).

I just wanna inspire you to work in web3 lol. This shit is awesome. And it'd be cool if you gave it a shot ;).

Be sure you click "Work in Web3" on the left and fill out your profile if you haven't done so already!!! **We're partnered w/ some of the best web3 companies in the world (ex. Uniswap, OpenSea, Chainlink, Edge and Node, and more) and they want to hire devs from the buildspace network :).** You've already picked up a skill that is actually extremely valuable and companies are paying top dollar for awesome web3 engineers.

### 🌈 Before you head out

Go to **#showcase** in Discord and drop us a link to your final product that we can mess around with :).

Also, you should totally tweet out your final project and show the world your epic creation! What you did wasn't easy by any means. Also, when you get your NFT be sure to save the file to your computer and attach it to your tweet directly. Looks better that way ;).

**We made sure to make the NFT look super cool so you can impress the Twitter-verse!!**

It may sounds weird, but, you being excited about your NFT and Solana on Twitter will likely inspire tons of other devs get into this stuff as well!

Be sure to tag @\_buildspace :). We'll RT you!

Give us that dopamine hit pls lol.

Lastly, what would also be awesome is if you told us in #feedback how you liked this project and the structure of the project. What did you love most about buildspace? What sucked? What would like us to change for future projects? Your feedback would be awesome!

See yah around!!!

# deploy

toml 수정

1. 프로그램환경
2. 클러스터
3. 프로그램id

lib 프로그램 id 수정

```bash
solana address -k target/deploy/myepicproject-keypair.json

lib.rs
declare_id!(address)
```

웹앱용 idl

```
anchor idl init  -f target/idl/myepicproject.json `solana address -k target/deploy/myepicproject-keypair.json`
```

# re deploy

```
anchor idl upgrade -f target/idl/myepicproject.json `solana address -k target/deploy/myepicproject-keypair.json`
//
Idl buffer created: 3y8tnRgSAKZNgKG6aLwRUnjSnY4k3KipZiT7a6pEx4FE
```
