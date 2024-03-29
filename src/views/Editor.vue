<template>
  <div class="ng-scope" v-if="!isLoading">
    <div class="editor-page ng-scope">
      <div class="container page">
        <div class="row">
          <div class="col-md-10 offset-md-1 col-xs-12">
            <validate-errors
              v-if="validateErrors"
              :validateErrors="validateErrors"
            />

            <form class="ng-pristine ng-valid" @submit.prevent="submit">
              <fieldset>
                <fieldset class="form-group">
                  <input
                    class="form-control form-control-lg ng-pristine ng-untouched ng-valid ng-empty"
                    type="text"
                    placeholder="Article Title"
                    v-model="title"
                    :disabled="isSubmitting"
                  />
                </fieldset>

                <fieldset class="form-group">
                  <input
                    class="form-control ng-pristine ng-untouched ng-valid ng-empty"
                    type="text"
                    placeholder="What's this article about?"
                    v-model="description"
                    :disabled="isSubmitting"
                  />
                </fieldset>

                <fieldset class="form-group">
                  <textarea
                    class="form-control ng-pristine ng-untouched ng-valid ng-empty"
                    rows="8"
                    placeholder="Write your article (in markdown)"
                    v-model="body"
                    :disabled="isSubmitting"
                  >
                  </textarea>
                </fieldset>

                <fieldset class="form-group">
                  <input
                    class="form-control ng-pristine ng-untouched ng-valid ng-empty"
                    type="text"
                    placeholder="Enter tags"
                    v-model="tag"
                    :disabled="isSubmitting"
                    @keypress.enter.prevent="addTag"
                  />

                  <div class="tag-list">
                    <span
                      class="tag-default tag-pill ng-binding ng-scope"
                      v-for="tagName in tagList"
                      :key="tagName"
                    >
                      <i
                        class="ion-close-round"
                        @click="removeTag(tagName)"
                      ></i>
                      {{ tagName }}
                    </span>
                  </div>
                </fieldset>

                <button
                  class="btn btn-lg pull-xs-right btn-primary"
                  type="submit"
                  :disabled="isSubmitting"
                >
                  Publish Article
                </button>
              </fieldset>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ValidateErrors from "../components/ValidateErrors.vue";
import { getArticle, postArticle, updateArticle } from "@/api/api";

export default {
  name: "AppEditor",
  components: { ValidateErrors },
  data() {
    return {
      body: "",
      description: "",
      tag: "",
      title: "",

      tagList: [],

      isSubmitting: false,
      validateErrors: null,

      isLoading: true
    };
  },
  created() {
    this.$store.dispatch("getCurrentUser").then(() => {
      if (this.currentUser === null) {
        this.$router.push({ name: "home" });
      } else {
        if (this.$route.params.slug) {
          getArticle(this.$route.params.slug)
            .then(article => {
              this.body = article.body;
              this.description = article.description;
              this.tagList = article.tagList;
              this.title = article.title;

              this.isLoading = false;
            })
            .catch(() => {
              this.$router.push({ name: "home" });
            });
        } else {
          this.isLoading = false;
        }
      }
    });
  },
  methods: {
    addTag() {
      if (this.tagList.indexOf(this.tag) === -1) {
        this.tagList.push(this.tag);
        this.tag = "";
      }
    },
    removeTag(tagName) {
      this.tagList = this.tagList.filter(i => i !== tagName);
    },
    submit() {
      this.isSubmitting = true;
      let promise;
      if (this.$route.params.slug) {
        promise = updateArticle(
          this.$route.params.slug,
          this.body,
          this.description,
          this.tagList,
          this.title
        );
      } else {
        promise = postArticle(
          this.body,
          this.description,
          this.tagList,
          this.title
        );
      }
      promise
        .then(article => {
          this.$router.push({
            name: "article",
            params: { slug: article.slug }
          });
        })
        .catch(errors => {
          this.validateErrors = errors;
          this.isSubmitting = false;
        });
    }
  }
};
</script>

<style></style>
