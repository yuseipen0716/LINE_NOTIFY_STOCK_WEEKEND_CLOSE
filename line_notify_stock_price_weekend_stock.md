# 米国株の週終値の先週比をお知らせするbot

5%ルール点灯をお知らせするためのもの。GASで作成

定期的にお知らせをしてもらうためのTrigger登録は`setTrigger`関数をGASの画面上で実行する必要がある。

```javascript
const ACCESS_TOKEN = 'XXXXXXXXXXXXXXXXXXXX'; // LINE NOTIFYのアクセストークン
const LINE_NOTIFY_URL = 'https://notify-api.line.me/api/notify';
const ALPHA_VANTAGE_API_KEY = 'XXXXXXXXXXXXXXXXX'; // alpha vantageのAPIキー
const ALPHA_VANTAGE_API_WEEKLY_URL = `https://www.alphavantage.co/query?function=TIME_SERIES_WEEKLY&symbol=VTI&apikey=${ALPHA_VANTAGE_API_KEY}`;

function checkStockPriceAndNotify () {
  const response = UrlFetchApp.fetch(ALPHA_VANTAGE_API_WEEKLY_URL);
  const data = JSON.parse(response.getContentText());
  const timeSeries = data['Weekly Time Series'];

  // 値を降順にソート → 最新の週のデータと前週のデータを取得
  const sortedKeys = Object.keys(timeSeries).sort().reverse();
  const thisWeek = parseFloat(timeSeries[sortedKeys[0]]['4. close']);
  const lastWeek = parseFloat(timeSeries[sortedKeys[1]]['4. close']);

  const comparePercentage = ((thisWeek - lastWeek) / lastWeek) * 100;

  let message = `\n今週のVTIの終値は先週比 ${comparePercentage > 0 ? '+' : ''}${comparePercentage.toFixed(2)}% です。`;
  if (comparePercentage <= -5) {
    message += '\n\n💡 5%ルール点灯しました💡'
  }
  message += '\n\n=================\nこちらも併せてご確認ください。\nhttps://us.kabutan.jp/stocks/VTI/historical_prices/weekly'

  UrlFetchApp.fetch(LINE_NOTIFY_URL, {
    'headers': {
      'Authorization': 'Bearer ' + ACCESS_TOKEN,
    },
    'method': 'post',
    'payload': {
      'message': message,
    }
  });
}

function compareStockPriceYesterdayToLastWeekend () {
  const response = UrlFetchApp.fetch(ALPHA_VANTAGE_API_WEEKLY_URL);
  const data = JSON.parse(response.getContentText());
  const timeSeries = data['Weekly Time Series'];

  // 値を降順にソート → 最新の週のデータと前週のデータを取得
  const sortedKeys = Object.keys(timeSeries).sort().reverse();
  const yesterday = parseFloat(timeSeries[sortedKeys[0]]['4. close']);
  const lastWeek = parseFloat(timeSeries[sortedKeys[1]]['4. close']);

  const comparePercentage = ((yesterday - lastWeek) / lastWeek) * 100;

  let message = `\n昨日VTIの終値は${yesterday}でした。先週比 ${comparePercentage > 0 ? '+' : ''}${comparePercentage.toFixed(2)}% です。`;
  if (comparePercentage <= -4) {
    message += '\n\n 4%以上の下落です💡'
  }
  message += '\n\n=================\nこちらも併せてご確認ください。\nhttps://us.kabutan.jp/stocks/VTI/historical_prices/weekly'

  UrlFetchApp.fetch(LINE_NOTIFY_URL, {
    'headers': {
      'Authorization': 'Bearer ' + ACCESS_TOKEN,
    },
    'method': 'post',
    'payload': {
      'message': message,
    }
  });
}

function setTrigger() {
  ScriptApp.newTrigger('checkStockPriceAndNotify').timeBased().everyWeeks(1).onWeekDay(ScriptApp.WeekDay.SATURDAY).atHour(7).create();
  ScriptApp.newTrigger('compareStockPriceYesterdayToLastWeekend').timeBased().everyWeeks(1).onWeekDay(ScriptApp.WeekDay.FRIDAY).atHour(10).create();
}
```
