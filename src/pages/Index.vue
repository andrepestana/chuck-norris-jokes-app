<template>
  <q-page
    class="flex flex-center full-height">
    <transition name="slide-fade">
      <q-btn
        id="download-link"
        round
        color="primary"
        size="md"
        icon="save_alt"
        v-if="showButtons"
        @click="generateScreenshotAndDownload"
        data-html2canvas-ignore="true"
      ></q-btn>
    </transition>
    <transition name="slide-fade">
      <q-btn
        id="share-screenshot"
        round
        color="primary"
        size="md"
        icon="share"
        v-if="showButtons"
        @click="share"
        data-html2canvas-ignore="true"
      ></q-btn>
    </transition>
    <transition name="slide-fade">
      <q-btn
        id="open-drawer-button"
        round
        color="primary"
        size="md"
        icon="more_vert"
        v-if="showButtons"
        @click="$parent.$parent.$parent.leftDrawerOpen = !$parent.$parent.$parent.leftDrawerOpen"
        data-html2canvas-ignore="true"
      ></q-btn>
    </transition> 
    <div
      class="full-height bg-serious"
      id="main-div">
      <transition name="fade">
        <transitional-slide v-if="transitionHappening"></transitional-slide> 
      </transition>
      <div id="phrase-outer-div">
          <div 
            id="phrase-text-div"
            v-html="phrase">
          </div>
          <div id="next-div">
            <transition name="bounce">
              <div id="next"
                  @click="loadPhrase"
                  v-if="showButtons"
                  data-html2canvas-ignore="true"></div>
            </transition>
          </div>
        </div>  
    </div>
  </q-page>
</template>

<script>
import TransitionalSlide from '../components/TransitionalSlide.vue'
import html2canvas from 'html2canvas'


