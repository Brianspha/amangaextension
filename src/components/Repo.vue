<template>

  <v-card class="mx-auto">

    <v-toolbar color="indigo" dark>
      <v-app-bar-nav-icon></v-app-bar-nav-icon>

      <v-toolbar-title>Reported URLS</v-toolbar-title>
      <div class="flex-grow-1"></div>
      <div class="flex-grow-1"></div>
      <div class="flex-grow-1"></div>
      <div class="flex-grow-1"></div>

      <v-text-field
        flat
        solo-inverted
        hide-details
        prepend-inner-icon="search"
        label="URL"
        hint="e.g. http://enca.co.za/laksjdalk"
        class="hidden-sm-and-down pa-2"
        v-model="url"
      ></v-text-field>

      <div class="flex-grow-1"></div>

      <v-btn icon>
        <v-icon>mdi-magnify</v-icon>
      </v-btn>
    </v-toolbar>

    <v-list three-line>
      <template v-for="(post, index) in filteredList">
        <v-subheader v-if="post.header" :key="post.header" v-text="post.post"></v-subheader>

        <v-divider
          v-else-if="post.divider"
          :key="index"
          :inset="post.inset"
        ></v-divider>

        <v-list-item v-else :key="post.url" @click>
          <v-list-item-avatar>
            <v-img :src="post.img"></v-img>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title>{{post.date_reported}}</v-list-item-title>
            <v-list-item-subtitle>
              <a :href="post.url">{{post.url}}</a>
            </v-list-item-subtitle>
            <v-list-item-subtitle>Legatimicy: {{isNaN(post.legit)?'0':post.legit}}%</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
            <v-container fluid class="text-center" v-bind:key="index" @click>
            <v-row justify="space-between">
                <v-col cols="12">
                <v-badge left>
                    <template v-slot:badge>{{post.up_votes}}</template>
                    <v-btn icon @click="selected_post=post;down_vote=false;dialog=true">
                    <v-icon color="grey lighten-1">thumb_up</v-icon>
                    </v-btn>
                </v-badge>
                <v-badge v-bind:key="index" right>
                    <template v-slot:badge>{{post.up_votes}}</template>
                    <v-btn icon @click="selected_post=post;down_vote=true;dialog=true">
                    <v-icon color="grey lighten-1">thumb_down</v-icon>
                    </v-btn>
                </v-badge>
                </v-col>
            </v-row>
            </v-container>
      </template>
    </v-list>


<template>
  <v-row justify="center">
    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Vote Legatimicy of URL</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col >
                <v-text-field hint="e.g. 0x1lk2jlkasd..." label="Your ethereum address" required></v-text-field>
              </v-col> 
              
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <div class="flex-grow-1"></div>
          <v-btn color="blue darken-1" text @click="dialog = false;">Close</v-btn>
          <v-btn color="blue darken-1" text @click="dialog = false;vote()">Submit</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

  </v-card>
</template>

