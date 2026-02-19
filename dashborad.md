## Operation
---
1. デイリーノートを作成

## Tasks
---
<!--
1. happens today - 今日発生するタスク（繰り返しタスクの次回発生日が今日の場合のみ）
2. (is not recurring) AND (starts on or before today) AND (happens on or after today) - 繰り返しでないタスクで、開始日が今日以前かつ、期限日（due date）またはスケジュール日（scheduled date）のいずれかが今日以降が今日以降のもの
3. (is recurring) AND (no happens date) - 日付が設定されていない繰り返しタスク（毎日実行すべきタスクなど）
-->

### Today
```tasks
not done
(happens today) OR ((is not recurring) AND (starts on or before today) AND (happens on or after today)) OR ((is recurring) AND (no happens date))
sort by priority
short
```

### Overdue
```tasks
not done
due before today
sort by priority
short
```

### Other Tasks
```tasks
not done
NOT ((happens today) OR ((is not recurring) AND (starts on or before today) AND (happens on or after today)) OR (due before today))
short
```



