<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="65585-101">このセクションでは、ユーザーの予定表にイベントを作成する機能を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="65585-102">**GraphManager.swift を** 開き、次の関数を追加して、ユーザーの予定表に新しいイベントを作成します。</span><span class="sxs-lookup"><span data-stu-id="65585-102">Open **GraphManager.swift** and add the following function to create a new event on the user's calendar.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="CreateEventSnippet":::

1. <span data-ttu-id="65585-103">**GraphTu touchl** **フォルダーに新** しい Cocoa Touch Class ファイルを作成します `NewEventViewController` 。</span><span class="sxs-lookup"><span data-stu-id="65585-103">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `NewEventViewController`.</span></span> <span data-ttu-id="65585-104">フィールド **のサブクラスで UIViewController** **を選択** します。</span><span class="sxs-lookup"><span data-stu-id="65585-104">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="65585-105">**NewEventViewController.swift を開き**、その内容を次の内容に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="65585-105">Open **NewEventViewController.swift** and replace its contents with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/NewEventViewController.swift" id="NewEventViewControllerSnippet":::

1. <span data-ttu-id="65585-106">**Main.storyboard を開きます**。</span><span class="sxs-lookup"><span data-stu-id="65585-106">Open **Main.storyboard**.</span></span> <span data-ttu-id="65585-107">ライブラリを **使用して** 、ビュー **コントローラーをストーリーボード** にドラッグします。</span><span class="sxs-lookup"><span data-stu-id="65585-107">Use the **Library** to drag a **View Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="65585-108">ライブラリを **使用して**、ビュー **コントローラーにナビゲーション バー** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-108">Using the **Library**, add a **Navigation Bar** to the view controller.</span></span>
1. <span data-ttu-id="65585-109">ナビゲーション バーでタイトル **を** ダブルクリックし、次の値に更新します `New Event` 。</span><span class="sxs-lookup"><span data-stu-id="65585-109">Double-click the **Title** in the navigation bar and update it to `New Event`.</span></span>
1. <span data-ttu-id="65585-110">ライブラリを **使用** して、ナビゲーション バー **の** 左側にバー ボタン項目を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-110">Using the **Library**, add a **Bar Button Item** to the left-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="65585-111">新しいバー ボタンを選択し、属性インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="65585-111">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="65585-112">Title **を次に変更** します `Cancel` 。</span><span class="sxs-lookup"><span data-stu-id="65585-112">Change **Title** to `Cancel`.</span></span>
1. <span data-ttu-id="65585-113">ライブラリを **使用** して、ナビゲーション バー **の** 右側にバー ボタン項目を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-113">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="65585-114">新しいバー ボタンを選択し、属性インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="65585-114">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="65585-115">Title **を次に変更** します `Create` 。</span><span class="sxs-lookup"><span data-stu-id="65585-115">Change **Title** to `Create`.</span></span>
1. <span data-ttu-id="65585-116">ビュー コントローラーを選択し、Id インスペクター **を選択します**。</span><span class="sxs-lookup"><span data-stu-id="65585-116">Select the view controller, then select the **Identity Inspector**.</span></span> <span data-ttu-id="65585-117">クラス **を** **NewEventViewController に変更します**。</span><span class="sxs-lookup"><span data-stu-id="65585-117">Change **Class** to **NewEventViewController**.</span></span>
1. <span data-ttu-id="65585-118">次のコントロールをライブラリから **ビュー** に追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-118">Add the following controls from the **Library** to the view.</span></span>

    - <span data-ttu-id="65585-119">ナビゲーション バー **の下にラベル** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-119">Add a **Label** under the navigation bar.</span></span> <span data-ttu-id="65585-120">テキストを次に設定します `Subject` 。</span><span class="sxs-lookup"><span data-stu-id="65585-120">Set its text to `Subject`.</span></span>
    - <span data-ttu-id="65585-121">ラベルの **下にテキスト フィールド** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-121">Add a **Text Field** under the label.</span></span> <span data-ttu-id="65585-122">Placeholder 属性 **を次に設定** します `Subject` 。</span><span class="sxs-lookup"><span data-stu-id="65585-122">Set its **Placeholder** attribute to `Subject`.</span></span>
    - <span data-ttu-id="65585-123">テキスト フィールド **の下にラベル** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-123">Add a **Label** under the text field.</span></span> <span data-ttu-id="65585-124">テキストを次に設定します `Attendees` 。</span><span class="sxs-lookup"><span data-stu-id="65585-124">Set its text to `Attendees`.</span></span>
    - <span data-ttu-id="65585-125">ラベルの **下にテキスト フィールド** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-125">Add a **Text Field** under the label.</span></span> <span data-ttu-id="65585-126">Placeholder 属性 **を次に設定** します `Separate multiple entries with ;` 。</span><span class="sxs-lookup"><span data-stu-id="65585-126">Set its **Placeholder** attribute to `Separate multiple entries with ;`.</span></span>
    - <span data-ttu-id="65585-127">テキスト フィールド **の下にラベル** を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-127">Add a **Label** under the text field.</span></span> <span data-ttu-id="65585-128">テキストを次に設定します `Start` 。</span><span class="sxs-lookup"><span data-stu-id="65585-128">Set its text to `Start`.</span></span>
    - <span data-ttu-id="65585-129">ラベルの **下に日付の選択** コントロールを追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-129">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="65585-130">[優先 **するスタイル] を** **[コンパクト] に** 設定し、**間隔\*\*\*\*を 15 分** に設定し、高さを **35 に設定します**。</span><span class="sxs-lookup"><span data-stu-id="65585-130">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="65585-131">日付の選択 **コントロールの** 下にラベルを追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-131">Add a **Label** under the date picker.</span></span> <span data-ttu-id="65585-132">テキストを次に設定します `End` 。</span><span class="sxs-lookup"><span data-stu-id="65585-132">Set its text to `End`.</span></span>
    - <span data-ttu-id="65585-133">ラベルの **下に日付の選択** コントロールを追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-133">Add a **Date Picker** under the label.</span></span> <span data-ttu-id="65585-134">[優先 **するスタイル] を** **[コンパクト] に** 設定し、**間隔\*\*\*\*を 15 分** に設定し、高さを **35 に設定します**。</span><span class="sxs-lookup"><span data-stu-id="65585-134">Set its **Preferred Style** to **Compact**, its **Interval** to **15 minutes**, and its height to **35**.</span></span>
    - <span data-ttu-id="65585-135">日付の選択 **コントロールの下に** テキスト ビューを追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-135">Add a **Text View** under the date picker.</span></span>

