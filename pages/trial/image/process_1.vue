<style lang="scss">
#trial-image-process-1 {
  .image {
    margin-bottom: 20px;
  }

  .name {
    margin-bottom: 10px;

    .tag-container {
      .el-tag {
        margin-right: 5px;
      }
    }

    a {
      margin-right: 8px;
    }
  }
}
</style>

<template>
  <div
    v-loading="loading"
    id="trial-image-process-1"
  >
    <div>
      <el-col
        v-for="(image, index) in list"
        :key="image.id"
        :span="7"
        :offset="1"
        class="image"
      >
        <el-card :body-style="{ padding: '0px' }">
          <a
            :href="$resize(image.url)"
            target="_blank"
          >
            <img :src="$resize(image.url, { width: 500, mode: 2 })">
          </a>
          <div style="padding: 14px;">
            <div class="name">
              <a
                :href="computeAlbumHref(image)"
                target="_blank"
              >
                <div class="tag-container">
                  <el-tag>查看相册</el-tag>
                  <a
                    v-if="image.bangumi_id"
                    :href="$alias.bangumi(image.bangumi_id, 'pins')"
                    target="_blank"
                  >
                    <el-tag>查看番剧</el-tag>
                  </a>
                  <a
                    v-else
                    :href="$alias.image(image.album_id)"
                    target="_blank"
                  >
                    <el-tag>查看相册</el-tag>
                  </a>
                  <el-tag
                    v-if="image.deleted_at"
                  >{{ image.user_id == image.state ? '系统删除' : '举报删除' }}</el-tag>
                </div>
              </a>
              <span
                v-if="image.name"
                v-text="image.name"
              />
            </div>
            <template v-if="image.deleted_at">
              <el-button
                type="danger"
                size="mini"
                @click="approveImage(image, index)"
              >确认删除</el-button>
              <el-button
                type="success"
                size="mini"
                @click="rejectImage(image, index)"
              >恢复图片</el-button>
            </template>
            <template v-else>
              <el-button
                type="success"
                size="mini"
                @click="passImage(image, index)"
              >通过</el-button>
              <el-button
                type="danger"
                size="mini"
                @click="deleteImage(image, index)"
              >删除</el-button>
            </template>
            <router-link
              :to="`/admin/user/show?id=${image.user_id}`"
              style="margin-left: 10px"
            >
              <el-button
                type="primary"
                size="mini"
              >查看用户</el-button>
            </router-link>
          </div>
        </el-card>
      </el-col>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      loading: true
    }
  },
  created() {
    this.getData()
  },
  methods: {
    computeAlbumHref(image) {
      if (image.bangumi_id) {
        return this.$alias.image(image.id)
      }
      return this.$alias.image(image.album_id)
    },
    getData() {
      this.$axios
        .$get('admin/trial/image/list')
        .then(data => {
          this.list = data
          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    },
    deleteImage(image, index) {
      this.$confirm('确定要删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$axios
            .$post('admin/trial/image/ban', {
              id: image.id,
              type: image.bangumi_id ? 'album' : 'image'
            })
            .then(() => {
              this.list.splice(index, 1)
            })
        })
        .catch(() => {})
    },
    passImage(image, index) {
      this.$axios
        .$post('admin/trial/image/pass', {
          id: image.id,
          type: image.bangumi_id ? 'album' : 'image'
        })
        .then(() => {
          this.list.splice(index, 1)
        })
    },
    approveImage(image, index) {
      this.$axios
        .$post('admin/trial/image/approve', {
          id: image.id,
          type: image.bangumi_id ? 'album' : 'image'
        })
        .then(() => {
          this.list.splice(index, 1)
        })
    },
    rejectImage(image, index) {
      this.$axios
        .$post('admin/trial/image/reject', {
          id: image.id,
          type: image.bangumi_id ? 'album' : 'image'
        })
        .then(() => {
          this.list.splice(index, 1)
        })
    }
  }
}
</script>
