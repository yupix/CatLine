# CatLine

このプロジェクトは簡易的な非同期 ジョブキューを実装するための物です。

## 注意事項

- `threading` を用いた並列処理はサポート・テストしていません
- `Queue` クラスは イベントループ作成後にインスタンス化してください。
- 優先度は 0 ~ 9 であり、 0に近づくほど優先度が高くなります。

## 使い方

以下が例になります。
このコードの意味をまとめると以下のようになります
- ジョブが実行された際に引数で渡されたジョブの名前で `meow` と出力する
- 優先度が0のcatには `VIP` を先頭につけ、それ以外のcatは `normal` をつける
- 待機中のジョブが0になった際にループを終了し、キューを停止させる


```py
import asyncio
import random
from catline.adapters.json_adapter import QueueStorageJSONAdapter
from catline.queue import Queue

async def say_meow(cat_name: str):
    print(f'{cat_name}: meow' + '!' * random.randrange(1,10))


async def main():
    cat_queue = Queue('cat',  QueueStorageJSONAdapter(), say_meow)
    cat_queue.run()
    for i in range(random.randrange(1, 20)):
        priority = random.randrange(0, 9)
        await cat_queue.add(priority, f'{"VIP" if priority == 0 else "normal"} cat{i}') # type: ignore
    while True:
        waiting_cat_number = await cat_queue.count_jobs('waiting')
        print(f'waiting cat number: {waiting_cat_number}')
        await asyncio.sleep(1)
        if waiting_cat_number == 0:
            break
    cat_queue.stop()
    print('finish!!!')

if __name__ == '__main__':
    asyncio.run(main())

```

## 実装予定の機能

|機能|説明|状態|
|---|---|---|
|ジョブの追加|ジョブをキューに追加できる|〇|
|ジョブの実行|キューにあるジョブをスケジューラーで実行できる|〇|
|ジョブのリトライ|ジョブが失敗した際に再実行できる|×|
|ジョブの優先度|ジョブを追加する際優先度を指定できる|〇|
|スケジューラーの実行間隔|スケジューラーがジョブを実行するまでの間隔を変更できる|〇|
|キューストレージの変更|キューをredisやjsonに保存する際自分でadapterを作れる|〇|
|threadingのサポート|並列処理のサポート|×|
|実行するジョブの上限変更|同時に実行できるジョブの数を変更できる|〇|

## ストレージを自分で作成する方法

このプロジェクトではキューのジョブを保存するためのクラスをアダプターパターンで作成しているため、それに沿ってクラスを作成すれば作成が可能です。

また、デフォルトで `json` 又は `redis` を用いた保存方法がライブラリに同封されているため、特段理由が無い場合はそちらを使うことを推奨します。

### 注意点

1. ジョブを取得する際に優先度順に取得するなどといった処理を自分で作成する必要があります。


