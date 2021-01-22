<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="383fd-101">この演習では、Microsoft Graph をアプリケーションに組み込む必要があります。</span><span class="sxs-lookup"><span data-stu-id="383fd-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="383fd-102">このアプリケーションでは [、Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) を使用して Microsoft Graph を呼び出します。</span><span class="sxs-lookup"><span data-stu-id="383fd-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="383fd-103">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="383fd-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="383fd-104">このセクションでは、クラスを拡張して、現在の週のユーザーのイベントを取得する関数を追加し、これらの新しい関数を使用 `GraphManager` `CalendarViewController` する更新を行います。</span><span class="sxs-lookup"><span data-stu-id="383fd-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="383fd-105">**GraphManager.swift を開** き、次のメソッドをクラスに追加 `GraphManager` します。</span><span class="sxs-lookup"><span data-stu-id="383fd-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="383fd-106">コードの実行を `getCalendarView` 検討します。</span><span class="sxs-lookup"><span data-stu-id="383fd-106">Consider what the code in `getCalendarView` is doing.</span></span>
    >
    > - <span data-ttu-id="383fd-107">呼び出される URL は `/v1.0/me/calendarview` です。</span><span class="sxs-lookup"><span data-stu-id="383fd-107">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="383fd-108">The `startDateTime` and query parameters define the start and end of the calendar `endDateTime` view.</span><span class="sxs-lookup"><span data-stu-id="383fd-108">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="383fd-109">クエリ パラメーターは、各イベントで返されるフィールドを、ビューが実際に使用 `select` するフィールドに限定します。</span><span class="sxs-lookup"><span data-stu-id="383fd-109">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="383fd-110">クエリ `orderby` パラメーターは、開始時刻で結果を並べ替える。</span><span class="sxs-lookup"><span data-stu-id="383fd-110">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="383fd-111">クエリ `top` パラメーターは、1 ページあたり 25 の結果を要求します。</span><span class="sxs-lookup"><span data-stu-id="383fd-111">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="383fd-112">このヘッダーにより、Microsoft Graph はユーザーのタイム ゾーン内の各イベントの開始時刻と終了 `Prefer: outlook.timezone` 時刻を返します。</span><span class="sxs-lookup"><span data-stu-id="383fd-112">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="383fd-113">**GraphToIana.swift** という **名前の GraphTu graphl** プロジェクトに新しい **Swift** ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="383fd-113">Create a new **Swift File** in the **GraphTutorial** project named **GraphToIana.swift**.</span></span> <span data-ttu-id="383fd-114">次のコードをファイルに追加します。</span><span class="sxs-lookup"><span data-stu-id="383fd-114">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.swift" id="GraphToIanaSnippet":::

    <span data-ttu-id="383fd-115">これにより、Microsoft Graph によって返されるタイム ゾーン名に基づいて IANA タイム ゾーン識別子を検索する簡単な検索が実行されます。</span><span class="sxs-lookup"><span data-stu-id="383fd-115">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="383fd-116">**CalendarViewController.swift を開き**、内容全体を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="383fd-116">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="383fd-117">これで、アプリを実行し、サインインし、メニューの **予定表** ナビゲーション項目をタップできます。</span><span class="sxs-lookup"><span data-stu-id="383fd-117">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="383fd-118">アプリにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="383fd-118">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="383fd-119">結果の表示</span><span class="sxs-lookup"><span data-stu-id="383fd-119">Display the results</span></span>

<span data-ttu-id="383fd-120">これで、JSON ダンプを何かに置き換え、結果をユーザー に分け親しみのある方法で表示できます。</span><span class="sxs-lookup"><span data-stu-id="383fd-120">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="383fd-121">このセクションでは、関数を変更して、型指定されたオブジェクトを返し、テーブル ビューを使用してイベントを `getCalendarView` `CalendarViewController` レンダリングする変更を行います。</span><span class="sxs-lookup"><span data-stu-id="383fd-121">In this section, you will modify the `getCalendarView` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="383fd-122">**GraphManager.swift を開きます**。</span><span class="sxs-lookup"><span data-stu-id="383fd-122">Open **GraphManager.swift**.</span></span> <span data-ttu-id="383fd-123">既存の `getCalendarView` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="383fd-123">Replace the existing `getCalendarView` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="3,28-49":::

