<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、microsoft graph [SDK for Use C](https://github.com/microsoftgraph/msgraph-sdk-objc)を使用して microsoft graph を呼び出すことにします。

## <a name="get-calendar-events-from-outlook"></a>Outlook から予定表のイベントを取得する

このセクションでは、 `GraphManager`クラスを拡張して、ユーザーのイベントを取得する関数を追加`CalendarViewController`し、これらの新しい関数を使用するように更新します。

1. **Graphmanager**を開き、次のメソッドを`GraphManager`クラスに追加します。

    ```Swift
    public func getEvents(completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=createdDateTime+DESC"
        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/events?\(select)&\(orderBy)")!)
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
    > のコードに`getEvents`ついて検討します。
    >
    > - 呼び出し先の URL は`/v1.0/me/events`になります。
    > - Query `select`パラメーターは、各イベントに対して返されるフィールドを、アプリが実際に使用するものだけに制限します。
    > - クエリ`orderby`パラメーターは、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。

1. **Calendarviewcontroller**を開き、その内容全体を次のコードで置き換えます。

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

            GraphManager.instance.getEvents {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

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

これで、アプリを実行し、サインインすると、メニューの [**予定表**] ナビゲーション項目をタップできるようになります。 アプリ内のイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果を表示する

これで、JSON ダンプを、ユーザーにわかりやすい方法で結果を表示するためのものに置き換えることができます。 このセクションでは、厳密に型`getEvents`指定されたオブジェクトを返すように関数`CalendarViewController`を変更し、テーブルビューを使用してイベントをレンダリングするように変更します。

### <a name="update-getevents"></a>GetEvents の更新

1. **Graphmanager を**開きます。 `getEvents`関数宣言を次のように変更します。

    ```Swift
    public func getEvents(completion: @escaping([MSGraphEvent]?, Error?) -> Void)
    ```

1. 行を次のコードに置き換えます。

    ```Swift
    do {
        // Deserialize response as events collection
        let eventsCollection = try MSCollection(data: eventsData)
        var eventArray: [MSGraphEvent] = []

        eventsCollection.value.forEach({
            (rawEvent: Any) in
            // Convert JSON to a dictionary
            guard let eventDict = rawEvent as? [String: Any] else {
                return
            }

            // Deserialize event from the dictionary
            let event = MSGraphEvent(dictionary: eventDict)!
            eventArray.append(event)
        })

        // Return the array
        completion(eventArray, nil)
    } catch {
        completion(nil, error)
    }
    ```

### <a name="update-calendarviewcontroller"></a>CalendarViewController の更新

1. という名前`CalendarTableViewCell.swift`の**graphtutorial**プロジェクトに、新しい**cocoa タッチクラス**ファイルを作成します。 Field**のサブクラス**で [ **Uitableviewcell セル**を選択します。
1. **Calendartableviewcell**を開き、次のコードを`CalendarTableViewCell`クラスに追加します。

    ```Swift
    @IBOutlet var subjectLabel: UILabel!
    @IBOutlet var organizerLabel: UILabel!
    @IBOutlet var durationLabel: UILabel!

    var subject: String? {
        didSet {
            subjectLabel.text = subject
        }
    }

    var organizer: String? {
        didSet {
            organizerLabel.text = organizer
        }
    }

    var duration: String? {
        didSet {
            durationLabel.text = duration
        }
    }
    ```

1. **メインのストーリーボード**を開いて、**予定表のシーン**を探します。 **予定表のシーン**で**ビュー**を選択し、削除します。

    ![予定表シーンのビューのスクリーンショット](./images/view-in-calendar-scene.png)

1. **ライブラリ**から予定表の**シーン**に**テーブルビュー**を追加します。
1. テーブルビューを選択し、[**属性インスペクター**] を選択します。 **プロトタイプセル**を**1**に設定します。
1. **ライブラリ**を使用して、[プロトタイプ] セルに3つの**ラベル**を追加します。
1. [プロトタイプ] セルを選択し、[ **Identity Inspector**] を選択します。 **クラス**を**Calendartableviewcell**に変更します。
1. [ **Attributes インスペクター** ] を選択し、 `EventCell`**識別子**をに設定します。
1. [ **Eventcell セル**を選択した状態で、[ `durationLabel`**接続インスペクター** ] `subjectLabel` 、[接続] `organizerLabel`、[] の順に選択し、ストーリーボードのセルに追加したラベルに接続します。

    ![プロトタイプセルのレイアウトのスクリーンショット](./images/prototype-cell-layout.png)

1. **Calendarviewcontroller**を開き、その内容を次のコードで置き換えます。

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UITableViewController {

        private let tableCellIdentifier = "EventCell"
        private let spinner = SpinnerViewController()
        private var events: [MSGraphEvent]?

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            GraphManager.instance.getEvents {
                (eventArray: [MSGraphEvent]?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let events = eventArray, error == nil else {
                        // Show the error
                        let alert = UIAlertController(title: "Error getting events",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    self.events = events
                    self.tableView.reloadData()
                }
            }
        }

        // Number of sections, always 1
        override func numberOfSections(in tableView: UITableView) -> Int {
            return 1
        }

        // Return the number of events in the table
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return events?.count ?? 0
        }

        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: tableCellIdentifier, for: indexPath) as! CalendarTableViewCell

            // Get the event that corresponds to the row
            let event = events?[indexPath.row]

            // Configure the cell
            cell.subject = event?.subject
            cell.organizer = event?.organizer?.emailAddress?.name

            // Build a duration string
            let duration = "\(self.formatGraphDateTime(dateTime: event?.start)) to \(self.formatGraphDateTime(dateTime: event?.end))"
            cell.duration = duration

            return cell
        }

        private func formatGraphDateTime(dateTime: MSGraphDateTimeTimeZone?) -> String {
            guard let graphDateTime = dateTime else {
                return ""
            }

            // Create a formatter to parse Graph's date format
            let isoFormatter = DateFormatter()
            isoFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSSSSS"

            // Specify the timezone
            isoFormatter.timeZone = TimeZone(identifier: graphDateTime.timeZone!)

            let date = isoFormatter.date(from: graphDateTime.dateTime)

            // Output like 5/5/2019, 2:00 PM
            let dateFormatter = DateFormatter()
            dateFormatter.dateStyle = .short
            dateFormatter.timeStyle = .short

            return dateFormatter.string(from: date!)
        }
    }
    ```

1. アプリを実行し、サインインして [**予定表**] タブをタップします。イベントの一覧が表示されます。

    ![イベントの表のスクリーンショット](./images/calendar-list.png)
