<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ffc42-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="ffc42-102">このアプリケーションでは、microsoft graph [SDK for Use C](https://github.com/microsoftgraph/msgraph-sdk-objc)を使用して microsoft graph を呼び出すことにします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="ffc42-103">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="ffc42-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="ffc42-104">このセクションでは、 `GraphManager`クラスを拡張して、ユーザーのイベントを取得する関数を追加`CalendarViewController`し、これらの新しい関数を使用するように更新します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="ffc42-105">**Graphmanager**を開き、次のメソッドを`GraphManager`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="ffc42-106">のコードに`getEvents`ついて検討します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="ffc42-107">呼び出される URL は `/v1.0/me/events` です。</span><span class="sxs-lookup"><span data-stu-id="ffc42-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="ffc42-108">Query `select`パラメーターは、各イベントに対して返されるフィールドを、アプリが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="ffc42-109">クエリ`orderby`パラメーターは、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="ffc42-110">**Calendarviewcontroller**を開き、その内容全体を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="ffc42-111">これで、アプリを実行し、サインインすると、メニューの [**予定表**] ナビゲーション項目をタップできるようになります。</span><span class="sxs-lookup"><span data-stu-id="ffc42-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="ffc42-112">アプリ内のイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="ffc42-113">結果の表示</span><span class="sxs-lookup"><span data-stu-id="ffc42-113">Display the results</span></span>

<span data-ttu-id="ffc42-114">これで、JSON ダンプを、ユーザーにわかりやすい方法で結果を表示するためのものに置き換えることができます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="ffc42-115">このセクションでは、厳密に型`getEvents`指定されたオブジェクトを返すように関数`CalendarViewController`を変更し、テーブルビューを使用してイベントをレンダリングするように変更します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="ffc42-116">**Graphmanager を**開きます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-116">Open **GraphManager.swift**.</span></span> <span data-ttu-id="ffc42-117">既存の `getEvents` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-117">Replace the existing `getEvents` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="1,17-38":::

1. <span data-ttu-id="ffc42-118">という名前`CalendarTableViewCell.swift`の**graphtutorial**プロジェクトに、新しい**cocoa タッチクラス**ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-118">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="ffc42-119">Field**のサブクラス**で [ **Uitableviewcell セル**を選択します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-119">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="ffc42-120">**Calendartableviewcell**を開き、次のコードを`CalendarTableViewCell`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-120">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="ffc42-121">**メインのストーリーボード**を開いて、**予定表のシーン**を探します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-121">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="ffc42-122">**予定表のシーン**で**ビュー**を選択し、削除します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-122">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![予定表シーンのビューのスクリーンショット](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="ffc42-124">**ライブラリ**から予定表の**シーン**に**テーブルビュー**を追加します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-124">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="ffc42-125">テーブルビューを選択し、[**属性インスペクター**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-125">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="ffc42-126">**プロトタイプセル**を**1**に設定します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-126">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="ffc42-127">**ライブラリ**を使用して、[プロトタイプ] セルに3つの**ラベル**を追加します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-127">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="ffc42-128">[プロトタイプ] セルを選択し、[ **Identity Inspector**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-128">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="ffc42-129">**クラス**を**Calendartableviewcell**に変更します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-129">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="ffc42-130">[ **Attributes インスペクター** ] を選択し、 `EventCell`**識別子**をに設定します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-130">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="ffc42-131">[ **Eventcell セル**を選択した状態で、[ `durationLabel`**接続インスペクター** ] `subjectLabel` 、[接続] `organizerLabel`、[] の順に選択し、ストーリーボードのセルに追加したラベルに接続します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-131">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="ffc42-132">3つのラベルのプロパティと制約を次のように設定します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-132">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="ffc42-133">**件名ラベル**</span><span class="sxs-lookup"><span data-stu-id="ffc42-133">**Subject Label**</span></span>
        - <span data-ttu-id="ffc42-134">制約の追加: 先頭の領域をコンテンツビューの先頭余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-134">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-135">制約の追加: 末尾のスペースをコンテンツビューの末尾の余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-135">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-136">制約の追加: 上スペースからコンテンツビューの上余白、値: 0</span><span class="sxs-lookup"><span data-stu-id="ffc42-136">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="ffc42-137">**開催者のラベル**</span><span class="sxs-lookup"><span data-stu-id="ffc42-137">**Organizer Label**</span></span>
        - <span data-ttu-id="ffc42-138">フォント: システム12.0</span><span class="sxs-lookup"><span data-stu-id="ffc42-138">Font: System 12.0</span></span>
        - <span data-ttu-id="ffc42-139">制約の追加: 先頭の領域をコンテンツビューの先頭余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-139">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-140">制約の追加: 末尾のスペースをコンテンツビューの末尾の余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-140">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-141">制約の追加: 上スペースを件名ラベルの下に、値: Standard にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-141">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="ffc42-142">**期間のラベル**</span><span class="sxs-lookup"><span data-stu-id="ffc42-142">**Duration Label**</span></span>
        - <span data-ttu-id="ffc42-143">フォント: システム12.0</span><span class="sxs-lookup"><span data-stu-id="ffc42-143">Font: System 12.0</span></span>
        - <span data-ttu-id="ffc42-144">色: 濃い灰色</span><span class="sxs-lookup"><span data-stu-id="ffc42-144">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="ffc42-145">制約の追加: 先頭の領域をコンテンツビューの先頭余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-145">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-146">制約の追加: 末尾のスペースをコンテンツビューの末尾の余白、値: 0 にします。</span><span class="sxs-lookup"><span data-stu-id="ffc42-146">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="ffc42-147">制約の追加: 上のスペースから開催者ラベルの下、値: 標準</span><span class="sxs-lookup"><span data-stu-id="ffc42-147">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="ffc42-148">制約の追加: 下のスペースをコンテンツビューの下の余白、値: 8 に制限します。</span><span class="sxs-lookup"><span data-stu-id="ffc42-148">Add constraint: Bottom space to Content View Bottom Margin, value: 8</span></span>

    ![プロトタイプセルのレイアウトのスクリーンショット](./images/prototype-cell-layout.png)

1. <span data-ttu-id="ffc42-150">**Calendarviewcontroller**を開き、その内容を次のコードで置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-150">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="ffc42-151">アプリを実行し、サインインして [**予定表**] タブをタップします。イベントの一覧が表示されます。</span><span class="sxs-lookup"><span data-stu-id="ffc42-151">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![イベント表のスクリーンショット](./images/calendar-list.png)