1. <span data-ttu-id="383fd-124">**GraphTu touchl** **プロジェクトに新** しい Cocoa Touch Class ファイルを作成します `CalendarTableViewController.swift` 。</span><span class="sxs-lookup"><span data-stu-id="383fd-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController.swift`.</span></span> <span data-ttu-id="383fd-125">フィールド **のサブクラスで UITableViewController** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="383fd-125">Choose **UITableViewController** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="383fd-126">**CalendarTableViewController.swift を開き**、その内容を次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="383fd-126">Open **CalendarTableViewController.swift** and replace its contents with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.swift" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="383fd-127">**GraphTu touchl** **プロジェクトに新** しい Cocoa Touch Class ファイルを作成します `CalendarTableViewCell.swift` 。</span><span class="sxs-lookup"><span data-stu-id="383fd-127">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="383fd-128">フィールド **のサブクラスで UITableViewCell** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="383fd-128">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="383fd-129">**CalendarTableViewCell.swift を開き**、次のプロパティをクラスに追加 `CalendarTableViewCell` します。</span><span class="sxs-lookup"><span data-stu-id="383fd-129">Open **CalendarTableViewCell.swift** and add the following properties to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="383fd-130">**Main.storyboard を開** き、予定表シーン **を見つける**。</span><span class="sxs-lookup"><span data-stu-id="383fd-130">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="383fd-131">ルート ビューからスクロール ビューを削除します。</span><span class="sxs-lookup"><span data-stu-id="383fd-131">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="383fd-132">ライブラリを **使用** して、ビュー **の上部** にナビゲーション バーを追加します。</span><span class="sxs-lookup"><span data-stu-id="383fd-132">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="383fd-133">ナビゲーション バーでタイトル **を** ダブルクリックし、次の値に更新します `Calendar` 。</span><span class="sxs-lookup"><span data-stu-id="383fd-133">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="383fd-134">ライブラリを **使用** して、ナビゲーション バー **の** 右側にバー ボタン項目を追加します。</span><span class="sxs-lookup"><span data-stu-id="383fd-134">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="383fd-135">新しいバー ボタンを選択し、属性インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-135">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="383fd-136">イメージ **をプラスに\*\*\*\*変更します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-136">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="383fd-137">ライブラリから **ナビゲーション バー** の **下の** ビューにコンテナー ビューを追加します。</span><span class="sxs-lookup"><span data-stu-id="383fd-137">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="383fd-138">コンテナー ビューのサイズを変更して、ビュー内の残りのすべての領域を取得します。</span><span class="sxs-lookup"><span data-stu-id="383fd-138">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="383fd-139">ナビゲーション バーとコンテナー ビューの制約を次のように設定します。</span><span class="sxs-lookup"><span data-stu-id="383fd-139">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="383fd-140">**ナビゲーション バー**</span><span class="sxs-lookup"><span data-stu-id="383fd-140">**Navigation Bar**</span></span>
        - <span data-ttu-id="383fd-141">制約の追加: 高さ、値: 44</span><span class="sxs-lookup"><span data-stu-id="383fd-141">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="383fd-142">制約の追加: 安全領域への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-142">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="383fd-143">制約を追加する: 安全領域への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-143">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="383fd-144">制約を追加する: 安全な領域に上のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-144">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="383fd-145">**コンテナー ビュー**</span><span class="sxs-lookup"><span data-stu-id="383fd-145">**Container View**</span></span>
        - <span data-ttu-id="383fd-146">制約の追加: 安全領域への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-146">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="383fd-147">制約を追加する: 安全領域への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-147">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="383fd-148">制約の追加: ナビゲーション バーの下に上のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-148">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="383fd-149">制約の追加: セーフ エリアへの下部スペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-149">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="383fd-150">コンテナー ビューを追加するときにストーリーボードに追加された 2 番目のビュー コントローラーを見つける。</span><span class="sxs-lookup"><span data-stu-id="383fd-150">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="383fd-151">埋め込みセグ **によってカレンダー** シーンに接続されます。</span><span class="sxs-lookup"><span data-stu-id="383fd-151">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="383fd-152">このコントローラーを選択し **、Id インスペクターを使用** して **Class** を **CalendarTableViewController に変更します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-152">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="383fd-153">カレンダー テーブル **ビュー** コントローラーから **ビューを削除します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-153">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="383fd-154">ライブラリから **予定表テーブル ビュー** **コントローラーに** テーブル **ビューを追加します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-154">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="383fd-155">テーブル ビューを選択し、属性インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-155">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="383fd-156">プロトタイプ セル **を** **1 に設定します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-156">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="383fd-157">プロトタイプ セルの下端をドラッグして、操作する領域を広くします。</span><span class="sxs-lookup"><span data-stu-id="383fd-157">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="383fd-158">ライブラリを **使用して** 、プロトタイプ セルに **3 つのラベル** を追加します。</span><span class="sxs-lookup"><span data-stu-id="383fd-158">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="383fd-159">プロトタイプ セルを選択し、Identity Inspector を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-159">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="383fd-160">クラス **を** **CalendarTableViewCell に変更します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-160">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="383fd-161">属性インスペクタ **ーを選択し、[** 識別子 **] を次に設定** します `EventCell` 。</span><span class="sxs-lookup"><span data-stu-id="383fd-161">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="383fd-162">**EventCell を** 選択した後 **、Connections Inspector** を選択して接続し、ストーリーボードのセルに追加したラベル `durationLabel` `organizerLabel` `subjectLabel` に接続します。</span><span class="sxs-lookup"><span data-stu-id="383fd-162">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="383fd-163">次のように、3 つのラベルのプロパティと制約を設定します。</span><span class="sxs-lookup"><span data-stu-id="383fd-163">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="383fd-164">**件名ラベル**</span><span class="sxs-lookup"><span data-stu-id="383fd-164">**Subject Label**</span></span>
        - <span data-ttu-id="383fd-165">制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0</span><span class="sxs-lookup"><span data-stu-id="383fd-165">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-166">制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-166">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-167">制約の追加: コンテンツ ビューの上余白に上余白、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-167">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="383fd-168">**開催者ラベル**</span><span class="sxs-lookup"><span data-stu-id="383fd-168">**Organizer Label**</span></span>
        - <span data-ttu-id="383fd-169">フォント: System 12.0</span><span class="sxs-lookup"><span data-stu-id="383fd-169">Font: System 12.0</span></span>
        - <span data-ttu-id="383fd-170">制約の追加: 高さ、値: 15</span><span class="sxs-lookup"><span data-stu-id="383fd-170">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="383fd-171">制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0</span><span class="sxs-lookup"><span data-stu-id="383fd-171">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-172">制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-172">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-173">制約の追加: 件名ラベルの下に上のスペース、値: Standard</span><span class="sxs-lookup"><span data-stu-id="383fd-173">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="383fd-174">**期間ラベル**</span><span class="sxs-lookup"><span data-stu-id="383fd-174">**Duration Label**</span></span>
        - <span data-ttu-id="383fd-175">フォント: System 12.0</span><span class="sxs-lookup"><span data-stu-id="383fd-175">Font: System 12.0</span></span>
        - <span data-ttu-id="383fd-176">色: 濃い灰色</span><span class="sxs-lookup"><span data-stu-id="383fd-176">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="383fd-177">制約の追加: 高さ、値: 15</span><span class="sxs-lookup"><span data-stu-id="383fd-177">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="383fd-178">制約の追加: コンテンツ ビューの先頭に余白を付け、値 : 0</span><span class="sxs-lookup"><span data-stu-id="383fd-178">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-179">制約を追加する: コンテンツ ビューの末尾の余白に末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-179">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="383fd-180">制約の追加: [開催者ラベルの下に上のスペース]、値 : [標準]</span><span class="sxs-lookup"><span data-stu-id="383fd-180">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="383fd-181">制約の追加: コンテンツ ビューの下余白に下のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="383fd-181">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="383fd-182">**EventCell を選択し**、サイズ検査を **選択します**。</span><span class="sxs-lookup"><span data-stu-id="383fd-182">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="383fd-183">行 **の高さに** 対 **して自動を有効にする**。</span><span class="sxs-lookup"><span data-stu-id="383fd-183">Enable **Automatic** for **Row Height**.</span></span>

    ![予定表と予定表のテーブル ビュー コントローラーのスクリーンショット](images/calendar-table-storyboard.png)

1. <span data-ttu-id="383fd-185">**CalendarViewController.swift を開き、** 内容を次のコードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="383fd-185">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="383fd-186">アプリを実行し、サインインして、[予定表] タブ **をタップ** します。イベントの一覧が表示されます。</span><span class="sxs-lookup"><span data-stu-id="383fd-186">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![イベント表のスクリーンショット](images/calendar-list.png)
