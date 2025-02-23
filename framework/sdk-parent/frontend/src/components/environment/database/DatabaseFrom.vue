<template>
  <div class="database-from" v-loading="result.loading">
    <el-form :model="currentConfig" :rules="rules" label-width="150px" size="small" :disabled="isReadOnly" ref="databaseFrom">

      <el-form-item :label="$t('api_test.request.sql.dataSource')" prop="name">
        <el-input v-model="currentConfig.name" maxlength="300" show-word-limit
                  :placeholder="$t('commons.input_content')"/>
      </el-form-item>

      <el-form-item :label="$t('api_test.request.sql.database_driver')" prop="driver">
        <el-select v-model="currentConfig.driver" class="select-100" @change="driverChange" clearable>
          <el-option v-for="p in drivers" :key="p" :label="p" :value="p"/>
        </el-select>
      </el-form-item>

      <el-form-item :label="$t('api_test.request.sql.database_url')" prop="dbUrl">
        <el-input v-model="currentConfig.dbUrl" maxlength="500" show-word-limit
                  :placeholder="$t('commons.input_content')"/>
        <div v-if="currentConfig.driver ==='com.mysql.jdbc.Driver'">
          <span style="font-size:10px">{{this.$t('api_test.request.sql.tips')}}</span>
        </div>
      </el-form-item>

      <el-form-item :label="$t('api_test.request.sql.username')" prop="username">
        <el-input v-model="currentConfig.username" maxlength="300" show-word-limit
                  :placeholder="$t('commons.input_content')"/>
      </el-form-item>

      <el-form-item :label="$t('api_test.request.sql.password')" prop="password">
        <el-input v-model="currentConfig.password" type="password" autocomplete="new-password" maxlength="200"
                  :placeholder="$t('commons.input_content')"/>
      </el-form-item>

      <el-form-item :label="$t('api_test.request.sql.pool_max')" prop="poolMax">
        <el-input-number size="small" :disabled="isReadOnly" v-model="currentConfig.poolMax"
                         :placeholder="$t('commons.please_select')" :max="100" :min="0"
                         onKeypress="return (/[\d]/.test(String.fromCharCode(event.keyCode)))"
                         :precision="0"/>
      </el-form-item>


      <el-form-item :label="$t('api_test.request.sql.timeout')" prop="timeout">
        <el-input-number size="small" :disabled="isReadOnly" v-model="currentConfig.timeout"
                         :placeholder="$t('commons.millisecond')" :max="1000*10000000" :min="0"
                         onKeypress="return (/[\d]/.test(String.fromCharCode(event.keyCode)))"
                         :precision="0"/>
      </el-form-item>

      <el-form-item>
        <div class="buttons">
          <el-button type="primary" size="small" @click="validate">{{$t('commons.validate')}}</el-button>
          <el-button type="primary" v-show="currentConfig.id" size="small" @click="save('update')">{{$t('commons.update')}}</el-button>
          <el-button type="primary" v-show="currentConfig.id" size="small" @click="clear">{{$t('commons.clear')}}</el-button>
          <el-button type="primary" v-show="!currentConfig.id"  size="small" @click="save('add')">{{$t('commons.add')}}</el-button>
        </div>
      </el-form-item>

    </el-form>

  </div>
</template>

<script>
import {databaseValidate} from "../../../api/environment";
import {DatabaseConfig} from "../../../model/EnvTestModel";

export default {
  name: "MsDatabaseFrom",
  components: {},
  props: {
    isReadOnly: {
      type: Boolean,
      default: false
    },
    config: {
      type: Object,
      default() {
        return new DatabaseConfig();
      }
    },
    callback: {
      type: Function
    },
  },
  watch: {
    config() {
      Object.assign(this.currentConfig, this.config);
    }
  },
  mounted() {
    Object.assign(this.currentConfig, this.config);
  },
  data() {
    return {
      drivers: DatabaseConfig.DRIVER_CLASS,
      result: false,
      currentConfig: new DatabaseConfig(),
      rules: {
        name: [
          {required: true, message: this.$t('commons.input_name'), trigger: 'blur'},
          {max: 300, message: this.$t('commons.input_limit', [0, 300]), trigger: 'blur'}
        ],
        driver: [
          {required: true, message: this.$t('commons.cannot_be_null'), trigger: ['change','blur']},
        ],
        password: [
          {max: 200, message: this.$t('commons.input_limit', [0, 200]), trigger: 'blur'}
        ],
        dbUrl: [
          {required: true, message: this.$t('commons.cannot_be_null'), trigger: 'blur'},
          {max: 500, message: this.$t('commons.input_limit', [0, 500]), trigger: 'blur'}
        ],
        username: [
          {required: true, message: this.$t('commons.cannot_be_null'), trigger: 'blur'},
          {max: 200, message: this.$t('commons.input_limit', [0, 200]), trigger: 'blur'}
        ],
        poolMax: [
          {required: true, message: this.$t('commons.cannot_be_null'), trigger: 'blur'},
        ],
        timeout: [
          {required: true, message: this.$t('commons.cannot_be_null'), trigger: 'blur'},
        ]
      }
    }
  },
  methods: {
    save(type) {
      this.$refs['databaseFrom'].validate((valid) => {
        if (valid) {
          if (this.callback) {
            if (type === 'add') {
              this.currentConfig.id = undefined;
            }
            this.callback(this.currentConfig);
          }
        } else {
          return false;
        }
      });
    },
    clear() {
      this.currentConfig = new DatabaseConfig();
    },
    validate() {
      this.result = databaseValidate(this.currentConfig).then(() => {
        this.$success(this.$t('commons.connection_successful'));
      })
    },
    driverChange(type){
      this.currentConfig.dbUrl = "";
      if(type === "com.mysql.jdbc.Driver"){
        this.currentConfig.dbUrl = "jdbc:mysql://127.0.0.1:3306/database";
      }else if(type === "com.microsoft.sqlserver.jdbc.SQLServerDriver"){
        this.currentConfig.dbUrl = "jdbc:sqlserver://127.0.0.1:1433;DatabaseName=database";
      }else if(type === "org.postgresql.Driver"){
        this.currentConfig.dbUrl = "jdbc:postgresql://127.0.0.1:5432/database";
      }else if(type === "oracle.jdbc.OracleDriver"){
        this.currentConfig.dbUrl = "jdbc:oracle:thin:@192.168.2.1:1521:database";
      }
    },
  }
}
</script>

<style scoped>

.buttons {
  float: right;
}

.select-100 {
  width: 100%;
}

</style>
