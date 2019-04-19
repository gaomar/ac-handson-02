---
id: dist
{{if .Meta.Status}}status: {{.Meta.Status}}{{end}}
{{if .Meta.Summary}}summary: {{.Meta.Summary}}{{end}}
{{if .Meta.Author}}author: {{.Meta.Author}}{{end}}
{{if .Meta.Categories}}categories: {{commaSep .Meta.Categories}}{{end}}
{{if .Meta.Tags}}tags: {{commaSep .Meta.Tags}}{{end}}
{{if .Meta.Feedback}}feedback link: {{.Meta.Feedback}}{{end}}
{{if .Meta.GA}}analytics account: {{.Meta.GA}}{{end}}

---

# Amazon Connectハンズオン 02

## Lambda関数を作ろう！
### 1-1. Lambda関数を作成する
サービス部分をクリックしてメニューを展開します。そこの検索窓に「Lambda」と入力します。

![s200](images/s200.png)

［関数の作成］ボタンをクリックします。

![s201](images/s201.png)

各項目を埋めて、［関数の作成］ボタンをクリックします。

| 項目       |       値 |
|:-----------------|:------------------|
|①関数名|AmazonConnect-BMI|
|②実行ロール|AWSポリシーテンプレートから新しいロールを作成|
|③ロール名|AmazonConnect-Role|
|④ポリシーテンプレート|シンプルなマイクロサービスのアクセス権限|

![s202](images/s202.png)

index.jsを全て下記に書き換えます。書き換えたら右上の［保存］ボタンをクリックします。

![s203](images/s203.png)

```javascript:index.js
exports.handler = async (event) => {
    // 身長と体重を取得する
    const heightVal = event.Details.ContactData.Attributes.HeightVal;
    const weightVal = event.Details.ContactData.Attributes.WeightVal;
    
    // BMI計算
    const bmiVal = (parseFloat(weightVal) / (parseFloat(heightVal)/100 * parseFloat(heightVal)/100)).toFixed(1);

    // 標準体重
    const stdWeight = (22 * (parseFloat(heightVal)/100 * parseFloat(heightVal)/100)).toFixed(1);

    var speechText = `あなたのBMIは${bmiVal}です。標準体重は${stdWeight}kgです。`;

    return {"BMI": speechText};
};
```

## LambdaをAmazon Connectに適用しよう！
### 2-1. LambdaをAmazon Connectに適用する
サービスを展開して、検索窓に「Amazon Connect」と入力してクリックします。

![s204](images/s204.png)

左側メニューから「問い合わせフロー」をクリックします。

![s205](images/s205.png)

AWS Lambdaの項目までスクロールして、関数のプルダウンメニューから「AmazonConnect-BMI」の関数を選択します。
選択したら、［追加］ボタンをクリックします。

![s206](images/s206.png)

左側メニューから「概要」をクリックします。［管理者としてログイン］をクリックします。

![s207](images/s207.png)

### 2-2.問い合わせフローの作成
［問い合わせフローの作成］をクリックします。

![s208](images/s208.png)

名前を「BMIフロー」と入力します。

![s209](images/s209.png)

設定カテゴリにある「音声の設定」ブロックをドラッグアンドドロップして、ドロップしたブロックをクリックします。

![s210](images/s210.png)

言語は「日本語」でお好きな音声を選択してください。

![s211](images/s211.png)

エントリポイントと音声の設定ブロックを繋げます。

![s212](images/s212.png)

操作カテゴリの「顧客の入力を保存する」をドラッグアンドドロップしてクリックします。

![s213](images/s213.png)

「テキストの読み上げ」を選択し、発話する内容を入力します。身長の桁数は3桁なので、最大桁数は3桁に設定します。

![s214](images/s214.png)

ブロックを繋げます。

![s215](images/s215.png)

設定カテゴリにある「問い合わせ属性の設定」をドラッグアンドドロップします。

![s216](images/s216.png)

「属性を使用する」を選択し、項目を埋めていきます。
宛先キーは大文字小文字に気をつけてください。

| 項目       |       値 |
|:-----------------|:------------------|
|宛先キー|HeightVal　※大文字小文字は一致させてください|
|タイプ|システム|
|属性|保存済みのお客様の入力|

![s217](images/s217.png)

ブロックを繋げます。

![s218](images/s218.png)

操作カテゴリの「顧客の入力を保存する」をドラッグアンドドロップします。

![s219](images/s219.png)

「テキストの読み上げ機能」を選択し、発話する内容を入力します。体重の最大桁数は3桁にします。

![s220](images/s220.png)

ブロックを繋げます。

![s221](images/s221.png)

設定カテゴリにある「問い合わせ属性の設定」をドラッグアンドドロップします。

![s222](images/s222.png)

「属性を使用する」を選択し、項目を埋めていきます。
宛先キーは大文字小文字に気をつけてください。

| 項目       |       値 |
|:-----------------|:------------------|
|宛先キー|WeightVal　※大文字小文字は一致させてください|
|タイプ|システム|
|属性|保存済みのお客様の入力|

![s223](images/s223.png)

ブロックを繋げます。

![s224](images/s224.png)

統合カテゴリにある「AWS Lambda 関数を呼び出す」をドラッグアンドドロップします。

