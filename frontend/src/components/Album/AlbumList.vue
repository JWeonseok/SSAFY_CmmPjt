<template>
  <div style="background-color: rgba(240, 241, 247); height: 100%">
    <div
      class="mx-auto d-flex justify-content-end"
      style="width: 57vw; margin-bottom: 10px"
    >
      <template v-if="$store.state.user.type === 2">
        <button data-bs-toggle="modal" data-bs-target="#profileModal">
          <img
            src="@/assets/flaticon/upload.png"
            class="file-uploader-btn"
            alt="upload-button"
          />
        </button>
      </template>
      <div style="height: 67.2px" v-else></div>
    </div>

    <div
      class="
        container
        d-flex
        flex-wrap
        justify-content-center
        content-container
        align-content-between
      "
      style="
        height: 75vh;
        width: 57vw;
        margin: 0 auto;
        border-radius: 8px;
        background-color: white;
      "
    >
      <div class="row row-cols-5">
        <div v-for="album in albumList" :key="album.albumId" cols="3">
          <div class="thumbnailWrap">
            <img
              :src="
                'https://ssafy-cmmpjt304.s3.ap-northeast-2.amazonaws.com/' +
                album.thumbnailUrl
              "
              @click="setDetail(album.albumId)"
              class="thumbnailImg"
            />
          </div>
          <div
            class="m-0"
            style="text-align: right; font-size:0.9rem; letter-spacing:-1; font-weight: 600; width: 91%"
          >
            {{ album.title }}
            <p style="font-size: 0.8rem; letter-spacing:-1;">
              {{ album.createDate2 }}
            </p>
          </div>
        </div>
      </div>

      <div class="more-btn">
        <button
          class="mt-2"
          @click="getMoreThumbnail"
          v-if="albumList.length > 0 && pageNum < pageCnt"
        >
          더보기
        </button>
        <p class="mb-0 mt-2" v-else>불러올 글이 없습니다</p>
        <AlbumDetail
          v-if="visible"
          :visible="visible"
          :selectedAlbumId="selectedAlbumId"
          @updateVisible="updateVisible"
          @get-album="reGetAlbumthumbnail"
        />
      </div>
    </div>

    <!-- 모달 -->
    <div style="width: 95%">
      <div
        class="modal fade"
        id="profileModal"
        tabindex="-1"
        aria-labelledby="profileModalLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header" style="background-color: #a8b1cf">
              <h5
                class="modal-title"
                style="color: rgb(256, 256, 256)"
                id="profileModalLabel"
              >
                사진 업로드
              </h5>
              <button
                type="button"
                class="btn-close"
                data-bs-dismiss="modal"
                aria-label="Close"
                @click="resetDate"
              ></button>
            </div>
            <div class="modal-body">
              <div class="d-flex justify-content-end align-items-center mt-5">
                <p style="margin: 0; margin-right: 2.3px">제목</p>
                <input
                  type="text"
                  id="title"
                  class="formInput me-0"
                  placeholder="10글자 이내"
                  maxlength="10"
                  v-model="title"
                />
              </div>

              <v-file-input
                id="photoupload"
                multiple
                filled
                ref="file-input"
                prepend-icon="mdi-camera"
                dense
                v-model="filename"
                @change="fileSelect"
              ></v-file-input>

            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn"
                style="background-color: #a8b1cf; border-radius: 40px"
                @click="uploadZip"
                data-bs-dismiss="modal"
                :disabled="!fileSelectFlag || !title"
              >
                사진 업로드
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AlbumDetail from "@/components/Album/AlbumDetail";
import albumApi from "@/api/album.js";
import awss3 from "@/utils/awss3.js";