1. <span data-ttu-id="65585-136">新しい **イベント ビュー コントローラーを選択し** 、 **接続インスペクターを使用して** 次の接続を行います。</span><span class="sxs-lookup"><span data-stu-id="65585-136">Select the **New Event View Controller** and use the **Connection Inspector** to make the following connections.</span></span>

    - <span data-ttu-id="65585-137">受信した **取り消** し操作を [キャンセル バー **] ボタンに** 接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-137">Connect the **cancel** received action to the **Cancel** bar button.</span></span>
    - <span data-ttu-id="65585-138">**createEvent 受信アクションを**[バーの作成]**ボタンに** 接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-138">Connect the **createEvent** received action to the **Create** bar button.</span></span>
    - <span data-ttu-id="65585-139">件名の **出口を** 最初のテキスト フィールドに接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-139">Connect the **subject** outlet to the first text field.</span></span>
    - <span data-ttu-id="65585-140">出席者の **出口を 2** 番目のテキスト フィールドに接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-140">Connect the **attendees** outlet to the second text field.</span></span>
    - <span data-ttu-id="65585-141">最初の **日付の** 選択に開始の出口を接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-141">Connect the **start** outlet to the first date picker.</span></span>
    - <span data-ttu-id="65585-142">最後の出口 **を** 2 番目の日付の選択に接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-142">Connect the **end** outlet to the second date picker.</span></span>
    - <span data-ttu-id="65585-143">本文の **出口を** テキスト ビューに接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-143">Connect the **body** outlet to the text view.</span></span>

