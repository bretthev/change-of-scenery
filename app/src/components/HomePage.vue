<template>
  <section class='search-container' v-on:keyup.27='closeModal'>
    <button class='photo-info-button' @click='openPhotoInfoModal'>Current Background Info</button>
    <input class='search-input' v-model='search' v-on:keyup.13='previewBackground' type='text' placeholder='Find a background'>
    <button class='submit-search-button' @click='previewBackground'>Get Image</button>
    <h1 class='error-message'>{{ errorMessage }}</h1>
    <section class='modal-container' v-show='showPhotographerModal'>
      <current-picture-info
        :closePhotoInfoModal='closePhotoInfoModal'
        :currentImage='currentImage'>
      </current-picture-info>
    </section>
    <section class='modal-preview-container' v-show='showPreviewModal'>
      <background-preview
        :previewBackground='previewBackground'
        :saveBackground='saveBackground'
        :thumbUrl='thumbUrl'
        :closePreviewModal='closePreviewModal'>
      </background-preview>
    </section>
  </section>
</template>

<script>
  import CurrentPictureInfo from './CurrentPictureInfo'
  import BackgroundPreview from './BackgroundPreview'
  const applescript = require('applescript')
  const axios = require('axios')
  const { remote } = require('electron')
  const path = require('path')
  const mainProcess = remote.require(path.join(process.cwd(), 'app/electron.js'))
  const downloadPath = path.join(process.cwd(), '/app', '/imageDownloads')
  const setCurrentPictureInLocalStorage = (pictureData, searchTerm) => {
    localStorage.setItem('currentPicture', JSON.stringify({
      photographer: pictureData.user.name,
      photographerWebsite: pictureData.user.portfolio_url,
      photoUrl: pictureData.links.html,
      searchTerm: searchTerm || 'random'
    }))
  }
  const showLoader = () => {
    document.querySelector('.background-preview').style.display = 'none'
    document.querySelector('.loader').style.display = 'block'
  }
  const hideLoader = () => {
    document.querySelector('.background-preview').style.display = 'block'
    document.querySelector('.loader').style.display = 'none'
  }

  let imageId

  export default {
    components: {
      CurrentPictureInfo,
      BackgroundPreview
    },
    data () {
      return {
        search: '',
        errorMessage: '',
        thumbUrl: '',
        imageData: null,
        showPhotographerModal: false,
        showPreviewModal: false,
        currentImage: {
          photographer: 'No Image Downloaded'
        }
      }
    },
    created () {
      this.getCurrentImage()
    },
    methods: {
      closePhotoInfoModal () {
        this.showPhotographerModal = false
      },
      openPhotoInfoModal () {
        this.showPhotographerModal = true
      },
      closePreviewModal () {
        this.showPreviewModal = false
        this.search = ''
      },
      openPreviewModal () {
        this.showPreviewModal = true
      },
      previewBackground () {
        this.errorMessage = ''
        hideLoader()
        axios.get(`https://api.unsplash.com/photos/random?query=${this.search}&client_id=f3ff11ed9e9a4de213e05ff00fa5e4f503cdf0b595de8dfd2d59cad26f7efb3f`)
        .then((response) => {
          this.imageData = response.data
          this.thumbUrl = response.data.urls.thumb
          this.openPreviewModal()
        })
        .catch((error) => {
          this.errorMessage = 'No images match that search, sorry.'
          console.log(error)
        })
      },
      saveBackground () {
        showLoader()
        imageId = Date.now()
        mainProcess.savePicture(this.imageData, imageId)
        setTimeout(this.setBackground, 3000)
      },
      setBackground () {
        setCurrentPictureInLocalStorage(this.imageData, this.search)
        this.getCurrentImage()
        this.closePreviewModal()
        let script = `tell application "Finder" to set desktop picture to POSIX file  "${downloadPath}/background${imageId}.jpg"`
        applescript.execString(script, (error, response) => {
          if (error) {
            console.log(error)
          }
        })
        this.search = ''
      },
      getCurrentImage () {
        let imageInLocalStorage = JSON.parse(localStorage.getItem('currentPicture'))
        if (imageInLocalStorage) {
          this.currentImage = imageInLocalStorage
        }
      }
    }
  }
</script>

<style lang='scss' scoped>
  $button-color: #DD1C1A;
  $button-text: #FFF1D0;

  @mixin flex-display-center {
    align-items: center;
    display: flex;
    justify-content: center;
  }

  .search-container {
    @include flex-display-center;
    background: rgba(0, 0, 0, 0.9);
    flex-direction: column;
    height: 250px;
    width: 100vw;
  }

  .search-input {
    background: #fff;
    border: none;
    border-radius: 50px;
    box-shadow: inset 0 0 10px #000000;
    color: #010101;
    font-size: 26px;
    outline: none;
    padding: 15px;
    text-align: center;
    width: 400px;
    &:focus {
      background: #d0d2d6;
      transition: 0.5s;
    }
  }

  .submit-search-button {
    background: $button-color;
    border: none;
    border-radius: 25px;
    box-shadow: 0px 2px 4px 0px rgba(0,0,0,0.50);
    color: $button-text;
    font-size: 20px;
    margin: 15px 0;
    outline: none;
    padding: 10px;
    width: 120px;
    &:hover {
      background: darken($button-color, 15%);
      cursor: pointer;
      transition: 0.4s;
    }
  }

  .modal-container {
    @include flex-display-center;
    background-color: rgba(0, 0, 0, .7);
    height: 100%;
    position: fixed;
    transition: opacity .3s ease;
    width: 100%;
    z-index: 1;
  }

  .modal-preview-container {
    @include flex-display-center;
    position: fixed;
  }

  .error-message {
    color: $button-text;
    font-size: 20px;
    height: 20px;
  }

  .photo-info-button {
    background: $button-color;
    border: 5px solid rgba(0, 0, 0, 0.9);
    border-radius: 50%;
    color: $button-text;
    font-size: 15px;
    height: 125px;
    left: 15px;
    line-height: 22px;
    outline: none;
    position: absolute;
    text-transform: uppercase;
    transition: all 0.8s ease;
    top: 15px;
    width: 125px;
    &:hover {
      background: $button-text;
      color: $button-color;
      cursor: pointer;
      transition: all 0.8s ease-out;
    }
  }
</style>
