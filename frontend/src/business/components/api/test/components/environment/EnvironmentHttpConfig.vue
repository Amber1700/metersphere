<template>
    <el-form :model="httpConfig" :rules="rules" ref="httpConfig">
      <span>{{$t('api_test.environment.socket')}}</span>
      <el-form-item prop="socket">
        <el-input v-model="httpConfig.socket" :placeholder="$t('api_test.request.url_description')" clearable>
          <template v-slot:prepend>
            <el-select v-model="httpConfig.protocol" class="request-protocol-select">
              <el-option label="http://" value="http"/>
              <el-option label="https://" value="https"/>
            </el-select>
          </template>
        </el-input>
      </el-form-item>

      <span>{{$t('api_test.request.headers')}}</span>
      <ms-api-key-value :items="httpConfig.headers" :isShowEnable="true" :suggestions="headerSuggestions"/>
    </el-form>
</template>

<script>
    import {HttpConfig} from "../../model/EnvironmentModel";
    import MsApiKeyValue from "../ApiKeyValue";
    import {REQUEST_HEADERS} from "../../../../../../common/js/constants";

    export default {
      name: "MsEnvironmentHttpConfig",
      components: {MsApiKeyValue},
      props: {
        httpConfig: new HttpConfig(),
      },
      data() {
        let socketValidator = (rule, value, callback) => {
          if (!this.validateSocket(value)) {
            callback(new Error(this.$t('commons.formatErr')));
            return false;
          } else {
            callback();
            return true;
          }
        }
        return {
          headerSuggestions: REQUEST_HEADERS,
          rules: {
            socket: [{required: false, validator: socketValidator, trigger: 'blur'}],
          },
        }
      },
      methods: {
        validateSocket(socket) {
          if (!socket) return true;
          let urlStr = this.httpConfig.protocol + '://' + socket;
          let url = {};
          try {
            url = new URL(urlStr);
          } catch (e) {
            return false;
          }
          this.httpConfig.domain = decodeURIComponent(url.hostname);

          this.httpConfig.port = url.port;
          if (url.port) {
            this.httpConfig.socket = this.httpConfig.domain + ':' + url.port + url.pathname;
          } else {
            this.httpConfig.socket = this.httpConfig.domain + url.pathname;
          }
          return true;
        },
        validate() {
          let isValidate = false;
          this.$refs['httpConfig'].validate((valid) => {
            isValidate = valid;
          });
          return isValidate;
        }
      }
    }
</script>

<style scoped>

  .request-protocol-select {
    width: 90px;
  }

</style>