1. <span data-ttu-id="65585-144">次の制約を追加します。</span><span class="sxs-lookup"><span data-stu-id="65585-144">Add the following constraints.</span></span>

    - <span data-ttu-id="65585-145">**ナビゲーション バー**</span><span class="sxs-lookup"><span data-stu-id="65585-145">**Navigation Bar**</span></span>
        - <span data-ttu-id="65585-146">セーフ エリアへの先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-146">Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="65585-147">セーフ エリアへの末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-147">Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="65585-148">トップスペースからセーフ エリア、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-148">Top space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="65585-149">高さ、値: 44</span><span class="sxs-lookup"><span data-stu-id="65585-149">Height, value: 44</span></span>
    - <span data-ttu-id="65585-150">**件名ラベル**</span><span class="sxs-lookup"><span data-stu-id="65585-150">**Subject Label**</span></span>
        - <span data-ttu-id="65585-151">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-151">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-152">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-152">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-153">ナビゲーション バーの上のスペース、値: 20</span><span class="sxs-lookup"><span data-stu-id="65585-153">Top space to Navigation Bar, value: 20</span></span>
    - <span data-ttu-id="65585-154">**[件名] テキスト フィールド**</span><span class="sxs-lookup"><span data-stu-id="65585-154">**Subject Text Field**</span></span>
        - <span data-ttu-id="65585-155">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-155">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-156">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-156">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-157">件名ラベルの上のスペース、値: Standard</span><span class="sxs-lookup"><span data-stu-id="65585-157">Top space to Subject Label, value: Standard</span></span>
    - <span data-ttu-id="65585-158">**出席者ラベル**</span><span class="sxs-lookup"><span data-stu-id="65585-158">**Attendees Label**</span></span>
        - <span data-ttu-id="65585-159">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-159">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-160">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-160">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-161">件名テキスト フィールドの上のスペース、値: Standard</span><span class="sxs-lookup"><span data-stu-id="65585-161">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="65585-162">**Attendees テキスト フィールド**</span><span class="sxs-lookup"><span data-stu-id="65585-162">**Attendees Text Field**</span></span>
        - <span data-ttu-id="65585-163">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-163">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-164">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-164">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-165">出席者ラベルの上のスペース、値: 標準</span><span class="sxs-lookup"><span data-stu-id="65585-165">Top space to Attendees Label, value: Standard</span></span>
    - <span data-ttu-id="65585-166">**開始ラベル**</span><span class="sxs-lookup"><span data-stu-id="65585-166">**Start Label**</span></span>
        - <span data-ttu-id="65585-167">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-167">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-168">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-168">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-169">件名テキスト フィールドの上のスペース、値: Standard</span><span class="sxs-lookup"><span data-stu-id="65585-169">Top space to Subject Text Field, value: Standard</span></span>
    - <span data-ttu-id="65585-170">**開始日の選択**</span><span class="sxs-lookup"><span data-stu-id="65585-170">**Start Date Picker**</span></span>
        - <span data-ttu-id="65585-171">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-171">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-172">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-172">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-173">出席者ラベルの上のスペース、値: 標準</span><span class="sxs-lookup"><span data-stu-id="65585-173">Top space to Attendees Label, value: Standard</span></span>
        - <span data-ttu-id="65585-174">高さ、値: 35</span><span class="sxs-lookup"><span data-stu-id="65585-174">Height, value: 35</span></span>
    - <span data-ttu-id="65585-175">**ラベルの終了**</span><span class="sxs-lookup"><span data-stu-id="65585-175">**End Label**</span></span>
        - <span data-ttu-id="65585-176">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-176">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-177">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-177">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-178">[開始日の選択] の上のスペース、値: 標準</span><span class="sxs-lookup"><span data-stu-id="65585-178">Top space to Start Date Picker, value: Standard</span></span>
    - <span data-ttu-id="65585-179">**終了日の選択**</span><span class="sxs-lookup"><span data-stu-id="65585-179">**End Date Picker**</span></span>
        - <span data-ttu-id="65585-180">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-180">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-181">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-181">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-182">トップスペースからエンド ラベル、値: Standard</span><span class="sxs-lookup"><span data-stu-id="65585-182">Top space to End Label, value: Standard</span></span>
        - <span data-ttu-id="65585-183">高さ: 35</span><span class="sxs-lookup"><span data-stu-id="65585-183">Height: 35</span></span>
    - <span data-ttu-id="65585-184">**本文テキスト ビュー**</span><span class="sxs-lookup"><span data-stu-id="65585-184">**Body Text View**</span></span>
        - <span data-ttu-id="65585-185">表示余白への先頭のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-185">Leading space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-186">ビューの余白への末尾のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-186">Trailing space to View margin, value: 0</span></span>
        - <span data-ttu-id="65585-187">トップスペースから終了日の選択、値: Standard</span><span class="sxs-lookup"><span data-stu-id="65585-187">Top space to End Date Picker, value: Standard</span></span>
        - <span data-ttu-id="65585-188">表示余白の下のスペース、値: 0</span><span class="sxs-lookup"><span data-stu-id="65585-188">Bottom space to View margin, value: 0</span></span>

    ![ストーリーボードの新しいイベント フォームのスクリーンショット](images/new-event-form.png)