![s225](images/s225.png)

関数は先程作成した「AmazonConnect-BMI」を選択します。

![s226](images/s226.png)

ブロックを繋げます。

![s227](images/s227.png)

操作カテゴリの「プロンプトの再生」をドラッグアンドドロップします。

![s228](images/s228.png)

「テキストの読み上げ機能」を選択し、下記コードを入力します。
Lambdaから帰ってくるbodyは「$.External」に格納されます。

```
$.External.BMI
```

![s229](images/s229.png)

ブロックを繋げます。

![s230](images/s230.png)

終了カテゴリーの「切断/ハングアップ」をドラッグアンドドロップします。

![s231](images/s231.png)

未接続のノードを全て「切断/ハングアップ」ブロックに繋げます。

![s232](images/s232.png)

右上の［保存して発行］ボタンをクリックします。

![s233](images/s233.png)

左側メニューのルーティングから［電話番号］をクリックします。

![s234](images/s234.png)

電話番号をクリックします。

![s235](images/s235.png)

問い合わせフローを作成した「BMIフロー」を選択します。

![s236](images/s236.png)

これで電話番号かけて、身長と体重の値を入力すればBMI値が返ってきます。

## DynamoDBと連携しよう！
### 3-1. テーブルを作成する
サービスから「DynamoDB」を検索してクリックします。

![s237](images/s237.png)

［テーブルの作成］ボタンをクリックします。

![s238](images/s238.png)

テーブル名とプライマリキーを入力して［作成］ボタンをクリックします。

| 項目       |       値 |
|:-----------------|:------------------|
|テーブル名|PhoneHistory|
|プライマリキー|Timestamp|

![s239](images/s239.png)

Lambda画面を開いて、新しいファイルを新規作成します。［＋］ボタンをクリックし、［New File］をクリックします。

![s240](images/s240.png)

下記コードを入力します。

```javascript:util.js
'use strict';
const AWS = require('aws-sdk');
const DynamoDB = new AWS.DynamoDB.DocumentClient({
  region: "ap-northeast-1"
});

// 日付フォーマット変更
module.exports.dateToStr12HPad0 = function dateToStr12HPad0(date, format) {
    if (!format) {
        format = 'YYYY/MM/DD hh:mm:dd AP';
    }
    format = format.replace(/YYYY/g, date.getFullYear());
    format = format.replace(/MM/g, ('0' + (date.getMonth() + 1)).slice(-2));
    format = format.replace(/DD/g, ('0' + date.getDate()).slice(-2));
    format = format.replace(/hh/g, ('0' + date.getHours()).slice(-2));
    format = format.replace(/mm/g, ('0' + date.getMinutes()).slice(-2));
    format = format.replace(/ss/g, ('0' + date.getSeconds()).slice(-2));
    format = format.replace(/dd/g, ('0' + date.getMilliseconds()).slice(-3));
    return format;
};

// 着信履歴を残す
module.exports.putPhoneNo = async function putPhoneNo(phoneNo) {
    // テーブル名取得
    const tableName = process.env.TABLE_NAME;
    
    var timezoneoffset = -9;     // UTC-表示したいタイムゾーン(単位:hour)。JSTなら-9
    var today = new Date(Date.now() - (timezoneoffset * 60 - new Date().getTimezoneOffset()) * 60000);

    const timestamp = this.dateToStr12HPad0(today, 'YYYY/MM/DD hh:mm:ss:dd');

    // DynamoDBにデータを保存する
    await DynamoDB.put( {
        "TableName": tableName,
        "Item": {
          "Timestamp": timestamp,
          "phoneNo": phoneNo
        }
    }, function( err, data ) {
        console.log(err);
        
    }).promise();
};
```

キーボードのコントロール+Sキーでファイルを保存します。ファイル名は「util.js」にして［Save］ボタンをクリックします。

![s241](images/s241.png)

環境変数を設定します。

| 項目       |       値 |
|:-----------------|:------------------|
|キー|TABLE_NAME|
|値|PhoneHistory|

![s242](images/s242.png)

index.jsのソースを編集します。

```javascript
// utilファイルを読み取り
const Util = require('util.js');

exports.handler = async (event) => {
    
    // 発信者番号
    const phoneNumber = event.Details.ContactData.CustomerEndpoint.Address;

    // 発信者番号をDynamoDBに記録
    await Util.putPhoneNo(phoneNumber);

    // 身長と体重を取得する
    const heightVal = event.Details.ContactData.Attributes.HeightVal;
    const weightVal = event.Details.ContactData.Attributes.WeightVal;
    
    // BMI計算
    const bmiVal = (parseFloat(weightVal) / (parseFloat(heightVal)/100 * parseFloat(heightVal)/100)).toFixed(1);

    // 標準体重
    const stdWeight = (22 * (parseFloat(heightVal)/100 * parseFloat(heightVal)/100)).toFixed(1);

    var speechText = `あなたのBMIは${bmiVal}です。標準体重は${stdWeight}kgです。`;

    return {"BMI": speechText};
};
```

![s243](images/s243.png)

これでAmazon Connectの電話にかけてBMIの返答が返ってきたら、DynamoDBに発信者の電話番号が記録されます。

![s244](images/s244.png)
