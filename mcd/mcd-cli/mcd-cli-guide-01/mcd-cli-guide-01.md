# How to use mcd-cli to interact with Kovan deployment of MCD

**This guide works under the [0.2.10 Release](https://changelog.makerdao.com/releases/0.2.10/index.html) of the system.** 

This guide will show how to use the [mcd-cli](https://github.com/makerdao/mcd-cli) to interact with the Kovan deployment of the MCD smart contracts. The guide will showcase how to go through the following stages of the lifecycle of a collateralized debt position (CDP):

 - Opening CDP
 - Depositing collateral tokens (REP, OMG, ZRX, BAT, DGD, GNT)
 - Drawing Dai
 - Paying back Dai
 - Unlocking collateral
 - Closing CDP

The command-line interface mcd-cli will enable you to easily interact with the Multi-Collateral Dai contracts. In the CLI, you can lock assets such as ETH and many other collateral tokens we have added (REP, OMG, ZRX, BAT, DGD, GNT)), draw Dai against them, check your CDP position, and much more.  

## Installing mcd-cli and seth
**The following link provides you with the necessary instructions to get started with the MCD CLI:**  [https://github.com/makerdao/mcd-cli](https://github.com/makerdao/mcd-cli)  

 - First install  [dapp tools](https://dapp.tools/):

`$ curl https://dapp.tools/install | sh`

 - Then install the  `mcd`  package:

`$ dapp pkg install mcd`

### Setting up variables in seth
Configuring Seth can be done with environment variables or command line options. Environment variables can be generally used in two ways: you can save them in a configuration file named `.sethrc` in specific locations, like your home folder, or just set them only for the current terminal session. In this guide we will use environment variables with the latter approach for simplicity’s sake, however for ease-of-use in the future, we strongly encourage to save the variables in your project folder. Follow [this example](https://github.com/dapphub/dapptools/tree/master/src/seth#example-sethrc-file) to do so.

Seth can connect to the Kovan Ethereum testnet through a default remote node provided by Infura, by specifying the `SETH_CHAIN` variable in a terminal or the `.sethrc` file:

`export SETH_CHAIN=kovan`

If you decide to create a new account, an easy method is using the "create new wallet" option in MEW: [https://www.myetherwallet.com/](https://www.myetherwallet.com/). It is also possible to use Parity or Geth to create a new account or you can use an existing keystore file for a Parity or Geth account. You are also going to need to save the password of your keystore file in a plain text file (Never use this keystore file for real ether - saving the password for your keystore file in plain text would be very unsafe for a real account! This also goes for the testnet account!).

Then you have to set up account variables:

    export ETH_KEYSTORE=<path to your keystore folder>
    export ETH_PASSWORD=<path and filename to the text file containing the password for your account e.g: /home/one1up/MakerDAO/415pass >
    export ETH_FROM=<your ethereum account address>
    export MCD_CHAIN=kovan

## Acquiring test tokens

To start using and interacting with the MCD contracts, you will need to get some Kovan ETH, REP, OMG, ZRX, BAT, DGD and MKR. Please note that MKR holders will eventually confirm the final collateral types through a proposed vote from the Maker Risk Team. Once you have the respective Kovan tokens, you can proceed to the guide below to go through the processes of a CDP lifecycle. This includes locking in some collateral, drawing Dai, paying back the Dai, and then unlocking the collateral.  

In the guide below, everything described above will be performed using the MCD-CLI.

### How to Get Kovan-ETH and K-Collateral Tokens

#### Getting Kovan ETH

There are many sources from which to get Kovan ETH, including:

-   **Standard Faucet**  **Method:**  [https://faucet.kovan.network/](https://faucet.kovan.network/)
    -   This method requires you to log in with your Github account. You then must paste your ETH address in the input box and request the funds.
-   **Gitter Method:**  [https://gitter.im/kovan-testnet/faucet](https://gitter.im/kovan-testnet/faucet)
    -   This method also requires you to log in with your GitHub or an existing Gitter account. To receive Kovan ETH through this method, join this Gitter Channel (which you also need to cover gas costs for use of the dApps):  [https://gitter.im/kovan-testnet/faucet](https://gitter.im/kovan-testnet/faucet). Once you join the Gitter Channel, post your ETH address from MetaMask to the main chat. The Kovan faucet will then populate your wallet with the test funds. This could take a couple minutes or a couple of hours, as it is done manually by the channel’s admin.

#### Getting Kovan Collateral Tokens

We have deployed a special faucet that allows you to withdraw testnet collateral tokens that essentially mimic the real tokens that exist on mainnet.  

**K-Collateral Token Faucet Address:** `0x94598157fcf0715c3bc9b4a35450cce82ac57b20`

**Note:**  You can call the  **gulp(address)**  function on it with  **seth**. The **address** parameter is the address of the REP to GNT collateral types we have added to this deployment.  

**Instructions:**  
In order to receive some REP tokens, you must run the following commands in the CLI:

**i. Setting the REP address to env variable:**

`$ export REP=0xc7aa227823789e363f29679f23f7e8f6d9904a9b`

**ii. Setting the Faucet address to env variable:**

`export FAUCET=0x94598157fcf0715c3bc9b4a35450cce82ac57b20`

**iii. Now, you can call the** **_gulp(address)_** **function:**

`$ seth send $FAUCET 'gulp(address)' $REP` 

**iv. Please verify your REP balance by running:**

`$ seth --from-wei $(seth --to-dec $(seth call $REP 'balanceOf(address)' $ETH_FROM)) eth`   
or   
`mcd --ilk=REP-A gem balance ext`

**An example of the output you should be viewing when running the above command:**

    50.000000000000000000

**That’s it! You now have some kovan REP tokens.**  

**Note:**  If you would like to receive some K-MKR tokens, you would need to replace the  **REP** token address with the  K-MKR  token address (0xaaf64bfcc32d0f15873a02163e7e500671a4ffcd) and follow the exact same process as above.
`export MKR=0xaaf64bfcc32d0f15873a02163e7e500671a4ffcd`

After you have successfully received the Kovan collateral tokens, you can continue on and explore the MCD-CLI.

## CDP Lifecycle Walkthrough

The following instructions will guide you through an example of a CDP’s lifecycle. We will be creating a loan with the REP token and will then pay it back. Make sure you have set up the env variables outlined above in **Setting up variables in seth** since we will be using those in the following.

**Once set up, you can begin to run through the CDP lifecycle using the commands noted below.**  

For this example, we are going to use the REP token as the first type of collateral in our CDP. Before proceeding, please check that you have already received some REP from the faucet. If you haven’t, please visit the  **‘Getting K Collateral tokens’** section above.  

**Instructions**  

**1. Add the REP token into the REP adapter. Here, you must change the below value of ’40’ to your own value.**  

**Run:**  
`$ mcd --ilk=REP-A gem join 40`

**Output Example:**  

    vat 40.000000000000000000 Unlocked collateral (REP)
    ink 0.000000000000000000 Locked collateral (REP)
    ext 0.000000000000000000 External account balance (REP))

----------

**2. Lock your REP collateral tokens and then draw 1 dai from VAT. Again, please don’t forget to change the below value of 40 to your own value.**  

**Run:**

`$ mcd --ilk=REP-A frob 40 1`

**Example Output:**  

    ilk  REP-A                                      Collateral type
    urn  0x16Fb96a5fa0427Af0C8F7cF1eB4870231c8154B6 Urn handler
    ink  40.000000000000000000                      Locked collateral (REP)
    art  1.000000000000000000                       Issued debt (Dai)
    tab  1.000000000000000000000000000              Outstanding debt (Dai)
    rap  0.000000000000000000000000000              Accumulated stability fee (Dai)
    -->  329.39                                     Collateralization ratio

    spot 8.234955555555555555555555555              REP price with safety mat (USD)
    rate 1.000000000000000000000000000              REP DAI exchange rate



----------

**3. Withdraw Dai and send it to your ETH personal account.**  
**Run:**    
`$ mcd dai exit 1`

**Example Output:**

    vat 0.000000000000000000000000000000000000000000000 Vat Dai balance
    ext 1.000000000000000000 ERC20 Dai balance

**Note:** When you want to pay back your debt and unlock your collateral, follow these steps again.  

----------
**4. Add your Dai back into the urn.**    

**Run:**   
`$ mcd dai join 1`  

**Example Output:**  

    vat 1.000000000000000000000000000000000000000000000 Vat Dai balance
    ext 0.000000000000000000 ERC20 Dai balance  

----------

**5. Remove your Dai debt and unlock your REP collateral from the internal system (vat).**  

**Run:**

`$ mcd --ilk=REP-A frob -- -40 -1`

**Example Output:**

    ilk  REP-A                                      Collateral type
    urn  0x16Fb96a5fa0427Af0C8F7cF1eB4870231c8154B6 Urn handler
    ink  0.000000000000000000                       Locked collateral (REP)
    art  0.000000000000000000                       Issued debt (Dai)
    tab  0                                          Outstanding debt (Dai)
    rap  0                                          Accumulated stability fee (Dai)
    -->  0                                          Collateralization ratio

    spot 8.234955555555555555555555555              REP price with safety mat (USD)
    rate 1.000000000000000000000000000              REP DAI exchange rate

----------

**6. Finally, remove your collateral REP token from the REP adapter.**  

**Run:**

`$ mcd --ilk=REP-A gem exit 40` 

**Example Output**

    vat 0.000000000000000000 Unlocked collateral (REP)
    ink 0.000000000000000000 Locked collateral (REP)
    ext 40.000000000000000000 External account balance (REP)

After running the above commands, please confirm that you have your initial collateral (REP) back in your wallet.  

This concludes the CDP Lifecycle Walkthrough Guide!

## Help
If you have any questions, don't hesitate to reach out on chat.makerdao.com in the #help channel.

## Additional resources
- [mcd-cli](https://github.com/makerdao/mcd-cli)
- [MCD 101](https://github.com/makerdao/developerguides/blob/master/mcd/mcd-101/mcd-101.md)
- Multi Collateral Dai source code: [https://github.com/makerdao/dss](https://github.com/makerdao/dss)
- Multi Colalteral Dai documentation: [https://github.com/makerdao/dss/blob/master/DEVELOPING.md](https://github.com/makerdao/dss/blob/master/DEVELOPING.md) & [https://github.com/makerdao/dss/wiki](https://github.com/makerdao/dss/wiki)
 - [Whitepaper](https://makerdao.com/whitepaper/)
