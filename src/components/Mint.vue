<template>
  <div class="white-b" id="main">
    <loading-screen v-if="!mounted" />
    <v-snackbar v-model="snackbar" :timeout="-1">
      <p style="font-size: 2rem; color: white">
        {{ snackBarMsg }}
      </p>

      <template v-slot:action="{ attrs }">
        <v-btn
          color="pink"
          text
          v-bind="attrs"
          @click="closeSnackbar"
          style="font-size: 2rem"
        >
          X
        </v-btn>
      </template>
    </v-snackbar>

    <v-dialog v-model="dialog" max-width="500">
      <v-card>
        <v-card-title class="text-h5">
          <p>
            <strong
              >Congrats! <br />
              Your mint was successful!</strong
            >
          </p>
        </v-card-title>

        <v-card-text>
          <p><small> Welcome to the Cryptocubans family! </small></p>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn
            class="mb-4"
            color="green darken-1"
            text
            @click="dialog = false"
            style="font-size: 2rem"
          >
            Close
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

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
            style="padding-bottom: 2rem; padding-right: 2rem"
            id="intro"
          >
            <!-- CONTAINER -->
            <div class="container">
              <p class="tag m-b-4">Cryptocubans</p>
              <h1 class="m-b-5 m-l-3">Mint</h1>
              <div class="seperator-line"></div>
              <v-btn
                :disabled="loading"
                :loading="loading"
                v-if="!user"
                class="m-t-3 default-button button-filled"
                elevation="2"
                rounded
                x-large
                v-on:click="connectWallet"
              >
                CONNECT WALLET
              </v-btn>
              <div v-else>
                <div>
                  <p><strong>Address: </strong> {{ user.formattedAddress }}</p>
                  <p><strong>Balance: </strong> {{ user.balance }}</p>
                </div>
                <div class="seperator-line"></div>
                <div>
                  <!-- <p><strong>Contract: </strong> {{ contractAddress }}</p> -->
                  <p>
                    <strong>NFTs: </strong> {{ totalSupply }} / {{ maxAmount }}
                  </p>
                  <p><strong>Price: </strong> {{ price }} ETH + gas</p>
                </div>
                <v-alert color="#480058" dark dense prominent>
                  You can now mint up to {{ maxPerWallet }} Cryptocubans.
                  <br />

                  <strong>
                    # NFTs minted by you so far: {{ balanceOf }} /
                    {{ maxPerWallet }}
                  </strong>
                </v-alert>
                <div class="seperator-line"></div>
                <div v-if="!isBlocked">
                  <v-row>
                    <v-col>
                      <v-select
                        :disabled="loading"
                        :loading="loading"
                        elevation="2"
                        rounded
                        x-large
                        :items="tokens"
                        v-model="numberOfTokens"
                        label="MINT"
                        solo
                        v-on:change="mint"
                      >
                      </v-select>
                    </v-col>
                  </v-row>
                </div>
                <div v-else>
                  <v-alert color="green" dense type="info">
                    <strong>
                      You have reached the maximum amount of NFTs you can mint.
                      <br />
                      ({{ maxPerWallet }} NFTs)
                    </strong>
                  </v-alert>
                </div>
                <div class="seperator-line"></div>
                <v-row align="center" justify="space-around">
                  <v-col sm="12" md="6">
                    <v-btn
                      class="default-button button-filled"
                      elevation="2"
                      rounded
                      x-large
                      :href="'https://etherscan.io/token/' + contractAddress"
                      target="_blank"
                    >
                      SMART CONTRACT
                    </v-btn>
                  </v-col>
                  <v-col sm="12" md="6">
                    <v-btn
                      class="default-button button-filled"
                      elevation="2"
                      rounded
                      x-large
                      :href="'https://opensea.io/collection/cryptocuban-social-club'"
                      target="_blank"
                    >
                      OPENSEA
                    </v-btn>
                  </v-col>
                </v-row>
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
import axios from "axios";
import ABI from "../assets/contract/ABI.json";
import LoadingScreen from "./LoadingScreen.vue";
import Web3Modal from "web3modal";
import WalletConnectProvider from "@walletconnect/web3-provider";
import Torus from "@toruslabs/torus-embed";
import WalletLink from "walletlink";

