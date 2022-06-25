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

## 建立第一個view
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

## 將view連接到URL
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
</pre></code> 

這時我們import include 函式，include的意義是讓url後面有challenges時，納入challenges app中的urls.py去進一步搜尋，因此程式碼變成這樣
<pre><code> 
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('challenges/', include("challenges.urls"))
]

</pre></code> 

現在我們在回到command line 啟動server一次
<pre><code> 
> python manage.py runserver
</pre></code> 

然後用瀏覽器打開網址 : http://127.0.0.1:8000/challenges/january

應該可以看到螢幕顯示: "This works!"
