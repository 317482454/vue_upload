<template>
  <div class="hello">
    <button @click="limitClick(1)">点击设置上传数量：2</button>
    <button @click="limitClick(0)">点击取消上传数量</button>
    <div class="upload">
      <div class="upload_warp">
        <div class="upload_warp_left" @click="fileClick">
          <img src="./upload.png">
        </div>
        <div class="upload_warp_right" @drop="drop($event)" @dragenter="dragenter($event)" @dragover="dragover($event)">
          或者将文件拖到此处
        </div>
      </div>
      <div class="upload_warp_text">
        选中{{imgList.length}}张文件，共{{bytesToSize(this.size)}}
      </div>
      <input @change="fileChange($event)" type="file" id="upload_file" multiple style="display: none"/>
      <div class="upload_warp_img" v-show="imgList.length!=0">
        <div class="upload_warp_img_div" v-for="(item,index) of imgList">
          <div class="upload_warp_img_div_top">
            <div class="upload_warp_img_div_text">
              {{item.file.name}}
            </div>
            <img src="./del.png" class="upload_warp_img_div_del" @click="fileDel(index)">
          </div>
          <img :src="item.file.src">
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'hello',
    data() {
      return {
        imgList: [],
        size: 0,

      }
    },
    methods: {
      //设置
      limitClick(state) {
        this.imgList = [];
        if (state)
          this.limit = 2;
        else
          this.limit = undefined;
      },
      fileClick() {
        document.getElementById('upload_file').click()
      },
      fileChange(el) {
        if (!el.target.files[0].size) return;
        this.fileList(el.target);
        el.target.value = ''
      },
      fileList(fileList) {
        let files = fileList.files;
        for (let i = 0; i < files.length; i++) {
          //判断是否为文件夹
          if (files[i].type != '') {
            this.fileAdd(files[i]);
          } else {
            //文件夹处理
            this.folders(fileList.items[i]);
          }
        }
      },
      //文件夹处理
      folders(files) {
        let _this = this;
        //判断是否为原生file
        if (files.kind) {
          files = files.webkitGetAsEntry();
        }
        files.createReader().readEntries(function (file) {
          for (let i = 0; i < file.length; i++) {
            if (file[i].isFile) {
              _this.foldersAdd(file[i]);
            } else {
              _this.folders(file[i]);
            }
          }
        })
      },
      foldersAdd(entry) {
        let _this = this;
        entry.file(function (file) {
          _this.fileAdd(file)
        })
      },
      fileAdd(file) {
        if (this.limit !== undefined) this.limit--;
        if (this.limit !== undefined && this.limit < 0) return;
        //总大小
        this.size = this.size + file.size;
        //判断是否为图片文件
        if (file.type.indexOf('image') == -1) {
          file.src = 'wenjian.png';
          this.imgList.push({
            file
          });
        } else {
          let reader = new FileReader();
          let image = new Image();
          let _this=this;
          reader.readAsDataURL(file);
          reader.onload = function () {
            file.src = this.result;
            image.onload=function(){
              let width = image.width;
              let height = image.height;
              file.width=width;
              file.height=height;
              _this.imgList.push({
                file
              });
              console.log( _this.imgList);
            };
            image.src= file.src;
          }
        }
      },
      fileDel(index) {
        this.size = this.size - this.imgList[index].file.size;//总大小
        this.imgList.splice(index, 1);
        if (this.limit !== undefined) this.limit = this.imgList.length;
      },
      bytesToSize(bytes) {
        if (bytes === 0) return '0 B';
        let k = 1000, // or 1024
          sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
          i = Math.floor(Math.log(bytes) / Math.log(k));
        return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i];
      },
      dragenter(el) {
        el.stopPropagation();
        el.preventDefault();
      },
      dragover(el) {
        el.stopPropagation();
        el.preventDefault();
      },
      drop(el) {
        el.stopPropagation();
        el.preventDefault();
        this.fileList(el.dataTransfer);
      }
    }
  }
</script>
<style scoped>
  .upload_warp_img_div_del {
    position: absolute;
    top: 6px;
    width: 16px;
    right: 4px;
  }

  .upload_warp_img_div_top {
    position: absolute;
    top: 0;
    width: 100%;
    height: 30px;
    background-color: rgba(0, 0, 0, 0.4);
    line-height: 30px;
    text-align: left;
    color: #fff;
    font-size: 12px;
    text-indent: 4px;
  }

  .upload_warp_img_div_text {
    white-space: nowrap;
    width: 80%;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .upload_warp_img_div img {
    max-width: 100%;
    max-height: 100%;
    vertical-align: middle;
  }

  .upload_warp_img_div {
    position: relative;
    height: 100px;
    width: 120px;
    border: 1px solid #ccc;
    margin: 0px 30px 10px 0px;
    float: left;
    line-height: 100px;
    display: table-cell;
    text-align: center;
    background-color: #eee;
    cursor: pointer;
  }

  .upload_warp_img {
    border-top: 1px solid #D2D2D2;
    padding: 14px 0 0 14px;
    overflow: hidden
  }

  .upload_warp_text {
    text-align: left;
    margin-bottom: 10px;
    padding-top: 10px;
    text-indent: 14px;
    border-top: 1px solid #ccc;
    font-size: 14px;
  }

  .upload_warp_right {
    float: left;
    width: 57%;
    margin-left: 2%;
    height: 100%;
    border: 1px dashed #999;
    border-radius: 4px;
    line-height: 130px;
    color: #999;
  }

  .upload_warp_left img {
    margin-top: 32px;
  }

  .upload_warp_left {
    float: left;
    width: 40%;
    height: 100%;
    border: 1px dashed #999;
    border-radius: 4px;
    cursor: pointer;
  }

  .upload_warp {
    margin: 14px;
    height: 130px;
  }

  .upload {
    border: 1px solid #ccc;
    background-color: #fff;
    width: 650px;
    box-shadow: 0px 1px 0px #ccc;
    border-radius: 4px;
  }

  .hello {
    width: 650px;
    margin-left: 34%;
  }
</style>
