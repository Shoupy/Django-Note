# Django-Note

## 1. 新增 Django Project
如果想要新增一個project mypage
在command line中移到想要建立project的基底資料夾，然後輸入 django-admin startproject mypage

<pre><code> django-admin startproject mypage </code></pre>

Django會自動建立一個資料夾mypage，並創建相關的內容

## 2. 啟動local server
在基底資料夾中，可以看到第二層mypage資料夾，以及manage.py，這是django自動建立的系統操作檔，只要使用，不需去編碼他。
在Command line中進入基底資料夾(有manage.py的那層)，然後輸入
<pre><code>
> python manage.py runserver
</code></pre>

之後會顯示如下結果：
<pre><code>
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
June 25, 2022 - 07:37:53
Django version 3.2.5, using settings 'mypage.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
</code></pre>

之後在瀏覽器上進入http://127.0.0.1:8000/ 應可順利看到Django範本葉面

如要結束server running。可以在Command line頁面中按下 Ctrl + c

## 3. 新增App
Django 中每一個功能/頁面不是直接寫出來，而是以模組(module)的形式完成，然後再呼叫模組。
當然也可以一個網站只有一個app，所有功能都放在同一個app中
不過這樣後續會蠻難維持的，通常一個特定的項目就會寫成一個 app，讓我們可以比較有條理地整理網站的每個項目。

現在來新增第一個app，在command line中進入基底資料夾(有manage.py)，輸入：

<pre><code> 
> python manage.py startapp challenges 
</code></pre>

這樣一來我們就建立了一個叫做challenges的app.

仔細看一下基底目錄，會發現建立了一個新的資料夾"challenges"，下面有一些python檔
<pre><code> 
__init__.py
admin.py
apps.py
models.py
tests.py
views.py
</code></pre>

現階段大致還不用擔心，之後會一一使用到，我們先從views.py開始。

## 4. URL & Views

### 甚麼是URL
URL 讓使用者鍵入後，能確定連接到某個特定的結果(頁面)

### 甚麼是Views
Views可以說是連接不同的URL後，網站啟動的邏輯，可能是一個function或class
接著程式會處理使用者的request，並且做出類似的回應
- 載入資料
- 載入另一個工作(logic)
- 準備並返回一個response data (例如: HTML)

### 建立第一個view
因為前面是隨便建立的，我們重新建立一個project，叫做"monthly_challenges"，也當作複習前面的操作
<pre><code> 
> django-admin startproject monthly_challenges
> cd monthly_challenges
> python manage.py startapp challenges 
</code></pre>

我們的第一個目標是，讓連結網址到 "./challenges/january" 的時候可以看到一月的挑戰

開啟 資料夾: /challenges/views.py
我們希望建立一個view，如前面所說，基本上等同於建立一個 function，暫時這個view function為 index好了
我們希望有"request"進來的時候才執行，因此給這個function一個parameter
並且最終希望function給出一個response顯示特定的網頁，這裡我們使用Django已經內建的函式 HttpResponse()
(記得import!)
最終我們新增如下的function
<pre><code> 
from django.http import HttpResponse

def index(request):
    return HttpResponse("This works!")
</code></pre>

### 將view連接到URL
現在我們已經新增好了第一個view function!問題是，網站並不知道何時要使用這個index function
這時就需要在challenges資料夾中新建一個urls.py 
路徑會如下 ./challenges/urls.py

打開以後我們首先import path 這個function
<pre><code> 
from django.urls import path  
</code></pre>
path 的本意就是讓知道特定網域地址要呼叫甚麼樣的function
舉例而言，我們的可能希望讓 https://主網址/challenges/january  這個url顯示我們剛剛的view
那麼要在此處加上urlpatterns並使用path，像這樣:

<pre><code> 
urlpatterns = [
    path("challenges/january", 顯示view)
    ]
</code></pre>

不過因為我們是在challenges app裡面，所以這邊的path不用這麼複雜，可以改為january就好。
另外 view的顯示，也要透過匯入的方式把app中的view.py匯入
因此最終整個程式檔如下：
<pre><code> 
from django.urls import path
from . import views

urlpatterns= [
    path("january", views.index)
]
</code></pre>

這樣我們就建立了一個 urlconf (url configuration)了。
當然還差一小個步驟，記得我們剛剛是在challenges裡面編寫嗎?
因此要讓url先指引到challenges裡面。
這時打開主資料夾中monthly_challenges的部分 找到urls.py  (位置 ./monthly_challenges/urls.py)
這個檔案應該已經被django建立好了
裡面一開始就呈現這樣
<pre><code> 
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]
</code> </pre>