export default {
  data() {
      return {
        phrase: "",
        transitionHappening: false,
        screenshot: null,
        title: "Chuck Norris Jokes",
        appWebsite: 'https://andrepestana.github.io/cn/',
        sharingMessage: 'Check this out! xD',
        sharedFileName: 'chucknorris.png',
        apiEndpoint: "https://api.icndb.com/jokes/random",
        showButtons: false
      }
  },
  methods: {
    loadPhrase: function() {
      this.setBgSmilingImage()
      this.showButtons = false
      this.transitionHappening = true
      this.phrase = ""
      this.$http.get(this.apiEndpoint).then(result => {
        this.transitionHappening = false
        this.phrase = result.body.value.joke
        setTimeout(() => {
            this.showButtons = true
            this.setBgSeriousImage()
            this.getScreenShot()
          }, 1500
        )
      }, error => {
        console.log(error)
      })
    }, 
    setBgSmilingImage: function() {
      document.getElementById('main-div').classList.remove('bg-serious')
      document.getElementById('main-div').classList.add('bg-smiling')
    },
    setBgSeriousImage: function() {
      document.getElementById('main-div').classList.remove('bg-smiling')
      document.getElementById('main-div').classList.add('bg-serious')
    },
    getScreenShot() {
      return new Promise((resolve, reject) => {
        const self = this

        html2canvas(document.getElementById('main-div'), {
              onclone: function (clonedDoc) {
                let newDiv = document.createElement("div") 
                let appWebsiteUrlNode = document.createTextNode(self.appWebsite)
                newDiv.appendChild(appWebsiteUrlNode)
                newDiv.classList.add('website-url')
                clonedDoc.getElementById('next-div').appendChild(newDiv)
          }
        }).then(function(canvas) {
            self.screenshot = canvas.toDataURL()
            resolve()
        }).catch(error => {
            console.log(error)
            reject()
        })
      })
    },
    generateScreenshotAndDownload() {
      if(this.screenshot) {
        if (window.cordova && cordova.platformId !== "browser") {
          this.downloadForCordova()
        } else {
          this.downloadForBrowser()
        }
      } else {
        this.getScreenShot().then(() => {
          if (window.cordova && cordova.platformId !== "browser") {
            downloadForCordova()
          } else {
            this.downloadForBrowser()
          }
        }).catch(error => {
          console.log(error)
        })
      }
    },
    downloadForCordova() {
      var blob = this.dataURItoBlob(this.screenshot)
      this.download('file.png', blob, 'image/png')
    },
    downloadForBrowser() {
      const blob = this.b64toBlob(this.screenshot, 'image/png');
      const url = window.URL.createObjectURL(blob)
      const link = document.createElement('a')
      link.href = url
      link.setAttribute('download', this.sharedFileName) //or any other extension
      document.body.appendChild(link)
      link.click()
    },
    b64toBlob(b64Data, contentType='', sliceSize=512)  {
      const byteCharacters = atob(b64Data.replace(/^data:image\/(png|jpeg|jpg);base64,/, ''));
      const byteArrays = [];

      for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
        const slice = byteCharacters.slice(offset, offset + sliceSize);

        const byteNumbers = new Array(slice.length);
        for (let i = 0; i < slice.length; i++) {
          byteNumbers[i] = slice.charCodeAt(i);
        }

        const byteArray = new Uint8Array(byteNumbers);
        byteArrays.push(byteArray);
      }

      const blob = new Blob(byteArrays, {type: contentType});
      return blob;
    },
    dataURLtoFile(dataurl, filename, type) {
 
        var arr = dataurl.split(','),
            //mime = arr[0].match(/:(.*?);/)[1],
            bstr = atob(arr[1]), 
            n = bstr.length, 
            u8arr = new Uint8Array(n);
            
        while(n--){
            u8arr[n] = bstr.charCodeAt(n);
        }
        
        return new File([u8arr], filename, type);
    },
    download(filename, data, mimeType) {
      var blob = new Blob([data], {
        type: mimeType
      });
      if (window.cordova && cordova.platformId !== "browser") {
        document.addEventListener("deviceready", function() {
          var storageLocation = "";

          switch (device.platform) {
            case "Android":
              storageLocation = cordova.file.externalDataDirectory;
              break;

            case "iOS":
              storageLocation = cordova.file.documentsDirectory;
              break;
          }

          var folderPath = storageLocation;

          window.resolveLocalFileSystemURL(
            folderPath,
            function(dir) {
              dir.getFile(
                filename,
                {
                  create: true
                },
                function(file) {
                  file.createWriter(
                    function(fileWriter) {
                      fileWriter.write(blob);

                      fileWriter.onwriteend = function() {
                        var url = file.toURL();
                        cordova.plugins.fileOpener2.open(url, mimeType, {
                          error: function error(err) {
                            console.error(err);
                            alert("Unable to download");
                          },
                          success: function success() {
                            console.log("success with opening the file");
                          }
                        });
                      };

                      fileWriter.onerror = function(err) {
                        alert("Unable to download");
                        console.error(err);
                      };
                    },
                    function(err) {
                      // failed
                      alert("Unable to download");
                      console.error(err);
                    }
                  );
                },
                function(err) {
                  alert("Unable to download");
                  console.error(err);
                }
              );
            },
            function(err) {
              alert("Unable to download");
              console.error(err);
            }
          );
        });
      } else {
        saveAs(blob, filename);
      }
    },
    dataURItoBlob(dataURI) {
      var isBase64 = dataURI.split(",")[0].split(";")[1] === "base64";
      var byteString;

      if (isBase64) {
        // convert base64 to raw binary data held in a string
        // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
        byteString = atob(dataURI.split(",")[1]);
      } else {
        byteString = dataURI.split(",")[1];
      } // separate out the mime component

      var mimeString = dataURI
        .split(",")[0]
        .split(":")[1]
        .split(";")[0]; // write the bytes of the string to an ArrayBuffer

      var ab = new ArrayBuffer(byteString.length); // create a view into the buffer

      var ia = new Uint8Array(ab); // set the bytes of the buffer to the correct values

      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      } // write the ArrayBuffer to a blob, and you're done

      var blob = new Blob([ab], {
        type: mimeString
      });
      return blob;
    },
    share() {
      if(this.screenshot) {
        window.plugins.socialsharing.share(
          this.sharingMessage, 
          this.sharedFileName, 
          this.screenshot,
          this.appWebsite)
      } else {
        this.getScreenShot().then(() => {
          window.plugins.socialsharing.share(
            this.sharingMessage, 
            this.sharedFileName, 
            this.screenshot,
            this.appWebsite)
        }).catch(error => {
          console.log(error)
        })
      }
    },
  },
  mounted() {
    this.loadPhrase()
  },
  components: {
    'transitionalSlide': TransitionalSlide
  }
}
</script>
<style lang="scss">
html, body {
  margin: 0;
  height: 100%;
  font-family: Arial, Helvetica, sans-serif;
}
#q-app, .q-page-container, .q-layout {
  height: 100%;
  margin: 0;
}
#main-div {
  width: 100%;
  //background-image: url('~assets/chucknorris.jpg');
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-position: center; /* Center the image */
  background-repeat: no-repeat; /* Do not repeat the image */
}
.bg-smiling {
  background-image: url('~assets/chucknorris_smiling.jpg');
}
.bg-serious {
  background-image: url('~assets/chucknorris.jpg');
}
#phrase-outer-div {
  margin-right: 120px;
  text-align: center;
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translate(-50%, -100%);
  color: rgb(255, 230, 0);
  text-shadow: 0.075em 0.08em 0.1em rgba(0, 0, 0, 1);
  font-size: 3vh;
  width: fit-content;
  font-weight: bold;
}
#phrase-text-div {
  background-color: rgba(128, 128, 128, 0.7);
  margin:0px;
}
.website-url {
    color: rgb(242, 246, 246);
    font-family: 'Courier New', Courier, monospace;
    font-size: 16px;
    background-color: rgba(11, 6, 49, 0.746);
    border-radius: 5px;
}
#next-div {
  border: none;
  font-size: 24px;
  padding: 0px;
  margin: 0;
  cursor: pointer;
  text-align: center;
  display: inline-block;
  min-height: 120px;
}
#next {
  background-image: url('~assets/button.png'); 
  width: 150px;
  height: 120px;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-position: center; /* Center the image */
  background-repeat: no-repeat; /* Do not repeat the image */
  cursor: pointer;
}
#next:active {
  background-image: url('~assets/button.png'); 
  width: 140px;
  height: 100px;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  background-position: center; /* Center the image */
  background-repeat: no-repeat; /* Do not repeat the image */
  cursor: pointer;
}
#open-drawer-button {
  position: fixed;
  top: 20px;
  right: 10px;
  cursor: pointer;
  z-index: 99;
}
#download-link {  
  position: fixed;
  top: 80px;
  right: 10px;
  cursor: pointer;
  z-index: 99;
}
#share-screenshot {
  position: fixed;
  top: 140px;
  right: 10px;
  cursor: pointer;
  z-index: 99;
}
.fade-enter-active {
  transition: opacity .5s;
}
.fade-leave-active {
  transition: opacity 1.3s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  //animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
.slide-fade-enter-active {
  transition: all .8s ease;
}
.slide-fade-leave-active {
  transition: all .5s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(20px);
  opacity: 0;
}
</style>
