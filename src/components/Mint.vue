<template>
  <div class="white-b" id="main">
    <!-- LEFT CONTENT -->
    <div class="left-content noselect">
      <!-- INNER CONTENT -->
      <div class="inner-content">
        <!-- NAME -->
        <div class="name">
          <p class="small color-white"></p>
        </div>
        <!-- /NAME -->
        <div class="picture-box">
          <div class="picture"></div>
        </div>
        <div class="picture-box">
          <div id="cc-gif" class="picture"></div>
          <canvas
            width="874"
            height="1076"
            style="display: block; width: 437px; height: 538px"
          ></canvas>
        </div>
        <div class="frame-1 frame"></div>
        <div class="frame-2 frame"></div>
        <div class="frame-3 frame"></div>
      </div>
      <!-- /INNER CONTENT -->
    </div>
    <!-- /LEFT CONTENT -->
    <!-- RIGHT CONTENT -->
    <div class="right-content">
      <!-- INNER CONTENT -->
      <div class="inner-content">
        <!-- SECTION CONTAINER -->
        <div class="section-container">
          <!-- SECTION ( INTRODUCTION ) -->
          <div
            class="section height-100 y-center"
            style="padding-bottom: 2rem"
            id="intro"
          >
            <!-- CONTAINER -->
            <div class="container">
              <p class="tag m-b-4">Cryptocubans</p>
              <h1 class="m-b-5 m-l-3">Mint</h1>
              <div class="seperator-line"></div>
              <button
                :disabled="loading"
                v-if="!user"
                class="m-t-3 default-button button-filled"
                v-on:click="connectWallet"
              >
                CONNECT WALLET
              </button>
              <div v-else>
                <div>
                  <p><strong>Address: </strong> {{ user.address }}</p>
                  <p><strong>Balance: </strong> {{ user.balance }}</p>
                </div>
                <div class="seperator-line"></div>
                <div>
                  <p><strong>Contract: </strong> {{ contractAddress }}</p>
                  <p><strong>NFTs: </strong> {{ totalSupply }} / 1492</p>
                  <p><strong>Max Per Mint: </strong> {{ maxPerMint }}</p>
                  <p><strong>Price: </strong> {{ parsedPrice }} ETH</p>
                </div>
                <div class="seperator-line"></div>
                <div>
                  <input
                    :disabled="loading"
                    type="number"
                    min="1"
                    :max="maxPerMint"
                    v-model="numberOfTokens"
                  />
                  <button
                    :disabled="loading"
                    class="m-t-3 default-button button-filled default-button"
                    v-on:click="mint"
                  >
                    MINT
                  </button>
                </div>
              </div>
            </div>
            <!-- /CONTAINER -->
          </div>
          <!-- /SECTION ( INTRODUCTION ) -->
        </div>
        <!-- /SECTION CONTAINER -->
      </div>
      <!-- /INNER CONTENT -->
    </div>
    <!-- /RIGHT CONTENT -->
  </div>
</template>
<script>
import { ethers } from "ethers";
import ABI from "../assets/contract/ABI.json";

export default {
  name: "Mint",
  data: () => {
    return {
      maxPerMint: 0,
      numberOfTokens: 1,
      price: 0,
      parsedPrice: 0,
      totalSupply: 0,
      user: null,
      provider: null,
      signer: null,
      contract: null,
      abi: ABI.abi,
      contractAddress: "0x49640c1327878ca58bd70754a5fb52fbe6a0ce70", // Replace it with your contract address,
      loading: false,
    };
  },
  methods: {
    connectWallet: async function () {
      try {
        this.loading = true;
        // The "any" network will allow spontaneous network changes
        this.provider = new ethers.providers.Web3Provider(
          window.ethereum,
          "any"
        );

        // Best practice: reload the page if the network changes
        this.provider.on("network", (newNetwork, oldNetwork) => {
          // When a Provider makes its initial connection, it emits a "network"
          // event with a null oldNetwork along with the newNetwork. So, if the
          // oldNetwork exists, it represents a changing network
          if (oldNetwork) {
            window.location.reload();
          }
        });

        // Prompt user for account connections
        await this.provider.send("eth_requestAccounts", []);
        this.signer = this.provider.getSigner();
        const network = await this.provider.getNetwork();
        if (network.chainId !== 1) {
          throw new Error(
            `You are not on the correct network (${network.name}) please switch to the Ethereum Mainnet.`
          );
        }
        this.user = {
          address: await this.signer.getAddress(),
          balance: await this.signer.getBalance(),
        };

        // formats the balance to be in ether
        this.user.balance = ethers.utils.formatEther(this.user.balance);

        // Loads the contract
        this.contract = new ethers.Contract(
          this.contractAddress,
          this.abi,
          this.signer
        );

        // Gets total supply
        this.totalSupply = Number(await this.contract.totalSupply());

        // Gets max per mint
        this.maxPerMint = Number(await this.contract.maxPerMint());

        // Gets price
        this.price = await this.contract.price();
        this.parsedPrice = ethers.utils.formatEther(this.price);

      } catch (e) {
        console.log("In Catch Block: Error : ", e.message);
      } finally {
        this.loading = false;
      }
    },
    mint: async function () {
      try {
        this.loading = true;
        this.numberOfTokens = Math.min(this.numberOfTokens, this.maxPerMint);
        const amount = ethers.utils.parseEther(
          (this.parsedPrice * this.numberOfTokens).toString()
        );
        // see https://docs.ethers.io/v5/api/contract/contract/#Contract--write
        const tx = await this.contract.mint(this.numberOfTokens, {
          value: amount,
        });
        console.log({
          tx,
        });
        await tx.wait();
      } catch (e) {
        console.log("In Catch Block: Error : ", e.message);
      } finally {
        this.loading = false;
      }
    },
  },
  beforeMount() {},
  mounted() {
    // this.connectWallet();
    // Remove the loader
    // document.getElementsByClassName("loading-screen")[0].style.display = "none";
  },
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.white-b {
  background-color: #fdc60c;
}
::selection {
  background: #480058;
}

#cc-gif {
  background-image: url("~@/assets/images/cryptocubans.gif");
}
</style>