這時我們import include 函式，include的意義是讓url後面有challenges時，納入challenges app中的urls.py去進一步搜尋，因此程式碼變成這樣
<pre><code> 
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('challenges/', include("challenges.urls"))
]

</code> </pre>

現在我們在回到command line 啟動server一次
<pre><code> 
> python manage.py runserver
</code> </pre>

然後用瀏覽器打開網址 : http://127.0.0.1:8000/challenges/january

應該可以看到螢幕顯示: "This works!"

### 建立其他幾個月份的views
自行做做看，不過記得function不要重複
因此我們把views.py中的index function重新命名為 january
後面的月份加上february, march....

這樣views.py 中看起會像下面:
<pre><code> 
def january(request):
    return HttpResponse("Start a new year!")

def february(request):
    return HttpResponse("Wake up early every day!")

def march(request):
    return HttpResponse('learning Django 20 minutes every day!')
</code> </pre>

challenges.py看起來則像:
<pre><code> 
urlpatterns= [
    path("january", views.january),
    path("february", views.february),
    path("march", views.march)
]
</code></pre>


### 建立有彈性的view (Dynamic)
在月份的任務中我們知道總共要創建多少view function(而且很幸運的只有12個)
不過才建了3個月分可以看出這樣非常的累，甚至在有的狀況下，你建立apps的時候並不知道最後會有多少function，每次都要修改urls.py和views.py
我們可以用一些方式來達成較有彈性，首先在challenges/urls.py中我們改成
<pre><code> 
urlpatterns= [
    path("&lt;month&gt;", views.monthly_challenges)
]
</code></pre>
"&lt;" 和 "&gt;" 框起來的中間對Django而言表示我們不在乎這裡面填寫了甚麼(這邊用month只是個parameter, 以自己可以理解的形式命名)，只要是這樣的形式我們都招喚monthly_challenges這個function

接著我們改動views.py, 建立一個monthly_challenges函式
(記得也import HttpResponseNotFound這個功能)

<pre><code> 
def monthly_challenges(request, month):
    challenge_text = None
    if month == "january":
        challenge_text = "Start a new year!"
    elif month == "february":
        challenge_text = "Wake up early every day!"
    elif month == "march":
        challenge_text = "learning Django 20 minutes every day!"
    else:
        return HttpResponseNotFound("this month is not supported. Sorry.")
    return HttpResponse(challenge_text)
</code></pre>

這時候已經可以把前面定義的function都刪掉了，只留下monthly_challenges就好了
可以鍵入http://127.0.0.1:8000/challenges/january 看看結果是不是一樣。

同時試試看 http://127.0.0.1:8000/challenges/december 
看看是不是有如我們安排的顯示not supported頁面

### 再簡化
可以建立一個dictionary，然後再monthly_challenges中呼叫dictionary的內容
<pre><code>
monthly_challenges_dict = {
    "january": "Start a new year!",
    "february": "Wake up early every day!",
    "march": "learning Django 20 minutes every day!",
    "april": "Wake up early!",
    "may":"learning Django 20 minutes every day!",
    "june": "Wake up early!",
    "july":"learning Django 20 minutes every day!",
    "august": "Wake up early!",
    "september":"learning Django 20 minutes every day!",
    "october": "Wake up early!",
    "november":"learning Django 20 minutes every day!",
    "december": "Wake up early!",
}

def monthly_challenges(request, month):
    try:
        challenge_text = monthly_challenges_dict[month]
    except:
        return HttpResponseNotFound("this month is not supported. Sorry.")
    return HttpResponse(challenge_text)
</code></pre>

### path converter
讓我們試著把view的彈性再擴大一點，前面的<month>並沒有指定任何狀況，通常會被當字串做處理
有時候我們希望程式把網址中的字串以數字做理解。
首先我們在views.py定義一個新的function: monthly_challenges_by_number
<pre><code>
def monthly_challenges_by_number(request, month):
    return HttpResponse(month)
</code></pre>

接著把urls.py改成
<pre><code>
urlpatterns= [
    path("&lt;int:month &gt;", views.monthly_challenges_by_numbers),
    path("&lt;str:month&gt;", views.monthly_challenges)
]
</code></pre>

&lt;int:month&gt;在這裡告訴django我們希望以數字的方式理解他，如果有符合的條件可以回傳
因此順序也很重要，如果str放在int前面，那永遠都不會呼叫到int了


### Redirect
我們可能希望數字的網址可以自動連結到相對應的月份，例如 /challenges/1 就連結到/challenges/january 。<br>
這時可以匯入 HttpResponseRedirect函式，並對monthly_challenges_by_number做下面更改
<pre><code>
from django.http import HttpResponse, HttpResponseNotFound, HttpResponseRedirect

