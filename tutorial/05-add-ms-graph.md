<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4a27b-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="4a27b-102">このアプリケーションでは、microsoft graph [SDK for Use C](https://github.com/microsoftgraph/msgraph-sdk-objc)を使用して microsoft graph を呼び出すことにします。</span><span class="sxs-lookup"><span data-stu-id="4a27b-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="4a27b-103">Outlook から予定表のイベントを取得する</span><span class="sxs-lookup"><span data-stu-id="4a27b-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="4a27b-104">このセクションでは、 `GraphManager`クラスを拡張して、ユーザーのイベントを取得する関数を追加`CalendarViewController`し、これらの新しい関数を使用するように更新します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="4a27b-105">**Graphmanager**を開き、次のメソッドを`GraphManager`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="4a27b-106">のコードに`getEvents`ついて検討します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="4a27b-107">呼び出し先の URL は`/v1.0/me/events`になります。</span><span class="sxs-lookup"><span data-stu-id="4a27b-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="4a27b-108">Query `select`パラメーターは、各イベントに対して返されるフィールドを、アプリが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="4a27b-109">クエリ`orderby`パラメーターは、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="4a27b-110">**Calendarviewcontroller**を開き、その内容全体を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="4a27b-111">これで、アプリを実行し、サインインすると、メニューの [**予定表**] ナビゲーション項目をタップできるようになります。</span><span class="sxs-lookup"><span data-stu-id="4a27b-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="4a27b-112">アプリ内のイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="4a27b-113">結果を表示する</span><span class="sxs-lookup"><span data-stu-id="4a27b-113">Display the results</span></span>

<span data-ttu-id="4a27b-114">これで、JSON ダンプを、ユーザーにわかりやすい方法で結果を表示するためのものに置き換えることができます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="4a27b-115">このセクションでは、厳密に型`getEvents`指定されたオブジェクトを返すように関数`CalendarViewController`を変更し、テーブルビューを使用してイベントをレンダリングするように変更します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="4a27b-116">GetEvents の更新</span><span class="sxs-lookup"><span data-stu-id="4a27b-116">Update getEvents</span></span>

1. <span data-ttu-id="4a27b-117">**Graphmanager を**開きます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-117">Open **GraphManager.swift**.</span></span> <span data-ttu-id="4a27b-118">`getEvents`関数宣言を次のように変更します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-118">Change the `getEvents` function declaration to the following.</span></span>

    ```Swift
    public func getEvents(completion: @escaping([MSGraphEvent]?, Error?) -> Void)
    ```

1. <span data-ttu-id="4a27b-119">行を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-119">Replace the `completion(eventsData, nil)` line with the following code.</span></span>

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

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="4a27b-120">CalendarViewController の更新</span><span class="sxs-lookup"><span data-stu-id="4a27b-120">Update CalendarViewController</span></span>

1. <span data-ttu-id="4a27b-121">という名前`CalendarTableViewCell.swift`の**graphtutorial**プロジェクトに、新しい**cocoa タッチクラス**ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-121">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="4a27b-122">Field**のサブクラス**で [ **Uitableviewcell セル**を選択します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-122">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="4a27b-123">**Calendartableviewcell**を開き、次のコードを`CalendarTableViewCell`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-123">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

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

1. <span data-ttu-id="4a27b-124">**メインのストーリーボード**を開いて、**予定表のシーン**を探します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-124">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="4a27b-125">**予定表のシーン**で**ビュー**を選択し、削除します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-125">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![予定表シーンのビューのスクリーンショット](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="4a27b-127">**ライブラリ**から予定表の**シーン**に**テーブルビュー**を追加します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-127">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="4a27b-128">テーブルビューを選択し、[**属性インスペクター**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-128">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="4a27b-129">**プロトタイプセル**を**1**に設定します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-129">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="4a27b-130">**ライブラリ**を使用して、[プロトタイプ] セルに3つの**ラベル**を追加します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-130">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="4a27b-131">[プロトタイプ] セルを選択し、[ **Identity Inspector**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-131">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="4a27b-132">**クラス**を**Calendartableviewcell**に変更します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-132">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="4a27b-133">[ **Attributes インスペクター** ] を選択し、 `EventCell`**識別子**をに設定します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-133">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="4a27b-134">[ **Eventcell セル**を選択した状態で、[ `durationLabel`**接続インスペクター** ] `subjectLabel` 、[接続] `organizerLabel`、[] の順に選択し、ストーリーボードのセルに追加したラベルに接続します。</span><span class="sxs-lookup"><span data-stu-id="4a27b-134">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![プロトタイプセルのレイアウトのスクリーンショット](./images/prototype-cell-layout.png)

1. <span data-ttu-id="4a27b-136">**Calendarviewcontroller**を開き、その内容を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-136">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

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

1. <span data-ttu-id="4a27b-137">アプリを実行し、サインインして [**予定表**] タブをタップします。イベントの一覧が表示されます。</span><span class="sxs-lookup"><span data-stu-id="4a27b-137">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![イベントの表のスクリーンショット](./images/calendar-list.png)
