<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込む必要があります。 このアプリケーションでは [、Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) を使用して Microsoft Graph を呼び出します。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

このセクションでは、クラスを拡張して、現在の週のユーザーのイベントを取得する関数を追加し、これらの新しい関数を使用 `GraphManager` `CalendarViewController` する更新を行います。

1. **GraphManager.swift を開** き、次のメソッドをクラスに追加 `GraphManager` します。

    ```Swift
    public func getCalendarView(viewStart: String,
                                viewEnd: String,
                                completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/calendarview
        // Set start and end of the view
        let start = "startDateTime=\(viewStart)"
        let end = "endDateTime=\(viewEnd)"
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=start/dateTime"
        // Request at most 25 results
        let top = "$top=25"

        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/calendarview?\(start)&\(end)&\(select)&\(orderBy)&\(top)")!)

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        eventsRequest.addValue("outlook.timezone=\"\(self.userTimeZone)\"", forHTTPHeaderField: "Prefer")

        let eventsDataTask = MSURLSessionDataTask(request: eventsRequest, client: self.client, completion: {
            (data: Data?, response: URLResponse?, graphError: Error?) in
            guard let eventsData = data, graphError == nil else {
                completion(nil, graphError)
                return
            }

            // TEMPORARY
            completion(eventsData, nil)
        })

        // Execute the request
        eventsDataTask?.execute()
    }
    ```

    > [!NOTE]
    > コードの実行を `getCalendarView` 検討します。
    >
    > - 呼び出される URL は `/v1.0/me/calendarview` です。
    >   - The `startDateTime` and query parameters define the start and end of the calendar `endDateTime` view.
    >   - クエリ パラメーターは、各イベントで返されるフィールドを、ビューが実際に使用 `select` するフィールドに限定します。
    >   - クエリ `orderby` パラメーターは、開始時刻で結果を並べ替える。
    >   - クエリ `top` パラメーターは、1 ページあたり 25 の結果を要求します。
    >   - このヘッダーにより、Microsoft Graph はユーザーのタイム ゾーン内の各イベントの開始時刻と終了 `Prefer: outlook.timezone` 時刻を返します。

1. **GraphToIana.swift** という **名前の GraphTu graphl** プロジェクトに新しい **Swift** ファイルを作成します。 次のコードをファイルに追加します。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.swift" id="GraphToIanaSnippet":::

    これにより、Microsoft Graph によって返されるタイム ゾーン名に基づいて IANA タイム ゾーン識別子を検索する簡単な検索が実行されます。