def monthly_challenges_by_numbers(request, month):
    months = list (monthly_challenges_dict.keys()) # ["january", "february",....]
    redirect_month = months[month-1]
    return HttpResponseRedirect("/challenges/" + redirect_month)
</code></pre>

進一步還能更改成：
<pre><code>def monthly_challenges_by_numbers(request, month):
    months = list (monthly_challenges_dict.keys()) # ["january", "february",....]
    if month > len(months):
        return HttpResponseNotFound("I am sorry, invalid month!")
    redirect_month = months[month-1]
    return HttpResponseRedirect("/challenges/" + redirect_month) </code></pre>

### Reverse
上面的程式碼有一些缺點，例如：如果我們想改變中間的子網域 challenges變成 challenge，那:
- 必須要改urls.py中的 challenges
- 同時也要改動views.py中 monthly_challenges_by_number裡最後回傳的 /challenges/ 變成 /challenge/

雖然我們不會常常更動網址，但當網站越來越大的時候，難保不會有一些小改動，這時如果漏改了某些地方就會導致掛掉。

這邊我們可以好好的運用Reverse，如果說url() 函式就是將 url --> view 而顯示網頁，那麼reverse就可以說是url的反函式。<br>
Reverse的功能就是將已經存在的特定view其url反轉出來。在redirect這樣的情形下會很好用。

我們將urls.py中，要導引到的url path命名 (運用 name argument) 改成這樣子：
<pre><code> urlpatterns= [
    # path("january", views.january),
    # path("february", views.february),
    # path("march", views.march),
    path("<int:month>", views.monthly_challenges_by_numbers),
    path("<str:month>", views.monthly_challenges,  name = "month-challenge"),
] </code></pre>

然後在views.py中這樣定義：
<pre><code>from django.urls import reverse
def monthly_challenges_by_numbers(request, month):
    months = list (monthly_challenges_dict.keys()) # ["january", "february",....]
    if month > len(months):
        return HttpResponseNotFound("I am sorry, invalid month!")
    redirect_month = months[month-1]
    redirect_path = reverse("month-challenge", args = [redirect_month])  # challenges/january
    return HttpResponseRedirect(redirect_path) </code></pre>
    
注意因為這個url原始的pattern有任意空缺值，因此使用reverse的時候要把args填進去 (以list的形式，外面加上中括弧)

### 回傳html
雖然我們目前成功讓網頁顯示了一般文字(plain text)，但實際上一般的網頁都是以HTML寫成
我們可以將views.py一部分改成html的格式 &lt;h1&gt; title看看
<pre><code>def monthly_challenges(request, month):
    try:
        challenge_text = monthly_challenges_dict[month]
        response_data = f'<h1>{challenge_text}</h1>'
    except:
        return HttpResponseNotFound("&lt;h1&gt;this month is not supported. Sorry.&lt;/h1&gt;")
    return HttpResponse(response_data)</code></pre>

這樣就第一次完成回傳html的任務了

### 自動生成清單頁
我們現在進入每月挑戰的方式是key入網址，但這並非一般使用者的模式，通常是要有一個目錄頁，讓使用者得以點選連結進去。
首先一個清單頁大概是以下面的形式
<pre><code> &lt;ul&gt;
&lt;li&gt;&lt;a href = "link"&gt; January &lt;/a&gt;&lt;/li&gt;
&lt;li&gt;&lt;a href = "link"&gt; February &lt;/a&gt;&lt;/li&gt;
&lt;/ul&gt; </pre></code>

我們新增一個view function: index <br>
裡面創建一個空白的string，然後用for迴圈把每個月的清單形式輸入，可以善用f字串的方式。
<pre><code>def index(request):
    list_item=""
    months = list(monthly_challenges_dict.keys()) # ["january", "february",....]

    for month in months:
        capitalized_month = month.capitalize()
        month_path = reverse("month-challenge", args = [month])
        list_item+=f'&lt;li&gt;&lt;href=\"{month_path}\"&gt; {capitalized_month}&lt;/a&gt;&lt;/li&gt;'

    response_data = f'&lt;ul&gt;{list_item}&lt;/ul&gt;'
    return HttpResponse(response_data) </code></pre>

最後也要記得在/challenges/urls.py 加上一個獨立的 path("", views.index) 才會呼叫該函數
<pre><code>urlpatterns= [
    path("", views.index), #/challenges/
    path("<int:month>", views.monthly_challenges_by_numbers),
    path("<str:month>", views.monthly_challenges,  name = "month-challenge"),
]</pre></code>
