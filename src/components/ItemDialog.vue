<template>
<!-- データ追加／編集ダイアログ -->
<v-dialog v-model="show" scrollable persistent max-width="500px" eager>
  <v-card>
    <v-card-title>{{ titleText }}</v-card-title>
    <v-divider />
    <v-card-text>
      <v-form ref="form" v-model="valid">
        <!-- 日付選択 -->
        <v-menu ref="menu" v-model="menu" :close-on-content-click="false" :return-value.sync="date" transition="scale-transition" offset-y max-width="290px" min-width="290px">
          <template v-slot:activator="{ on }">
            <v-text-field v-model="date" prepend-icon="mdi-calendar" readonly v-on="on" hide-details />
          </template>
          <v-date-picker v-model="date" color="green" locale="ja-jp" :day-format="date => new Date(date).getDate()" no-title scrollable>
            <v-spacer />
            <v-btn text color="grey" @click="menu = false">キャンセル</v-btn>
            <v-btn text color="primary" @click="$refs.menu.save(date)">選択</v-btn>
          </v-date-picker>
        </v-menu>
        <!-- 支払者 -->
        <v-radio-group row v-model="inout" hide-details @change="onChangeInout">
          <v-radio label="悠太" value="yuta" />
          <v-radio label="佳菜子" value="kana" />
        </v-radio-group>
        <!-- カテゴリ -->
        <v-select label="カテゴリ" v-model="category" :items="categoryItems" hide-details />
        <!-- 金額 -->
        <v-text-field label="金額" v-model.number="amount" type="number" prefix="￥" pattern="[0-9]*" :rules="amountRules" />
        <!-- メモ -->
        <v-text-field label="メモ" v-model="memo" :counter="50" :rules="[memoRule]" />
      </v-form>
    </v-card-text>
    <v-divider />
    <v-card-actions>
      <v-spacer />
      <v-btn color="grey darken-1" text :disabled="loading" @click="onClickClose">
        キャンセル
      </v-btn>
      <v-btn color="blue darken-1" text :disabled="!valid" :loading="loading" @click="onClickAction">
        {{ actionText }}
      </v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
</template>

<script>
import {
  mapActions,
  mapGetters,
  mapState
} from 'vuex'

export default {
  name: 'ItemDialog',

  data() {
    return {
      /** ダイアログの表示状態 */
      show: false,
      /** 入力したデータが有効かどうか */
      valid: false,
      /** 日付選択メニューの表示状態 */
      menu: false,

      /** 操作タイプ 'add' or 'edit' */
      actionType: 'add',
      /** id */
      id: '',
      /** 日付 */
      date: '',
      /** 支払者 'yuta' or 'kana' */
      inout: '',
      /** カテゴリ */
      category: '',
      /** 金額 */
      amount: 0,
      /** メモ */
      memo: '',

      /** 編集前の年月（編集時に使う） */
      beforeYM: '',

      /** バリデーションルール */
      amountRules: [
        v => v >= 0 || '金額は0以上で入力してください',
        v => Number.isInteger(v) || '整数で入力してください'
      ],
      memoRule: v => v.length <= 50 || 'メモは50文字以内で入力してください'
    }
  },

  computed: {
    ...mapGetters([
        /** カテゴリ */
        'categoryItems',
  ]),
        ...mapState({
          /** ローディング状態 */
          loading: state => state.loading.add || state.loading.update
        }),

        /** ダイアログのタイトル */
        titleText() {
          return this.actionType === 'add' ? 'データ追加' : 'データ編集'
        },
        /** ダイアログのアクション */
        actionText() {
          return this.actionType === 'add' ? '追加' : '更新'
        }
      },

      methods: {
        ...mapActions([
          /** データ追加 */
          'addAbData',
          /** データ更新 */
          'updateAbData'
        ]),

        /**
         * ダイアログを表示します。
         * このメソッドは親から呼び出されます。
         */
        open(actionType, item) {
          this.show = true
          this.actionType = actionType
          this.resetForm(item)

          if (actionType === 'edit') {
            this.beforeYM = item.date.slice(0, 7)
          }
        },
        /** キャンセルがクリックされたとき */
        onClickClose() {
          this.show = false
        },

        /** 追加／更新がクリックされたとき */
        async onClickAction() {
          const item = {
            date: this.date,
            category: this.category,
            yuta: null,
            kana: null,
            memo: this.memo
          }
          item[this.inout] = this.amount || 0

          if (this.actionType === 'add') {
            await this.addAbData({
              item
            })
          } else {
            item.id = this.id
            await this.updateAbData({
              beforeYM: this.beforeYM,
              item
            })
          }

          this.show = false
        },

        /** フォームの内容を初期化します */
        resetForm(item = {}) {
          const today = new Date()
          const year = today.getFullYear()
          const month = ('0' + (today.getMonth() + 1)).slice(-2)
          const date = ('0' + today.getDate()).slice(-2)

          this.id = item.id || ''
          this.date = item.date || `${year}-${month}-${date}`
          this.inout = item.yuta != null ? 'yuta' : 'kana'

          if (this.inout === 'yuta') {
            this.amount = item.yuta || 0
          } else {
            this.amount = item.kana || 0
          }

          this.category = item.category || this.categoryItems[0]
          this.memo = item.memo || ''

          this.$refs.form.resetValidation()
        }
      }
    }
</script>