1. **CalendarViewController.swift を開き**、内容全体を次のコードに置き換えます。

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            // Calculate the start and end of the current week
            let timeZone = GraphToIana.getIanaIdentifier(graphIdentifer: GraphManager.instance.userTimeZone)
            let now = Date()
            var calendar = Calendar(identifier: .gregorian)
            calendar.timeZone = TimeZone(identifier: timeZone)!

            let startOfWeek = calendar.dateComponents([.calendar, .yearForWeekOfYear, .weekOfYear], from: now).date!
            let endOfWeek = calendar.date(byAdding: .day, value: 7, to: startOfWeek)!

            // Convert start and end to ISO 8601 strings
            let isoFormatter = ISO8601DateFormatter()
            let viewStart = isoFormatter.string(from: startOfWeek)
            let viewEnd = isoFormatter.string(from: endOfWeek)

            GraphManager.instance.getCalendarView(viewStart: viewStart, viewEnd: viewEnd) {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    // TEMPORARY
                    guard let eventsData = data, error == nil else {
                        self.calendarJSON.text = error.debugDescription
                        return
                    }

                    let jsonString = String(data: eventsData, encoding: .utf8)
                    self.calendarJSON.text = jsonString
                    self.calendarJSON.sizeToFit()
                }
            }
        }
    }
    ```

これで、アプリを実行し、サインインし、メニューの **予定表** ナビゲーション項目をタップできます。 アプリにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、JSON ダンプを何かに置き換え、結果をユーザー に分け親しみのある方法で表示できます。 このセクションでは、関数を変更して、型指定されたオブジェクトを返し、テーブル ビューを使用してイベントを `getCalendarView` `CalendarViewController` レンダリングする変更を行います。

1. **GraphManager.swift を開きます**。 既存の `getCalendarView` 関数を、以下の関数で置き換えます。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="3,28-49":::

1. **GraphTu touchl** **プロジェクトに新** しい Cocoa Touch Class ファイルを作成します `CalendarTableViewController.swift` 。 フィールド **のサブクラスで UITableViewController** **を選択** します。

1. **CalendarTableViewController.swift を開き**、その内容を次の内容に置き換えてください。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.swift" id="CalendarTableViewControllerSnippet":::

1. **GraphTu touchl** **プロジェクトに新** しい Cocoa Touch Class ファイルを作成します `CalendarTableViewCell.swift` 。 フィールド **のサブクラスで UITableViewCell** **を選択** します。

1. **CalendarTableViewCell.swift を開き**、次のプロパティをクラスに追加 `CalendarTableViewCell` します。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. **Main.storyboard を開** き、予定表シーン **を見つける**。 ルート ビューからスクロール ビューを削除します。
1. ライブラリを **使用** して、ビュー **の上部** にナビゲーション バーを追加します。
1. ナビゲーション バーでタイトル **を** ダブルクリックし、次の値に更新します `Calendar` 。
1. ライブラリを **使用** して、ナビゲーション バー **の** 右側にバー ボタン項目を追加します。
1. 新しいバー ボタンを選択し、属性インスペクター **を選択します**。 イメージ **をプラスに****変更します**。
1. ライブラリから **ナビゲーション バー** の **下の** ビューにコンテナー ビューを追加します。 コンテナー ビューのサイズを変更して、ビュー内の残りのすべての領域を取得します。
1. ナビゲーション バーとコンテナー ビューの制約を次のように設定します。

    - **ナビゲーション バー**
        - 制約の追加: 高さ、値: 44
        - 制約の追加: 安全領域への先頭のスペース、値: 0
        - 制約を追加する: 安全領域への末尾のスペース、値: 0
        - 制約を追加する: 安全な領域に上のスペース、値: 0
    - **コンテナー ビュー**
        - 制約の追加: 安全領域への先頭のスペース、値: 0
        - 制約を追加する: 安全領域への末尾のスペース、値: 0
        - 制約の追加: ナビゲーション バーの下に上のスペース、値: 0
        - 制約の追加: セーフ エリアへの下部スペース、値: 0

1. コンテナー ビューを追加するときにストーリーボードに追加された 2 番目のビュー コントローラーを見つける。 埋め込みセグ **によってカレンダー** シーンに接続されます。 このコントローラーを選択し **、Id インスペクターを使用** して **Class** を **CalendarTableViewController に変更します**。
1. カレンダー テーブル **ビュー** コントローラーから **ビューを削除します**。
1. ライブラリから **予定表テーブル ビュー** **コントローラーに** テーブル **ビューを追加します**。
1. テーブル ビューを選択し、属性インスペクター **を選択します**。 プロトタイプ セル **を** **1 に設定します**。
1. プロトタイプ セルの下端をドラッグして、操作する領域を広くします。
1. ライブラリを **使用して** 、プロトタイプ セルに **3 つのラベル** を追加します。
1. プロトタイプ セルを選択し、Identity Inspector を **選択します**。 クラス **を** **CalendarTableViewCell に変更します**。
1. 属性インスペクタ **ーを選択し、[** 識別子 **] を次に設定** します `EventCell` 。
1. **EventCell を** 選択した後 **、Connections Inspector** を選択して接続し、ストーリーボードのセルに追加したラベル `durationLabel` `organizerLabel` `subjectLabel` に接続します。
1. 次のように、3 つのラベルのプロパティと制約を設定します。

    - **件名ラベル**
        - 制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0
        - 制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0
        - 制約の追加: コンテンツ ビューの上余白に上余白、値: 0
    - **開催者ラベル**
        - フォント: System 12.0
        - 制約の追加: 高さ、値: 15
        - 制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0
        - 制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0
        - 制約の追加: 件名ラベルの下に上のスペース、値: Standard
    - **期間ラベル**
        - フォント: System 12.0
        - 色: 濃い灰色
        - 制約の追加: 高さ、値: 15
        - 制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0
        - 制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0
        - 制約の追加: [開催者ラベルの下に上のスペース]、値 : [標準]
        - 制約の追加: コンテンツ ビューの下余白に下のスペース、値: 0

1. **EventCell を選択し**、サイズ検査を **選択します**。 行 **の高さに** 対 **して自動を有効にする**。

    ![予定表と予定表のテーブル ビュー コントローラーのスクリーンショット](images/calendar-table-storyboard.png)

1. **CalendarViewController.swift を開き、** 内容を次のコードに置き換えます。

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. アプリを実行し、サインインして、[予定表] タブ **をタップ** します。イベントの一覧が表示されます。

    ![イベント表のスクリーンショット](images/calendar-list.png)