1. <span data-ttu-id="65585-190">予定表シーン **を選択し**、接続インスペクタ **ーを選択します**。</span><span class="sxs-lookup"><span data-stu-id="65585-190">Select the **Calendar Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="65585-191">**Triggered Segues の下** で、手動の横にある未入力の円をストーリーボードの新しいイベント ビュー **コントローラー** にドラッグします。</span><span class="sxs-lookup"><span data-stu-id="65585-191">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **New Event View Controller** on the storyboard.</span></span> <span data-ttu-id="65585-192">ポップアップ **メニューで [モーダルに** 表示] を選択します。</span><span class="sxs-lookup"><span data-stu-id="65585-192">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="65585-193">追加したセグを選択し、属性インスペクターを **選択します**。</span><span class="sxs-lookup"><span data-stu-id="65585-193">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="65585-194">[識別子] **フィールドを** 次に設定します `showEventForm` 。</span><span class="sxs-lookup"><span data-stu-id="65585-194">Set the **Identifier** field to `showEventForm`.</span></span>
1. <span data-ttu-id="65585-195">**showNewEventForm 受信アクションを** ナビゲーション バー ボタン **+** に接続します。</span><span class="sxs-lookup"><span data-stu-id="65585-195">Connect the **showNewEventForm** received action to the **+** navigation bar button.</span></span>
1. <span data-ttu-id="65585-196">変更内容を保存し、アプリを再起動します。</span><span class="sxs-lookup"><span data-stu-id="65585-196">Save your changes and restart the app.</span></span> <span data-ttu-id="65585-197">予定表ページに移動し、ボタンをタップ **+** します。</span><span class="sxs-lookup"><span data-stu-id="65585-197">Go to the calendar page and tap the **+** button.</span></span> <span data-ttu-id="65585-198">フォームに入力し、[作成] **をタップ** して新しいイベントを作成します。</span><span class="sxs-lookup"><span data-stu-id="65585-198">Fill in the form and tap **Create** to create a new event.</span></span>

    ![新しいイベント フォームのスクリーンショット](images/create-event.png)