<script>
import Web3 from "web3";
import Swal from "sweetalert2";
import loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";
export default {
  data: () => ({
      userAddress:'',
      selected_post:{},
     down_vote:false, 
    dialog: false,
    isLoading: false,
    url: "",
    urls: [],
    web3: new Web3(new Web3.providers.HttpProvider("http://localhost:11003")),
    abi:[{"constant":true,"inputs":[{"name":"url","type":"bytes"}],"name":"check_reached_max_votes","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x10b8823c"},{"constant":true,"inputs":[{"name":"url","type":"bytes"}],"name":"is_url_reported","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x16f3b1ca"},{"constant":false,"inputs":[{"name":"url","type":"bytes"},{"name":"reason","type":"bytes32"},{"name":"user","type":"address"}],"name":"report_url","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x415e757c"},{"constant":true,"inputs":[],"name":"get_min_token_payout","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x5189a954"},{"constant":true,"inputs":[{"name":"url","type":"bytes"}],"name":"get_url_details","outputs":[{"name":"","type":"address"},{"name":"","type":"uint256"},{"name":"","type":"uint256"},{"name":"","type":"uint256"},{"name":"","type":"uint256"},{"name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x6a6c5d90"},{"constant":true,"inputs":[],"name":"get_urls_reported_keys","outputs":[{"name":"","type":"bytes[]"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x7094e402"},{"constant":false,"inputs":[{"name":"url","type":"bytes"},{"name":"up_down_vote","type":"uint256"},{"name":"user","type":"address"}],"name":"up_down_vote_url","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x751c9c56"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor","signature":"constructor"}],    
    amangaContract: {},
    contractAddress: "0x638c4a669FdC0ab3fB2Dd7FDADc1B94CE1A868F7",
    relayAddress: "0xae3478f55ae21b9139652467d0b73177d6bec204"
  }),
  mounted() {
    /**class Post {
  constructor(title, url, author, img) {
    this.title = title;
    this.url = url;
    this.author = author;
    this.img = img;
  }
}
 */
    this.init();
    this.getUrls();
  },
  methods: {
    init() {
      this.amangaContract = new this.web3.eth.Contract(
        this.abi,
        this.contractAddress
      );
      console.log(this.amangaContract);
      console.log(this.web3);
    },
    vote:async function(){
        let This=this
        this.isLoading=true
        if(this.down_vote){
            this.amangaContract.methods.up_down_vote_url(this.web3.utils.toHex(this.selected_post.url), 1,this.userAddress).send({
            gas: 8000000,
            from: this.relayAddress
        }).then((receipt,err)=>{
            console.log(receipt)
            if(!err){
                This.success('success!!')
            }
            This.isLoading=false
        }).catch((err)=>{
            console.log('err: ',err)
            This.isLoading=false
            This.error('Something went wrong!!')
        })
        }
        else{
    this.amangaContract.methods.up_down_vote_url(this.web3.utils.toHex(this.selected_post.url), 0,this.userAddress).send({
            gas: 8000000,
            from: this.relayAddress
        }).then((receipt,err)=>{
            console.log(receipt)
            if(!err){
                This.success('success!!')
            }
            This.isLoading=false
        }).catch((err)=>{
            console.log('err: ',err)
            This.isLoading=false
            This.error('Something went wrong!!')
        })
        }
        console.log('down_voting: ',post)
    },
    getUrls: async function() {
      let This = this;
      this.amangaContract.methods
        .get_urls_reported_keys()
        .call({
          gas: 8000000,
          from: this.relayAddress
        })
        .then((keys, err) => {
          if (!err && keys.length === 0) {
            this.error("There arent any reported urls yet");
          } else if (keys.length > 0) {
              var index=0
            keys.map(key => {
              This.amangaContract.methods
                .get_url_details(key)
                .call({
                  gas: 8000000,
                  from: this.relayAddress
                })
                .then((details, err) => {
                  This.urls.push({
                    url: this.web3.utils.toAscii(key),
                    img:
                      "https://image.flaticon.com/icons/svg/1862/1862164.svg",
                    title: this.web3.utils.toAscii(key),
                    up_votes: details[1],
                    down_votes: details[2],
                    author: details[0],
                    reason: this.web3.utils.toAscii(details[5]),
                    date_reported: new Date(
                      parseInt(details[4]) * 1000
                    ).toUTCString(),
                    legit: Math.round(
                      (details[1] / (details[1] + details[2])) * 100,
                      5
                    )
                  });
                  index++
                  if(index +1 < keys.length){
                  This.urls.push({ divider: true, inset: true });
                  }
                  console.log({
                    url: this.web3.utils.toAscii(key),
                    img:
                      "https://image.flaticon.com/icons/svg/1862/1862164.svg",
                    title: this.web3.utils.toAscii(key),
                    up_votes: details[1],
                    down_votes: details[2],
                    author: details[0],
                    reason: this.web3.utils.toAscii(details[5]),
                    date_reported: new Date(
                      parseInt(details[4]) * 1000
                    ).toUTCString(),
                    legit: Math.round(
                      (details[1] / (details[1] + details[2])) * 100,
                      5
                    )
                  });
                })
                .catch(err => {
                  console.log("error getting url details: ", err);
                });
            });
          }
        })
        .catch(err => {
          console.log("error getting urls: ", err);
        });
    },
    success(message) {
      Swal.fire("Success", message, "success");
    },
    error(message) {
      Swal.fire({
        type: "error",
        title: "Oops...",
        text: message,
        allowOutsideClick: true
      });
    }
  },
  computed: {
    filteredList() {
      return this.urls.filter(post =>
        post.url.toLowerCase().includes(this.url.toLowerCase())
      );
    }
  }
};
</script>