export default {
  name: "Mint",
  components: {
    "loading-screen": LoadingScreen,
  },
  data: () => {
    return {
      maxPerMint: 0,
      maxPerWallet: 0,
      numberOfTokens: null,
      price: 0,
      maxAmount: 0,
      totalSupply: 0,
      user: null,
      provider: null,
      signer: null,
      contract: null,
      abi: ABI.abi,
      contractAddress: "0xca9eE3460D84Eac6C2F2284CFe3E3B35A2267d78", // Replace it with your contract address,
      collectionId: "KkWtXwKdIwQXhTvxrIHU", // Replace it with your Nifty Kit Collection ID,
      loading: false,
      mounted: false,
      tokens: [],
      snackbar: false,
      snackBarMsg: "",
      presaleActive: false,
      dialog: false,
      balanceOf: 0,
      isBlocked: false,
    };
  },
  methods: {
    connectWallet: async function () {
      try {
        this.loading = true;

        const infuraId = "23cf1ohT2vMJPnvg9xycCGCt7hr";

        const providerOptions = {
          walletconnect: {
            package: WalletConnectProvider,
            options: {
              infuraId: infuraId,
            },
          },
          torus: {
            package: Torus,
          },
          walletlink: {
            package: WalletLink,
            options: {
              appName: "Cryptocuban Social Club",
              infuraId,
            },
          },
        };

        const web3Modal = new Web3Modal({
          providerOptions,
        });

        const instance = await web3Modal.connect();
        if (!instance) {
          throw new Error("No wallet selected");
        }
        this.provider = new ethers.providers.Web3Provider(instance);

        if (instance.on) {
          instance.on("disconnect", () => {
            window.location.reload();
          });
          instance.on("accountsChanged", () => {
            window.location.reload();
          });
          instance.on("chainChanged", () => {
            window.location.reload();
          });
        }

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
          formattedAddress: "",
        };

        // formats the balance to be in ether
        this.user.balance = ethers.utils.formatEther(this.user.balance);
        this.user.balance = Math.round(this.user.balance * 1e4) / 1e4;

        // formats the address to be more readable
        this.user.formattedAddress =
          this.user.address.substring(0, 6) +
          "..." +
          this.user.address.substring(
            this.user.address.length - 4,
            this.user.address.length
          );

        // Loads the contract
        this.contract = new ethers.Contract(
          this.contractAddress,
          this.abi,
          this.signer
        );

        // Gets total supply
        this.totalSupply = await this.contract.totalSupply();

        // Gets max amount
        this.maxAmount = await this.contract.maxAmount();

        // Gets max per mint
        this.maxPerMint = await this.contract.maxPerMint();

        // Gets max per wallet
        this.maxPerWallet = await this.contract.maxPerWallet();

        // Gets balance of the current address
        this.balanceOf = await this.contract.balanceOf(this.user.address);

        this.isBlocked = this.balanceOf >= this.maxPerWallet;

        // Gets presale active
        this.presaleActive = await this.contract.presaleActive();

        // Fills the tokens array with the total supply
        for (let i = 0; i < this.maxPerMint - this.balanceOf; i++) {
          this.tokens.push(i + 1);
        }

        // Gets price
        this.price = ethers.utils.formatEther(await this.contract.price());
      } catch (e) {
        this.handleError(e);
      } finally {
        this.loading = false;
      }
    },
    mint: async function () {
      try {
        this.loading = true;

        if (this.isBlocked) {
          throw new Error(
            "You have reached the maximum amount of tokens you can mint."
          );
        }
        this.numberOfTokens = Number(
          Math.min(Number(this.numberOfTokens), this.maxPerMint)
        );
        const amount = ethers.utils
          .parseEther(this.price)
          .mul(this.numberOfTokens);

        const args = [this.numberOfTokens];
        const mintFunc = this.presaleActive
          ? this.contract.presaleMint
          : this.contract.mint;

        if (this.presaleActive) {
          const data = await this.generateProof();
          if (!data.proof) {
            throw new Error("Your wallet is not part of presale.");
          }
          args.push(data.proof);
        }
        // see https://docs.ethers.io/v5/api/contract/contract/#Contract--write
        const tx = await mintFunc(...args, {
          value: amount,
        });
        await tx.wait();

        // Minted successfully
        this.totalSupply = await this.contract.totalSupply();
        this.balanceOf = await this.contract.balanceOf(this.user.address);

        this.isBlocked = this.balanceOf >= this.maxPerWallet;

        // Fills the tokens array with the total supply
        for (let i = 0; i < this.maxPerMint - this.balanceOf; i++) {
          this.tokens.push(i + 1);
        }
        this.dialog = true;
      } catch (e) {
        this.handleError(e);
      } finally {
        this.loading = false;
        this.numberOfTokens = null;
      }
    },
    // Merkle Proof to veriy that the wallet belongs to the presale list
    generateProof: async function () {
      try {
        const { data } = await axios.post(
          `https://app.niftykit.com/api/drops/list/${this.collectionId}`,
          {
            wallet: this.user.address,
          }
        );
        return data;
      } catch (e) {
        return { proof: null };
      }
    },
    openSnackbar: function (msg) {
      this.snackbar = true;
      this.snackBarMsg = msg;
    },
    closeSnackbar() {
      this.snackbar = false;
      this.snackBarMsg = "";
    },
    handleError: function (e) {
      const msg = e?.error?.message || e.message;
      // checks if the user has metamask installed
      if (msg.includes("missing provider")) {
        this.openSnackbar(
          "Please install the MetaMask extension. If you are on mobile, open your MetaMask app and browse to this page."
        );
        return;
      }

      if (msg.includes("err: insufficient funds for gas * price + value")) {
        this.openSnackbar("You do not have enough balance.");
        return;
      }

      this.openSnackbar(msg);
    },
  },
  beforeMount() {},
  mounted() {
    this.mounted = true;
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

p {
  color: #480058;
}

p::selection {
  color: #fff;
}

#cc-gif {
  background-image: url("~@/assets/images/cryptocubans.gif");
}

.button-filled {
  background-color: #480058 !important;
  background: #480058;
  padding: 0 4.5rem !important;
  color: #fff !important;
  font-size: 2rem !important;
  min-width: none !important;
}

::v-deep .v-snack__wrapper {
  font-size: 2rem !important;
}
</style>