export default {
  name: "AlbumList",
  components: {
    AlbumDetail,
  },
  data() {
    return {
      title: "",
      photoList: [],
      ZipUrl: "",

      pageNum: 0,
      pageCnt: 0,

      albumList: [],
      fileSelectFlag: 0,
      visible: false,
      selectedAlbumId: 0,
      filename: null,
    };
  },
  methods: {
    resetDate() {
      this.fileSelectFlag = 0;
      this.title = "";
    },

    fileSelect() {
      if (this.filename.length) {
        this.fileSelectFlag = 1;
      } else {
        this.fileSelectFlag = 0;
      }
    },

    reGetAlbumthumbnail() {
      this.pageNum = 0;
      this.albumList = [];
      this.getMoreThumbnail();
    },
    getMoreThumbnail() {
      if (this.pageNum <= this.pageCnt) {
        this.getAlbumthumbnail();
      }
    },

    // 썸네일 조회
    async getAlbumthumbnail() {
      let accessToken = sessionStorage.getItem("access-token");
      let refreshToken = sessionStorage.getItem("refresh-token");
      let classCode = this.$store.state.user.classCode;
      let data = {
        classCode: classCode,
        pageNum: this.pageNum,
      };
      let result = await albumApi.getAlbumthumbnail(data, {
        "access-token": accessToken,
        "refresh-token": refreshToken,
      });
      this.pageCnt = result.pageCnt;
      this.albumList.push(...result.thumbnailList);
      this.pageNum += 1;
    },

    // 사진 업로드
    async uploadZip() {
      this.title = document.getElementById("title").value;
      var list1 = await awss3.uploadPhoto("album", "photoupload");
      var list2 = awss3.uploadZip();

      this.photoList = list1;
      this.ZipUrl = list2;
      let classCode = this.$store.state.user.classCode;
      let writerId = this.$store.state.user.userId;
      let accessToken = sessionStorage.getItem("access-token");
      let refreshToken = sessionStorage.getItem("refresh-token");
      let data = {
        title: this.title,
        classCode: classCode,
        writerId: writerId,
        photoList: this.photoList,
        downloadUrl: this.ZipUrl,
      };
      await albumApi
        .createAlbum(data, {
          "access-token": accessToken,
          "refresh-token": refreshToken,
        })
        .then(() => {})
        .catch(() => {
          this.$fire({
            html: `<a href="javascript:void(0);"></a><p style="font-size: 0.95rem; font-family: 'NanumSquareRound';">사진 업로드에 실패했습니다.</p>
            <p style="font-size: 0.95rem; font-family: 'NanumSquareRound';">다시 시도해주세요.</p>`,
            focusConfirm: false,
            confirmButtonColor: "#58679A",
            type: "error",
          });
        });
      this.title = "";
      this.$refs['file-input'].reset()
      this.fileSelectFlag = 0;
      this.pageNum = 0;
      this.albumList = [];
      this.getAlbumthumbnail();
    },

    setDetail(n) {
      this.selectedAlbumId = n;
      this.updateVisible();
    },
    updateVisible() {
      this.visible = !this.visible;
    },
  },
  created() {
    this.getAlbumthumbnail();
  },
};
</script>

<style scoped>
* {
  font-family: "NanumSquareRound";
  font-size: 0.95rem;
}
.container {
  max-width: 93%;
}

/* 버튼 */
.file-uploader-btn {
  height: 2.5rem;
  width: 2.5rem;
  margin-top: 1.7rem;
  cursor: pointer;
}
.more-btn {
  color: rgb(156, 156, 156);
  background-color: rgba(102, 122, 188, 0.05);
  font-size: 0.8rem;
  border-radius: 5px;
  width: 100%;
  height: 2rem;
  text-align: center;
}

/* 썸네일 */
.thumbnailImg {
  width: 100%;
  height: 15vh;
  margin-top: 1rem;
  margin-bottom: 0.4rem;
  transition-duration: 1s;
  object-fit: cover;
}
.thumbnailImg:hover {
  transition: 0.4s;
  transform: scale(1.06, 1.06);
  transition-duration: 0.5s;
}

.thumbnailWrap {
  overflow: hidden;
}

/* 스크롤 */
.content-container {
  overflow-y: scroll;
  height: 80vh;
}
.content-container::-webkit-scrollbar {
  width: 7px;
  background-color: rgba(233, 234, 239, 0.5);
  border-radius: 5px;
}
.content-container::-webkit-scrollbar-thumb {
  background-color: #a8b1cf;
  border-radius: 5px;
}
.content-container::-webkit-scrollbar-track {
  background-color: rgba(233, 234, 239, 0.5);
  border-radius: 5px;
}

/* 모달 */
.btnstyle {
  border: 2px solid rgb(196, 196, 196);
  color: gray;
  background-color: white;
  padding: 8px 20px;
  border-radius: 10px;
  font-size: 20px;
  font-weight: bold;
}
.upload-btn-wrapper input[type="file"] {
  font-size: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
}
.upload-btn-wrapper {
  position: relative;
  overflow: hidden;
  display: inline-block;
}

/* v-button을 맞추기 위한 커스텀 */
.formInput {
  background-color: rgba(156, 156, 156, 0.1);
  border-radius: 2px;
  height: 40px;
  width: 93%;
  padding: 0px 0px 0px 15px;
  margin: 3px 3px 3px 3px;
  border-bottom: solid 1.6px rgba(156, 156, 156, 0.85);
}
.formInput:hover {
  background-color: rgba(156, 156, 156, 0.35);
  border-bottom: solid 1.5px rgba(40, 40, 40, 0.8);
}
</style>