# CSS 筆記、建議與指導方針總整理

---

在參與大規模、歷時漫長且人手眾多的專案時，所有網頁開發人員都能遵守以下原則極為重要：

+ **維持 CSS 樣式的可維護性 (maintainable)**
+ **維持撰寫風格清晰明瞭並具可讀性 (readable)**
+ **維持 CSS 樣式的延展性 (scalable)**

為了達成上述原則，我們必須使用許多方法才能達成這個目標。

本文第一部分將探討語法、格式與 CSS 剖析；第二部分將從方法論 (approach)、思維框架 (mindframe) 與架構 CSS 的見解著手。

## 內容大綱

* [剖析 CSS 文件](#css-document-anatomy)
  * [總覽](#general)
  * [單一檔案與多檔案](#one-file-vs-many-files)
  * [目錄大綱](#table-of-contents)
  * [區段標題](#section-titles)
* [樣式載入順序](#source-order)
* [規則解析](#anatomy-of-rulesets)
* [命名規範](#naming-conventions)
  * [HTML 中的 class 屬性](#class-in-html)
  * [JS 鉤子 (JS hooks)](#js-hooks)
  * [i18n](#internationalisation)
* [註解](#comments)
  * [編寫註解的技巧](#comments-on-steroids)
    * [使用看似合法的選取器 (Quasi-qualified selectors)](#quasi-qualified-selectors)
    * [替樣式加上特殊的標籤 (Tagging code)](#tagging-code)
    * [用物件繼承的方式註記 (Object/extension pointers)](#objectextension-pointers)
* [撰寫 CSS](#writing-css)
* [建構新組件 (Building new components)](#building-new-components)
* [物件導向 CSS (OOCSS)](#oocss)
* [版面配置](#layout)
* [調整 UI 的尺寸](#sizing-uis)
  * [字級大小](#font-sizing)
* [簡寫](#shorthand)
* [關於 ID 的使用](#ids)
* [選取器](#selectors)
  * [過修飾選取器](#over-qualified-selectors)
  * [選取器效能](#selector-performance)
* [使用 CSS 選取器的目的](#css-selector-intent)
* [`!important`](#important)
* [魔數與絕對比例](#magic-numbers-and-absolutes)
* [條件式註解](#conditional-stylesheets)
* [偵錯(Debugging)](#debugging)
* [前置處理器](#preprocessors)

---

<a name="css-document-anatomy"></a>
## 剖析 CSS 文件

無論撰寫什麼文件，我們都應該盡量維持一致的風格，包括一致的註解、一致的語法與一致的命名規範。

<a name="general"></a>
### 總則

盡量將每行寬度控制在 80 個字元以下。漸變（gradient）相關的語法與註解中的 URL 等可以當作例外，畢竟這部分我們也無能為力。

我傾向於用 4 個空白字元來縮排，而非使用 Tab 字元，並且習慣將不同的樣式拆分成多行。

<a name="one-file-vs-many-files"></a>
### 單檔案與多檔案

有些人喜歡將樣式表寫在一個很大的檔案裡，這還不錯，而且如果你按照下文的規則來撰寫的話，也不會遇到什麼問題。我在轉換到 Sass 之後，開始將樣式拆分成許多小檔案，這其實也是個不錯的選擇。但無論你採用什麼方式，下文所描述的規則，也依然適用。這兩種寫法僅僅在目錄以及區段標題上有所差異而已。

<a name="table-of-contents"></a>
### 目錄大綱 (註解)

在 CSS 檔案的開頭，我會寫一份目錄 (以註解形式撰寫)，例如：

    /*------------------------------------*\
        $CONTENTS
    \*------------------------------------*/
    /**
     * CONTENTS............You're reading it!
     * RESET...............Set our reset defaults
     * FONT-FACE...........Import brand font files
     */

這份目錄可以告訴其他網頁開發人員，這份樣式表具體包含哪些內容。這份目錄中的每一項標題，都應該與對應的區段標題相同。

如果你在維護一份規模較大的 CSS 樣式表檔案，對應的區塊也會在同一個檔案裡。如果你在維護的是一份小型的 CSS 檔案，那麼目錄中的每一項，也應該要有相對應的 @include 語句。

<a name="section-titles"></a>
### 區段標題 (註解)

從目錄大綱對應的區塊標題，其範例如下：

    /*------------------------------------*\
        $RESET
    \*------------------------------------*/

在區段標題使用前綴 `$` 可以方便我們使用（[Cmd|Ctrl]+F）命令搜尋 `$[SECTION-NAME]`，最主要的目的則是 **將搜尋範圍限制在區段標題中**，不會搜尋到其他的 CSS 關鍵字。

如果你維護的是一份大型的 CSS 樣式表，那麼建議在區段與區段之間間隔 5 行，範例如下：

    /*------------------------------------*\
        $RESET
    \*------------------------------------*/
    [Our
    reset
    styles]





    /*------------------------------------*\
        $FONT-FACE
    \*------------------------------------*/

在大型的 CSS 樣式表之間快速捲動時，這些間距較大的區塊，有助於我們在視覺上區分不同的區塊，以增加可讀性。

如果你在維護多份以 @include 連接的 CSS 樣式表，那麼在每個檔案的檔頭加上區段標題即可，不必像這樣額外空行。

<a name="source-order"></a>
## 樣式編寫順序

盡量按照特定順序編寫樣式規則，這可確保你充分發揮 CSS 縮寫中第一個字母 <i>C</i> 的意義：Cascade〔串聯〕。

一份妥善規劃的 CSS 應該按照如下順序撰寫：

1. **Reset** – 重置所有預設樣式，將許多元素預設的 margin, padding, border 都先歸零
2. **Elements** – 重新定義預設元素的樣式，也就是那些沒有設定 class 屬性的 `h1`、`ul` 等元素
3. **Objects and abstractions** – 定義那些通用的，或套用一些基礎設計模式的樣式
4. **Components** – 將不同物件與擴充組合而成的完整元件
5. **Style trumps** – error states etc.

如此一來，當你依序撰寫 CSS 的時候，每個區塊都可以自動繼承之前定義過的樣式屬性。這樣就可以減少樣式之間互相衝突或需要重新定義的部分，也可以減少某些特定的問題，建構出更完美的 CSS 結構。

關於這方面的更多訊息，強烈推薦 Jonathan Snook 的 [SMACSS](http://smacss.com)。

<a name="anatomy-of-rulesets"></a>
## 解剖 CSS 規則集

    [選取器] {
        [屬性]:[值];
        [<- 其他樣式 ->]
    }

撰寫 CSS 樣式時，我習慣遵守這些規則：

* class 名稱以減號（-）連接，除了下文提到的 BEM 表示法
* 預設縮排 4 個空白字元
* 將不同的樣式拆分成多行
* 不同的樣式以 **相關性** 的順序進行排列，而非以字母順序
* 有瀏覽器前綴(Vendor-specific Properties)的樣式要適當縮排，將主要的名稱對齊，其值也會對齊
* 不同樣式之間應該要適當縮排，適當反映 DOM 的結構
* 在樣式中保留最後一條樣式規則的結尾分號

一個完整的範例如下：

    .widget{
        padding:10px;
        border:1px solid #BADA55;
        background-color:#C0FFEE;
        -webkit-border-radius:4px;
           -moz-border-radius:4px;
                border-radius:4px;
    }
        .widget-heading{
            font-size:1.5rem;
            line-height:1;
            font-weight:bold;
            color:#BADA55;
            margin-right:-10px;
            margin-left: -10px;
            padding:0.25em;
        }

我們可以發現，`.widget-heading` 是 `.widget` 的子元素，因為前者比後者多縮排了一級，這使網頁開發人員在閱讀這些樣式時，可以迅速得知在 HTML 裡面的標籤結構大致為何。

我們還可以發現 `.widget-heading` 的樣式是根據其 **相關性** 排列的，例如 `.widget-heading` 會套用在文字的元素上，所以我們先新增字體相關的樣式，接下來是其它的。

但這裡有個例外，如下是一個沒有將樣式規則拆分成多行的例子：

    .t10    { width:10% }
    .t20    { width:20% }
    .t25    { width:25% }       /* 1/4 */
    .t30    { width:30% }
    .t33    { width:33.333% }   /* 1/3 */
    .t40    { width:40% }
    .t50    { width:50% }       /* 1/2 */
    .t60    { width:60% }
    .t66    { width:66.666% }   /* 2/3 */
    .t70    { width:70% }
    .t75    { width:75% }       /* 3/4*/
    .t80    { width:80% }
    .t90    { width:90% }

在這個例子（來自[inuit.css's table grid system](https://github.com/csswizardry/inuit.css/blob/master/inuit.css/partials/base/_tables.scss#L88)），你可以發現把這些樣式規則擺在一行內是比較清楚的。

<a name="naming-conventions"></a>
## 命名規範

一般情況下我都是以減號（-）連接 class 的名字（例如 `.foo-bar` 而非 `.foo_bar` 或 `.fooBar`），不過在某些情況下，我會用 BEM（Block, Element, Modifier）表示法。

<abbr title="Block, Element, Modifier">BEM</abbr> 表示法可以讓選取器更加嚴謹、更加清晰、所表達的資訊也更豐富。

這個 <abbr title="Block, Element, Modifier">BEM</abbr> 表示法的格式大致如下：

    .block{}
    .block__element{}
    .block--modifier{}

其中：

* `.block` 代表某個較為高階或較為抽象的樣式定義
* `.block__element` 代表是套用在 `.block` 下的一個子元素
* `.block--modifier` 代表 `.block` 在不同狀態下的樣式

舉例來說：

    .person{}
    .person--woman{}
        .person__hand{}
        .person__hand--left{}
        .person__hand--right{}

這個例子，我們描述的最上層的元素是一個人，然後這個人可能是一個女人。我們還知道人擁有手，這些是人體的一部分，而手也有不同的狀態，例如左手與右手。

這樣我們就可以根據上層元素來定義選取器的命名空間，並從名稱就能傳達出該選取器的主要功能，看它是一個子元素（`__`），還是不同狀態（`--`）？

這時你若看到 `.page-wrapper` 則代表一個獨立的選取器，因為它不是某個元素下的子元素，或某個元素的某個狀態；然而如果你有一個 `.widget-heading` 樣式，而他其實跟 `.widget` 有關聯，是 `.widget` 的子元素，那麼我們應該將這個樣式重新命名為 `.widget__heading` 才對。

BEM 表示法雖然有點醜，而且有點囉嗦，但是它使得我們可以透過名稱快速得知元素的功能，以及元素之間的關係。除此之外，BEM 語法中的重複部分，其實非常利於 gzip 壓縮，名稱重複的部分會自動被 gzip 壓縮後剔除，並不會耗用頻寬。

無論你是否使用 BEM 表示法，你都應該確保 class 命名得當，確保一字不多、一字不少，也確保類別樣式的命名抽象一點，以提高複用性（例如 `.ui-list`，`.media`）。那些因為特定目而設計的樣式，在命名時則要盡量精準（例如 `.user-avatar-link`），你不用擔心類別名稱的數量太多或字串長度過長，因為 gzip 壓縮演算法會很驚人的幫你把重複的文字進行壓縮。

<a name="class-in-html"></a>
### HTML 中的 class 屬性

為了確保可讀性，建議在 HTML 標籤的 class 屬性中，使用 2 個空白字元間隔不同的 class 名稱，例如：

    <div class="foo--bar  bar__baz">

增加的空白字元應該可以讓你在閱讀一個 class 屬性中的類別名稱時，更易於閱讀。

<a name="js-hooks"></a>
### JS hooks

**千萬不要把 CSS 樣式當成 JS hooks 來用。**我們在寫 jQuery 的時候，經常會自訂一些 class 樣式類別名稱，以方便我們透過 jQuery 的選取器選中這個元素。除此之外，有時候我們也會自訂一些 HTML 屬性，讓 HTML 擁有一些特殊的行為，這些都算是 JS hooks 的應用，如果你把 JS 的行為與樣式綁在一起時，代表我們套用的樣式與 JavaScript 行為無法區分開來，這對可維護性來說也蠻傷的。

如果你要把 JS 行為與某些標籤綁定起來的話，寫一個 JS 專用的 class 類別名稱。簡單地說就是在名稱上增加一個前綴 `.js-` 的命名空間，例如 `.js-toggle`，`.js-drag-and-drop`，這意味著我們可以透過不同的 class 綁定不同的 JS 行為和 CSS 樣式，而不會為偶發的衝突帶來困擾，範例如下：

    <th class="is-sortable  js-is-sortable">
    </th>

上面的這個 th 標籤有兩個 class，你可以用 is-sortable 這個類別來定義這個表格的樣式，而用另一個 js-is-sortable 來套用排序功能。

<a name="internationalisation"></a>
### i18n

雖然我（該 CSS Guideline 文件原作者 Harry Roberts）是個英國人，而且我一向把 **顏色** 拼寫成 <i>colour</i> 而非 <i>color</i>，但是為了命名的一致性，我認為在 CSS 中使用美式英語的拼法更好。CSS 以及其它多數語言都是以美式英語的拼法寫成，所以如果在 `.colour-picker{}` 中寫 `color:red` 就缺乏一致性。我以前主張同時用兩種拼法，例如：

    .color-picker,
    .colour-picker{
    }

但是我最近參與了一份規模龐大的 Sass 專案，這個專案中有許多的顏色變數（例如 `$brand-color`，`$highlight-color` 等等），每個變數要維護兩種拼法實在辛苦，要搜尋並替換時也需要兩倍的工作量。

所以為了一致性，把所有的 class 與變數都以你參與的專案的慣用拼法命名即可。

<a name="comments"></a>
## 註解

我使用每行寬度不超過 80 個字元的區塊註解：

    /**
     * 這是一個 docBlock 風格的註解。
     *
     * 這裡是一個詳細且完整的說明文字，用來更進一步的說明程式碼。
     * 當然，我們要把每行寬度控制在 80 個字元以內。
     *
     * 我們可以在註解中嵌入 HTML 標記，而且也建議這麼做：
     *
        <div class=foo>
            <p>Lorem</p>
        </div>
     *
     * 如果是註解內嵌標籤的話，我選擇不在它前面加上星號，否則要複製貼上時還挺麻煩的。
     */

在註解中應該盡量詳細描述你的語法，因為對你來說清晰易懂的內容，對其他人可能並非如此。所以，建議每寫一部分樣式後，就要立刻編寫註解。

<a name="comments-on-steroids"></a>
### 編寫註解的技巧

註解其實有許多先進的用法，例如：

* 使用看似合法的選取器 (Quasi-qualified selectors)
* 替樣式加上特殊的標籤 (Tagging code)
* 用物件繼承的方式註記 (Object/extension pointers)

<a name="quasi-qualified-selectors"></a>
#### 使用看似合法的選取器 (Quasi-qualified selectors)

你應該避免過分修飾選取器，例如如果你能寫 `.nav{}` 就盡量不要寫 `ul.nav{}`。過分修飾選取器會影響網頁效能，影響 class 的複用性，也會增加選取器的權重 (Specificity)，這些都是你應該竭力避免的。

不過，有時你可能希望告訴其他網頁開發人員 class 的使用範圍。以 `.product-page` 為例，這個 class 看起來像是一個較為上層的容器，可能是 `html` 或者 `body` 之類的元素，但是僅憑 `.product-page` 並無法有效判斷他會套用在哪個元素上。

我們可以在選取器前加上一些註解來修飾這個語法（即將前面的元素選取器註解掉）來描述 class 的作用範圍：

    /*html*/.product-page{}

這樣我們就能準確得知，該 class 的作用範圍，並明確知道這個 class 不具複用性。看 CSS 就能知道這些事，可讀性就大大提升許多。

其它例子如：

    /*ol*/.breadcrumb{}
    /*p*/.intro{}
    /*ul*/.image-thumbs{}

這樣我們就就可清楚地了解到這些樣式實際的套用範圍，而不用擔心它會不會套用到其他元素上。

<a name="tagging-code"></a>
#### 替樣式加上特殊的標籤 (Tagging code)

如果你寫了一個新的樣式規則，可以在它前面套用一些標籤(Tag)，例如：

    /**
     * ^navigation ^lists
     */
    .nav{}

    /**
     * ^grids ^lists ^tables
     */
    .matrix{}

這些標籤可以使得其他網頁開發人員快速找到相關樣式。如果一個網頁開發人員需要搜尋和列表相關的部分，他只要搜尋 `^lists` 就能快速定位到 `.nav`，`.matrix` 以及其它相關部分。

<a name="objectextension-pointers"></a>
#### 用物件繼承的方式註記 (Object/extension pointers)

當物件導向的觀念用在 CSS 的時候，你經常能找到兩段相關的 CSS 樣式定義分散在不同地方（可能其中一個為基底樣式，另一個則為擴充樣式），我們可以用物件繼承的方式註記該樣式與原本樣式之間的關聯。這些在註解中的寫法如下：

也許在你的 base.css 檔案中，可以看到 `.foo` 的定義如下，這裡明確指出會有另一個 `.foo` 被定義在 theme.css 檔案中：

    /**
     * Extend `.foo` in theme.css
     */
     .foo{}

而在 theme.css 檔案中，則能看到一個 `.foo` 的定義，明確告知這是繼承自 base.css 檔案中的 `.foo` 樣式：

    /**
     * Extends `.foo` in base.css
     */
     .foo{}

如此一來，我們就能在兩塊相隔很遠的樣式定義中，建立一個聯繫關係，在搭配好用的開發工具之下，還能快速幫你找到彼此。

---

<a name="writing-css"></a>
## 撰寫 CSS

之前的章節主要探討如何組織我們的 CSS 樣式，這些都是非常量化的規則。接下來我們要探討更理論的東西，也將探討我們的見解與方法論。

<a name="building-new-components"></a>
## 建構新元件 (Building new components)

建構新元件時，必須要在撰寫 CSS **之前** 先寫好 HTML 部分。這可以幫助你準確判斷哪些 CSS 屬性可以繼承，避免重複套用多餘的樣式。

優先撰寫 HTML 可以讓你專注在資料、內容與語意上，而在這之後才新增相關的 class 和 CSS 樣式。

<a name="oocss"></a>
## 物件導向 CSS (OOCSS)

我都是以物件導向的方式撰寫 CSS，我把元件區分成結構（物件）與外觀（擴充）。正如以下想法（注意這個只是想法而非例子）：

    .room{}

    .room--kitchen{}
    .room--bedroom{}
    .room--bathroom{}

我們在屋子裡有許多房間，它們都有共同的特點：它們都包含地板、天花板、牆壁和門。這些共享的部分我們可以放到一個抽象的 `.room{}` class 中。不過我們還有其它與眾不同的房間：一個廚房可能有地磚，臥室可能有地毯，洗手間可能沒有窗戶但是臥室會有，每個房間的牆壁顏色也許也會不一樣。物件導向 CSS 的思考方式，使我們把相同部分抽象出來，並組成結構部分，然後再用更具體的 class 來擴充這些外觀，並新增特殊的處理方法。

所以，與其撰寫大量的元件，倒不如努力找出這些元件中重複的設計模式，並將其抽取出來，寫成一個可以重複使用的 class，然後把這些骨架與基底的 '物件' 透過 class 擴充其樣式，這樣才能套用到各種特殊的使用情境上。

當你要撰寫一個新元件時，先將其拆解成結構和外觀。撰寫結構的部分時，使用通用的 class 以確保這個元件的複用性，撰寫外觀時則使用更具體的 class 來點綴這些視覺上的設計。

<a name="layout"></a>
## 版面配置

所有元件都不應該定義寬度，保持其流動性(fluid)，盡量由上層元素或 **網格系統** (Grid systems) 來決定其寬度。

**永遠不要** 定義元件的高度。高度應該僅用於尺寸已經固定的東西，例如圖片和 CSS Sprite 等等。在 `p`，`ul`，`div` 等元素上不應該定義高度，如果需要的話可以寫 `line-height` 會彈性許多。

**網格系統** 應該用 **書架** 來理解，你一定是拿書架來放書，是先有書(內容)，然後把書放到書架(網格系統)上；而不是把書架當成書，然後再擺到另一個書架上。將網格系統與我們的元件區分開來，將有助於我們更加彈性的配置元件在版面中的位置，也使得我們的前端工作更有效率。

你不應該套用任何樣式在網格系統上，他們單純的只為了版面配置之用。記得要在網格系統內套用樣式到內容上。記得：無論在任何情境下，永遠不要在網格系統的任何一格裡，套用任何 box-model 屬性(margin, padding, border)。

<a name="sizing-uis"></a>
## 調整 UI 的尺寸

我會用各種方法設定 UI 尺寸，包括百分比 (`%`)、`px`、`em`、`rem` ("root em")，如此而已。

理想情況下，網格系統應該用百分比設定。如上所述，因為我用網格系統來固定欄寬和頁寬，所以我可以完全不用理會元素的尺寸，他會自己根據網格系統自由縮放。

我用 `rem` 定義字級大小，並且同時使用 `px` 相容於舊版瀏覽器，這可以讓你在使用 `em` 的情況下，又不用擔心舊版瀏覽器會無法正確顯示。這裡有個好用的 Sass mixin 程式碼片段如下 (假設你可以任意指定字形的預設大小)：

    @mixin font-size($font-size){
        font-size:$font-size +px;
        font-size:$font-size / $base-font-size +rem;
    }

我只會在已經使用固定尺寸的元素上使用 px，包括已經使用 `px` 定義過並自動繼承的圖片或 CSS Sprite。

<a name="font-sizing"></a>
### 字級大小

我會定義一些與網格系統原理類似的 class 來定義字級大小，這些 classes 可以在 double stranded heading hierarchy 的結構下設定字級。關於 double stranded heading hierarchy 的詳細解釋，請參閱 [Pragmatic, practical font-sizing in CSS](http://csswizardry.com/2012/02/pragmatic-practical-font-sizing-in-css) 文章。

<a name="shorthand"></a>
## 簡寫

**使用簡寫的 CSS 語法應該要特別注意。**

或許你會嘗試撰寫像 `background:red;` 這樣的屬性，或許你這樣寫的背後真正意思是 `background-image:none; background-position:top left; background-repeat: repeat; background-color:red;` 這樣的語法，雖然這樣寫通常不會出什麼問題，但是哪怕只出一次問題就值得考慮要不要放棄簡寫了，在這個例子裡，你應該將其改寫為 `background-color:red;` 比較洽當。

類似的情況，像 `margin:0;` 這樣的宣告的確清楚明瞭，但是還是應該 **盡量寫清楚**，如果你只是想修改底部的 `margin`，最好具體一點，寫成 `margin-bottom:0;` 會來的好很多。

你必須把樣式定義的很清楚，不要因為習慣用簡寫，而不小心改到其他樣式的相關屬性。例如你只想改掉底部的 `margin`，那就不要用也會把其它邊距也歸零的 `margin:0` 語法。

簡寫雖然是好東西，但是切忌濫用。

<a name="ids"></a>
## 關於 ID 的使用

在我們開始撰寫選取器之前，牢記這句話：

**在 CSS 裡千萬不要用 ID**

在 HTML 裡 ID 可以用於 JS 以及錨點定位(anchor)，但是在 CSS 裡建議只用 class 來設定樣式，你不會想看到在任何一個樣式表中看見使用 ID 的選取器 (`#someid`)。

使用 class 宣告的好處在於複用性，而且權重也並不高。樣式的權重太高很容易導致問題，所以減少樣式的權重是非常重要的。每個 ID 的權重是 class 的 **255** 倍，所以在 CSS 中請千萬不要使用這種寫法。

<a name="selectors"></a>
##  選取器

請維持選取器簡短、有效率與可攜性。

那些依賴頁面元素來定位的選取器有很多缺點。例如 `.sidebar h3 span{}` 這樣的選取器，就是太過依賴元素的相對位置，所以很難把 span 移到 h3 和 sidebar 外面並維持其樣式。

結構複雜的選取器也會影響網頁顯示效能，選取器結構越複雜（如 `.sidebar h3 span` 為三層，`.content ul p a` 是四層），瀏覽器對於顯示網頁的負擔就越大。

所以，盡量不要讓樣式依賴於其他元素的位置，也盡量讓選取器保持簡短而易懂。

整體來說，選取器應該盡量簡短（例如只有一層就能定位），但是 class 名稱則不應該過於簡略，例如 `.user-avatar` 就遠比 `.usr-avt` 來的好。

**請記得：** class 無所謂是否語意化的問題；你應該關注它們是否合理，不要刻意強調 class 名稱要符合語意，而要注重使用的合理性與未來性。

<a name="over-qualified-selectors"></a>
### 過度修飾的選取器

由前文所述，過度修飾的選取器並不理想。

過度修飾的選取器是指像 `div.promo` 這樣的。很可能你只用 `.promo` 也能得到相同的效果。當然你可能偶爾會需要用元素類型來修飾 class（例如你寫了一個 `.error` 而且想讓它在不同的元素類型中顯示效果不一樣，例如 `.error{ color:red; }` `div.error{ padding:14px;}`），但是大多數時候還是應該盡量避免。

再舉一個修飾過度的選取器例子，`ul.nav li a{}`。如前文所說，我們馬上就可以刪掉 `ul` 因為我們知道 `.nav` 是個列表，然後我們就可以發現 `a` 一定在 `li` 中，所以我們就能將這個選取器改寫成 `.nav a{}`，這樣就可以減少一個層級，增加 CSS 的顯示速度。

<a name="selector-performance"></a>
### 選取器效能

雖然瀏覽器效能日益提升，顯示 CSS 的速度也越來越快，但是你還是應該關注 CSS 的顯示效能。使用簡短、沒有巢狀的選取器，不要使用全域選取器（`*{}`）作為主要的選取器，避免使用更複雜的 CSS3 選取器，都可以讓你避免選取器效能的問題。

<a name="css-selector-intent"></a>
## 使用 CSS 選取器的目的

比起運用選取器定位到某元素，更好的辦法則是直接在你想要新增樣式的元素上新增一個 class，我們以 `.header ul{}` 這樣一個選取器為例。

假設這個 `ul` 就是這個網站的主選單，它位於 header 中，而且目前為止是 header 中唯一的 `ul` 元素。`.header ul{}` 的確可以生效，但是這樣並不是好方法，這種寫法比較沒有未來性，而且也不太明確。如果我們在 header 中再新增一個 `ul` 的話，它就會套用我們給這個主選單寫的樣式，哪怕我們設想的不是這個效果。這意味著我們要麼重構許多樣式，要麼給後面的 `ul` 新增許多樣式來抵消之前的影響。

你的選取器必須符合你要給這個元素新增樣式的原因，思考一下， **「我定位到這個元素，是因為它是 `.header` 下的 `ul`，還是因為它是我網站的主選單？」**這將決定你應該如何使用選取器。

確保你的主要選取器不是元素選取器(element/type selector)或物件/抽象(object/abstraction)的類別。例如在我們的 CSS 中肯定找不到像是 `.sidebar ul{}` 或者 `.footer .media{}` 這樣的選取器。

要明確表達：直接找到你要新增樣式的元素，而非其上層元素。不要想當然地認為 HTML 不會改變。 **用選取器直接命中你需要的元素，而不是目前剛好選中的狀態。**

完整內容請參考我的文章 [Shoot to kill; CSS selector intent](http://csswizardry.com/2012/07/shoot-to-kill-css-selector-intent/)

<a name="important"></a>
## `!important`

你只應該在一些輔助類別(helper classes)上使用 `!important` 修飾子。用 `!important` 提升優先級也可以，例如如果你要讓某條規則 **一直** 生效的話，可以用 `.error{ color:red!important; }`。

避免主動使用 `!important` 修飾子。例如當你的 CSS 寫得很複雜的時候，不要因為想偷懶而使用 `!important` 來取巧，建議重寫你之前寫好的樣式，並重構選取系的使用方式。記得：維持選取器的簡短並且避免用 ID，將可有效幫助你寫好 CSS。

<a name="magic-numbers-and-absolutes"></a>
## 魔數與絕對定位

魔數（Magic Number）是指那些「剛好有效果」的數字，這東西非常不好，因為它們只是治標不治本，而且缺乏延展性。

例如你用 `.dropdown-nav li:hover ul{ top:37px; }` 把下拉選單移動到主選單的下方，因為這裡的 37px 就是個魔數，37px 會生效的原因是因為這時 `.dropbox-nav` 碰巧高 37px 而已。

這時你應該用 `.dropdown-nav li:hover ul{ top:100%; }`，這時無論 `.dropbox-down` 多高，這個下拉選單都會往下移動 100%，這才是明智之舉。

每當你要在樣式中放入數字的時候，請三思而後行。如果你能用一個關鍵字或別名（例如 `top:100%` 意即「從上面拉到最下面」），或有更好的解決方法的話，就盡量避免直接出現數字。

你在 CSS 中留下的每一個數字，都好像在告訴別人說：「這個數字我不是真的很需要」，這是一種不負責任的表現。

<a name="conditional-stylesheets"></a>
## 條件式註解

IE 專屬的樣式定義，基本上都應該避免使用，唯一可以用的時機點就是為了處理舊版 IE 不支援的內容（例如 PNG fixes 的問題）。

一個基本原則是，所有的版面配置與 box-model 的樣式都不需要用到 IE 專屬的樣式。也就是說，你在重構樣式之後，你應該不會想看到 `<!--[if IE 7]> element{ margin-left:-9px; } < ![endif]-->` 或其他類似的語法。

<a name="debugging"></a>
## CSS 偵錯(Debugging)

如果你遇到 CSS 問題的時候， **請先把舊的、有問題的樣式移除後再寫新的** 。如果舊的 CSS 樣式有問題的話，寫再多的新樣式是解決不了問題的。

把 CSS 原始碼和 HTML 部分刪掉，直到沒有 BUG 為止，然後你就知道問題出在哪裡了。

有時候你可能會寫上一個 `overflow:hidden` 把一些破版的問題給隱藏起來，但也許問題根本不出在 overflow 這部分。所以還是切記： **要治本，而不是單純治標而已**。

<a name="preprocessors"></a>
## 前置處理器

我選擇 Sass 當成我撰寫 CSS 時的前置處理器，請 **靈活運用** 這類前置處理器。用 Sass 可以令你的 CSS 更強大，但是不要用的太複雜。例如在 [Vanilla CSS](http://www.vanillacss.com/) 裡面，只需在必要的地方套用樣式層級即可：

    .header{}
    .header .site-nav{}
    .header .site-nav li{}
    .header .site-nav li a{}

這樣的寫法在一般的 CSS 裡完全用不到，所以以下範例就是個 **不好的** Sass 寫法：

    .header{
        .site-nav{
            li{
                a{}
            }
        }
    }

如果你用 Sass 的話，應該改寫為：

    .header{}
    .site-nav{
        li{}
        a{}
    }
